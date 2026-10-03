# 03 — Sandbox Persistence and Cross-User Isolation

## Goal
Observe whether Code Interpreter artifacts persist for the same user across chats and whether a different user can see another user's artifact.

## Findings
- Same-user artifact persistence across chats was observed in the tested environment.
- The first new-chat check was inconclusive and is retained for transparency.
- A later same-user new-chat check found the existing artifact.
- A separate user could execute Code Interpreter but did not see the first user's isolation artifact.

## Evidence
See [`evidence/`](evidence/) for the original September persistence/isolation captures that motivated the stronger October 3 lifecycle test.

## Boundary
This test supports user-level isolation in the tested environment. It does not establish Microsoft's internal sandbox implementation or a contractual retention period.
