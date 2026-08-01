# 0004 — GitHub Actions workflow status index (iOS and Android)

**Date:** 2026-08-01

## Context

iOS got a systematic, workflow-by-workflow audit ([the-forest-network-ios#23](https://github.com/The-Forest-Network/the-forest-network-ios/issues/23)) — every `.github/workflows/*.yml` file was read, its actual run history checked, and a keep/delete/fix recommendation made for each. Android did **not** get the same treatment. Everything found on Android (LFS, Detekt dead code, stale locale strings, Codecov, Maestro flakiness) was discovered incidentally, while trying to get [the-forest-network-android#16](https://github.com/The-Forest-Network/the-forest-network-android/pull/16) green — a real but ad-hoc process, not a deliberate pass over the whole workflow directory the way iOS got.

This record exists so there's one place that says what's actually been decided for each repo's CI, rather than that being scattered across ~10 issues and ~8 PRs with no index. It supersedes nothing in [0001](0001-disable-fork-translation-sync-workflows.md)/[0002](0002-git-lfs-current-refs-only-strategy.md)/[0003](0003-pin-swiftformat-to-stable-releases.md) — those go deeper on their specific topics; this is the map of everything.

## Decision

### iOS (`the-forest-network-ios`)

| Workflow | Status | Reference |
|---|---|---|
| `translations-pr.yml` (Localazy) | Deleted | [#21](https://github.com/The-Forest-Network/the-forest-network-ios/issues/21) → [PR #20](https://github.com/The-Forest-Network/the-forest-network-ios/pull/20) (open) |
| `sync-sas-strings.yml` | Added, kept | [0001](0001-disable-fork-translation-sync-workflows.md) |
| LFS-dependent workflows (`unit-tests`, `ui-tests`, `accessibility-tests`, `compound-ios`, `unit-tests-enterprise`) | Checkout fixed | [0002](0002-git-lfs-current-refs-only-strategy.md) |
| SwiftFormat pin (affects `unit-tests.yml`'s lint step) | Pinned to 0.62.1 | [PR #24](https://github.com/The-Forest-Network/the-forest-network-ios/pull/24), [0003](0003-pin-swiftformat-to-stable-releases.md) |
| `automatic-calendar-version.yml`, `post-release.yml`, `triage-incoming.yml`, `triage-labelled.yml` | **Recommended delete, not yet done** | [#23](https://github.com/The-Forest-Network/the-forest-network-ios/issues/23) |
| `unit-tests-enterprise.yml`, `integration-tests.yml` | **Recommended delete, not yet done** | [#23](https://github.com/The-Forest-Network/the-forest-network-ios/issues/23) |
| `danger.yml` | **Recommended fix-or-delete, not yet done** | [#23](https://github.com/The-Forest-Network/the-forest-network-ios/issues/23) |
| `zizmor.yml`, `blocked.yml`, `stale.yml`, `record-snapshots.yml`, `renovate-xcodegen.yml` | Keep as-is, working | [#23](https://github.com/The-Forest-Network/the-forest-network-ios/issues/23) |

### Android (`the-forest-network-android`)

| Workflow / issue | Status | Reference |
|---|---|---|
| `sync-localazy.yml` | Deleted | [#15](https://github.com/The-Forest-Network/the-forest-network-android/issues/15) → [PR #14](https://github.com/The-Forest-Network/the-forest-network-android/pull/14) (merged) |
| `sync-sas-strings.yml` | Kept as-is, already working | [0001](0001-disable-fork-translation-sync-workflows.md) |
| `validate-lfs.yml` / `tests.yml` LFS checkout | Fixed | [0002](0002-git-lfs-current-refs-only-strategy.md) |
| Detekt dead-code error (`OnBoardingLogo` unused) | Fixed | [PR #16](https://github.com/The-Forest-Network/the-forest-network-android/pull/16) |
| Lint `StringFormatCount` (76 pre-existing errors, stale locale strings) | Baselined; real fix (dedicated string keys, mirroring the iOS branding-string pattern) not yet done | [#18](https://github.com/The-Forest-Network/the-forest-network-android/issues/18) |
| Lint `UnusedResources` on `onboarding_logo.png` | Fixed properly (`tools:keep`, not baselined — resource is real, just dynamically referenced) | [PR #16](https://github.com/The-Forest-Network/the-forest-network-android/pull/16) |
| Codecov upload (missing `CODECOV_TOKEN`) | **Open, not fixed** — same gap as iOS | [#19](https://github.com/The-Forest-Network/the-forest-network-android/issues/19) |
| Maestro emulator boot flakiness | **Open, not fixed** — likely runner flakiness, retry is the workaround for now | [#20](https://github.com/The-Forest-Network/the-forest-network-android/issues/20) |
| Everything else (`build_enterprise.yml`, `build.yml`, `fork-pr-notice.yml`, `generate_github_pages.yml`, `gradle-wrapper-update.yml`, `nightly.yml`, `nightlyReports.yml`, `post-release.yml`, `pull_request.yml`, `quality.yml`, `recordScreenshots.yml`, `release.yml`, `sonar.yml`, `stale-issues.yml`, `triage-incoming.yml`, `triage-labelled.yml`) | **Not reviewed at all** — no equivalent of iOS's #23 audit has been done | — |

## Consequences

- Both repos still have open, known gaps (iOS: 6 recommended-but-not-executed deletions/fixes from #23; Android: #19, #20, and the #18 real fix).
- Android in particular has a much larger unreviewed surface than iOS did before #23 — worth doing the equivalent systematic audit there rather than continuing to find things incidentally. Not done as part of this record; flagging it as the natural next step.
- Future workflow-related findings should get an issue + be added to this table (or a follow-up decision record) rather than living only in PR conversation history, so this stays a usable index instead of going stale immediately.
