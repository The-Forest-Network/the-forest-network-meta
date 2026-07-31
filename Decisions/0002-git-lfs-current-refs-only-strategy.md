# 0002 — Fetch/push Git LFS objects for current refs only, not full history

**Date:** 2026-07-31

## Context

Both `the-forest-network-ios` and `the-forest-network-android` use Git LFS for snapshot/screenshot test images (and, on Android, media test assets). Neither fork's LFS storage was ever actually populated — the regular git history came across when each repo was created, but the LFS binary blobs never did. Every CI workflow that checks out the repo with `nschloe/action-cached-lfs-checkout` has been failing 100% of the time as a result, immediately at checkout, with `Object does not exist on the server: [404]`. See [issue #23](https://github.com/The-Forest-Network/the-forest-network-ios/issues/23) (iOS) for the full CI audit this was found during; the same root cause affects Android's `validate-lfs.yml` and `tests.yml`.

We'd tried to fix this once before by pulling the **full** LFS history (`git lfs fetch --all`) and abandoned it — it's a huge amount of data and kept failing/stalling. Investigating this time around found two things that change the picture:

1. **Full history vs. current state is roughly a 10x difference.** Current `develop` HEAD only needs ~378 MB (iOS) / ~204 MB (Android) — full history (every LFS object ever committed, including every replaced/superseded snapshot) is ~4.0 GB (iOS) / ~2.0 GB (Android). CI only ever checks out one commit at a time and only needs the LFS objects referenced by that commit's tree, so full history was never actually necessary for CI to pass — it was ~10x more data than needed for no benefit.
2. **GitHub's free LFS quota is 10 GiB storage + 10 GiB bandwidth per month, shared per-organization (not per-repo)** — a correction from outdated information (previously assumed 1 GiB per-repo). Even the *full* combined history (~6 GB across both repos) would have fit within that. So the earlier abandoned attempt likely wasn't a quota rejection at all, but a reliability/timeout problem — bulk-transferring ~65,000 individual objects across two repos is exactly the kind of long-running operation that stalls on a flaky connection, independent of any hard quota block.

We also found `git lfs push <remote> <ref>` (without `--all`) is unsafe for this situation: by default it only pushes objects it thinks aren't already on the remote, determined by diffing against the local `origin/<ref>` tracking ref — not by actually querying the server. Since our local `develop` matched `origin/develop` exactly (no new commits), it silently pushed nothing at all, despite the server genuinely being empty. `--all <ref>` doesn't fix this either — it walks the *entire* commit history reachable from that ref (effectively the same ~4 GB/~2 GB full-history problem), not just the current tree. The fix that actually worked: `git lfs ls-files -l` to list current-tree object IDs, piped into `git lfs push --object-id --stdin`, which pushes exactly those objects regardless of what git-lfs's local ref-diffing heuristic assumes.

## Decision

Fetch and push Git LFS objects for **current refs only**, not full history, using explicit object IDs rather than `git lfs push <ref>`:

```
git lfs fetch origin develop
git lfs ls-files -l | awk '{print $1}' | git lfs push --object-id --stdin origin
```

Confirmed working: previously-failing CI runs on both repos were re-run after this and passed (or, on iOS, progressed past the LFS checkout step entirely, surfacing an unrelated pre-existing SwiftFormat issue — see [0003](0003-pin-swiftformat-to-stable-releases.md)).

Full history remains available (and affordable, per the quota correction above) if we ever want it for other reasons — e.g. being able to check out an old commit and have its snapshots resolve — but it isn't needed for CI, which is the immediate problem this solves.

## Consequences

- CI can now actually check out both repos' LFS-tracked files. Whichever tests were blocked purely on the LFS 404 (as opposed to failing for unrelated reasons) should now run for real.
- Anyone who needs LFS objects for a commit *other* than current `develop` (e.g. checking out an old PR branch, bisecting) will still hit 404s until we either push that ref's objects too, or eventually do a full-history push. Not addressed by this decision.
- No GitHub LFS billing exposure from this — 378 MB + 204 MB is well within the free 10 GiB/month org-wide quota, with plenty of headroom left even for occasional full-history pushes later.
