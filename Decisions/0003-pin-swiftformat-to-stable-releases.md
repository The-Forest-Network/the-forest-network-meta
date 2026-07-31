# 0003 — Pin SwiftFormat to stable releases instead of tracking --HEAD

**Date:** 2026-07-31

## Context

While investigating the Git LFS CI failures ([0002](0002-git-lfs-current-refs-only-strategy.md)), fixing LFS on iOS revealed a second, unrelated problem underneath it: once CI could actually check out the repo, it started failing at the `swiftformat --lint .` step instead — flagging formatting violations across ~20 pre-existing files nobody had recently touched (`BlurHashEncode.swift`, `String.swift`, `LocationSharingScreenViewModel.swift`, etc.).

The cause is in `ci_scripts/ci_common.sh`, which is upstream Element's own script, not something we changed: `install_swiftformat_head()` installs SwiftFormat via `brew install swiftformat --HEAD` — the bleeding-edge unreleased build — rather than a pinned version. `Tools/Sources/Commands/CI/CI.swift` then runs `swiftformat --lint .` across the *entire* repository on every CI run, not just changed files. Combined, this means CI's formatting rules are a moving target: whenever SwiftFormat's own main branch changes or adds a rule, any repo using this setup can start failing lint with zero code changes, simply because nobody has reformatted against that day's HEAD build yet. Upstream Element apparently keeps pace with this themselves (their own `develop` branch was passing cleanly as of 2026-07-31, against SwiftFormat 0.62.1); we don't have the equivalent ongoing attention on this fork, so it had drifted.

Checked whether pinning to something *behind* what upstream currently uses would somehow be incompatible with existing files — it isn't. SwiftFormat isn't stateful between runs; running it with any given version just reformats to that version's rules and produces a clean, self-consistent baseline regardless of what formatted the code previously. The only cost of picking an older version is a larger one-time reformatting diff.

Also worth noting: upstream doesn't pin either, so "match whatever version they're using" isn't a stable, checkable target — there's no version tag on their side to look up, only whatever HEAD commit SwiftFormat happened to be at when their last CI run passed. We confirmed this empirically by checking a recent successful upstream `develop` CI run's logs directly, which is how we landed on 0.62.1.

## Decision

Pin to the latest **stable** SwiftFormat release (currently 0.62.1, which happened to exactly match Homebrew's current stable formula and what upstream's `develop` was clean against at the time) instead of `--HEAD`. Renamed `install_swiftformat_head` → `install_swiftformat` in `ci_scripts/ci_common.sh` accordingly (also used by local dev setup via `Tools/Sources/Commands/SetupProject.swift`), and reformatted the repository to match (~20 files, purely mechanical changes — see [PR #24](https://github.com/The-Forest-Network/the-forest-network-ios/pull/24)).

Going forward: since upstream continues tracking HEAD and we don't, formatting will drift again over time, and will drift further whenever we merge upstream changes (their code will have been formatted against whatever HEAD was current for them at that point, which may not match our pin). Rather than trying to keep pace with a moving target, **reconcile formatting as a normal step of every upstream merge** — re-run `swiftformat .` against our currently-pinned version as part of resolving that merge, the same way any other merge conflict gets resolved, and optionally bump the pin to a newer stable release at that point if there's a good reason to.

## Consequences

- CI's formatting checks are now stable indefinitely, regardless of what SwiftFormat's upstream project does next — no more zero-code-change breakage.
- We've deliberately diverged from upstream Element's own practice of tracking `--HEAD`. That's a reasonable tradeoff for a small fork not actively contributing formatting-rule-compliance patches back upstream, but it does mean our formatting will gradually diverge from theirs between merges.
- Future upstream merges need an explicit reformatting step as part of the merge process — not automatic, and easy to forget if whoever does the merge doesn't know this decision exists.
