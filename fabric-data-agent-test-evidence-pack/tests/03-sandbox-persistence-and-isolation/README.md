# 03 — Sandbox Persistence and Isolation

## Findings
- Artifacts were accessible across chats for the same user in the tested context.
- A second user could execute Code Interpreter but did not see the first user's isolation artifact.
- Older artifacts from an earlier POC were no longer visible several days later.
- Exact retention duration remains unknown.
