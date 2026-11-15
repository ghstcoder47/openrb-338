---
id: TASK-188
title: 'Audio device selection: finish review fixes and reliable recovery'
status: Done
assignee:
  - '@agent-worker'
created_date: '2026-11-15T20:09:00Z'
updated_date: '2026-11-15T20:10:00Z'
labels:
  - player
  - ui
  - cli
  - tests
milestone: m-3
dependencies: []
references:
  - crates/orb-ui/src/app.rs
  - crates/orb-player/src/output.rs
priority: high
type: bug
ordinal: 204000
---

## Description

<!-- SECTION:DESCRIPTION:BEGIN -->
Track the audio-output selection implementation and its review corrections on the user fork. The user needs live switching between Sound BlasterX, Polyend Tracker and installed ASIO drivers. Previous review fixed blocking handoff, ASIO cross-build dependencies, loss of the environment-selected device on retry, aborted retries after a failed open and duplicate error dialogs. Upstream PR #14 now needs rebasing over the new buffer-size request/fallback and callback statistics without losing them. The old fork-local task number collided with the upstream audio-buffer task; preserve the upstream task unchanged and track this work under a newly allocated ID. The user requires ghstcoder47 as sole commit author and no Co-authored-by trailer.
<!-- SECTION:DESCRIPTION:END -->

## Acceptance Criteria
<!-- AC:BEGIN -->
- [x] #1 CLI and UI selection, saved settings and live switching remain available; the effective environment-selected device is preserved for recovery.
- [x] #2 The stream handoff remains lock-free and preserves the player command connection; no_alloc and lock checks pass.
- [x] #3 ASIO is present with defaults and absent with --no-default-features used by the cross-build script.
- [x] #4 Recovery continues through failed opens without Health, schedules up to three attempts with 500 ms spacing, and reports only the last failure once; success and manual selection cancel pending recovery and idle UI schedules a frame.
- [x] #5 Upstream fixed-buffer request, default-buffer fallback, requested/granted diagnostics, callback stats and newer UI changes are preserved.
- [x] #6 Affected player, UI, CLI and artwork tests, clippy and nightly format checks pass. The original task ID collision is gone, no AGENTS changes appear in the PR diff, and the same PR branch is updated as ghstcoder47.
<!-- AC:END -->

## Implementation Plan

<!-- SECTION:PLAN:BEGIN -->
1. Snapshot open upstream PR 14, its base b38cad9 and published head e374b7d; rebase this one-commit branch in place.
2. Combine the two overlapping stream changes: preserve upstream buffer request/fallback and callback statistics, and reuse its existing wait-free Keeper handoff for our stream stop/recovery instead of a second handoff.
3. Preserve upstream task-176, remove only the colliding fork task by taking upstream's absence for that path, and recreate our task through Backlog as task-188.
4. Check callback handoff, retry and UI/art regressions, ASIO-disabled builds, lint and format; continue the rebase as ghstcoder47 and update the same PR branch using an explicit lease.
<!-- SECTION:PLAN:END -->

## Implementation Notes

<!-- SECTION:NOTES:BEGIN -->
Branch: neonbytevoid-audio-device-selection. User approved resolving conflicts in existing pingw33n/OpenRB-338 PR 14, not merging. The published commit e374b7d was one commit ahead and 24 upstream commits behind. Only output.rs had textual conflicts; the newer UI redesign, DX12 startup, CLI Ctrl+C handling and buffer diagnostics merged automatically and are preserved. Reused upstream Keeper and its rtrb::Producer<Audio> with a retained Consumer<Audio> in Stream; removed the former second queue/Box/Output::drop path. Failed fixed-buffer builds still recover Audio and fall back to default size. A play failure now returns recovered Audio with its error. Added a device-free test that a dropped callback returns a player which still receives Stop commands. Prior review evidence: 381 affected-crate tests passed before rebase; the user confirmed live switching between Sound BlasterX and Polyend Tracker; disconnect/reconnect and actual ASIO sound were not newly verified. The fork task was first called task-176, but upstream independently allocated that ID to its audio-buffer fix; doctor preview wanted to rename upstream, which was deliberately NOT applied. The upstream task is unchanged; only our colliding task was recreated as task-188 through Backlog.

Rebased verification: cargo test -p orb-player -p orb-ui -p orb-cli -p orb-ui-art passed, 412 tests total, including the new callback-teardown handoff test, the six retry tests, buffer_choice, callback_stats and no_alloc/lock checks. Clippy for those four crates with --all-targets -- -D warnings and nightly format check passed. The Windows feature tree includes asio-sys by default and excludes it with --no-default-features; cargo check -p orb-ui -p orb-cli --no-default-features --locked also passed. No upstream task, AGENTS.md, art, DSP or reference renders were edited by the rebased PR. Physical audio switching was not repeated for this rebase. Prior published commit e374b7d is retained in history for recovery; update only the same PR branch with a lease against that exact SHA.
<!-- SECTION:NOTES:END -->

## Final Summary

<!-- SECTION:FINAL_SUMMARY:BEGIN -->
Rebased the single audio-device-selection commit onto upstream b38cad9. Kept the upstream buffer fallback and diagnostics, reusing its wait-free callback Keeper to return Audio for device switching and failed starts. Both callback handoffs are now one mechanism with no Box allocation on teardown. Added a command-connection handoff regression. Tests: 412 passed across player/UI/CLI/art; clippy, nightly fmt, ASIO feature trees and no-default-feature build passed. Renumbered only the fork task to task-188; upstream task-176 is unchanged. Branch: neonbytevoid-audio-device-selection, replacing e374b7d in existing upstream PR 14. Author/committer ghstcoder47, no coauthor as explicitly requested. No merge requested or performed.
<!-- SECTION:FINAL_SUMMARY:END -->
