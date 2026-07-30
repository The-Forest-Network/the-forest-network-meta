# 0001 — Disable scheduled Localazy sync on our iOS/Android forks; keep SAS string sync

## Context

Both `the-forest-network-ios` and `the-forest-network-android` are copies of `element-hq/element-x-ios` / `element-hq/element-x-android`, not native GitHub forks, but they retain the full upstream commit history. Neither repo has ever pulled/merged upstream changes since being created.

While investigating how our hand-branded onboarding copy (e.g. "Not in the Village yet?") could survive a future upstream merge, we found each repo also inherited upstream's own scheduled string-sync CI workflows:

- **iOS** `translations-pr.yml` — downloads all Localazy translations weekly. Correctly gated with `if: github.repository == 'element-hq/element-x-ios'`, so it has never actually run on our fork.
- **Android** `sync-localazy.yml` — same purpose, but its fork-skip condition (`if: github.event_name != 'pull_request' || ...`) only excludes `pull_request`-triggered runs. Since this workflow only ever fires on `schedule`/`workflow_dispatch`, the condition never applies — **it has been running weekly on our repo since at least 2026-06-01**, pulling Element's official strings via a public read-only key hardcoded in `tools/localazy/generateLocalazyConfig.py`. It's failed every time at the final "Create Pull Request" step because `secrets.DANGER_GITHUB_API_TOKEN` isn't configured here, so nothing has actually landed in our repo — but it was one added secret away from silently opening a PR that overwrites our strings with upstream's stock copy. Confirmed the same broken gate exists in Element's own current upstream copy of this file — not something introduced by our fork.
- **Android** `sync-sas-strings.yml` — syncs the Matrix SAS (Short Authentication String) emoji/word list used in device-verification, sourced from `matrix-org/matrix-spec`, unrelated to Localazy. Same broken gate, but this one uses the default `GITHUB_TOKEN` rather than a missing secret, so it's been **succeeding** weekly since 2026-06-08. It hasn't produced a PR yet only because the upstream word list hasn't changed since our fork started.
- **iOS** had no standalone equivalent — its SAS generation (`swift run tools generate-sas`, confirmed independent of Localazy) was bundled inside the correctly-gated `translations-pr.yml`, so it has never run here either.

## Decision

**Delete the Localazy sync workflows** on both platforms (iOS `translations-pr.yml`, Android `sync-localazy.yml`). Reasoning: Localazy syncs are just regular commits inside `element-hq/element-x-ios`/`element-x-android`. Whenever we do eventually merge upstream's `develop` branch, we get their already-resolved translations for free as part of that same merge — there's no separate "pull strings" step we're supposed to run independently, and running one on our own schedule only risks clobbering our hand-branded strings with upstream's stock copy on a cadence we don't control.

**Keep SAS string sync running on both platforms**, unlike Localazy. Unlike translations, it's not purely upstream-merge-redundant: SAS verification requires two devices to independently compute the same emoji from the same secret, so a Forest Network user verifying against a *different, more up-to-date* Matrix client could see mismatched emoji if we fall behind on the canonical `matrix-spec` list. In practice, the security-relevant emoji↔index mapping is effectively frozen for ecosystem-wide interop reasons (changing it would break every Matrix client, not just ours), so what actually changes over time is new localized *names* for the emoji, not the emoji themselves — meaning the realistic cost of falling behind is a missing translation, not a broken verification. Given the low cost of keeping it and the (small but nonzero) safety margin against a cross-client verification mismatch, we chose to keep it rather than rely solely on the next upstream merge.
- Android's `sync-sas-strings.yml` was left as-is (already correctly functioning, if only by the same accidental gate as the deleted Localazy job).
- iOS didn't have a standalone version, so we split its `generate-sas` step out into a new `sync-sas-strings.yml`, mirroring Android's, using the default `GITHUB_TOKEN` (not `ELEMENT_BOT_TOKEN`, which we don't have configured).

## Consequences

- Neither fork will auto-sync translations from Localazy going forward. Hand-branded strings (onboarding copy, etc.) are only at risk during an actual future upstream merge, not from routine CI.
- Both platforms now have a working, low-risk SAS string sync that will open a PR against `develop` if Matrix's spec ever adds new language support for the verification emoji names.
- We still don't have a documented process for pulling upstream code changes into either fork (no `upstream` git remote configured, no native GitHub fork relationship, no automation). That's a separate open item, not resolved by this decision.
