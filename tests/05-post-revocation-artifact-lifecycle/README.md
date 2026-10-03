# 05 — Post-Revocation Artifact Lifecycle

## Objective
Determine whether data already materialized into Code Interpreter remains available after the same user's semantic-model Read permission is revoked.

## Test sequence

```text
Read granted
→ authorized source query succeeds
→ Code Interpreter materializes CSV
→ Read revoked
→ fresh source query blocked
→ existing chat: artifact still readable
→ new chat, same user: artifact still readable
→ different user/admin: artifact not found
→ Read restored
→ fresh source query succeeds again
→ original artifact still present
```

## Primary artifact
- filename suffix: `ci_post_revoke_test_20261003_v2.csv`
- size: `104` bytes
- SHA-256: `f50ce54091d0e1898d53637c8bd42ff36aba75a25d7e859a574ae97df34bfede`

## Evidence
The [`evidence/`](evidence/) folder contains:
- curated screenshots for revocation, blocked source query, post-revocation Code Interpreter execution/readback, and Read restoration
- `post_revoke_artifact_check_20261003.txt`
- `post_revoke_content_check_20261003.txt`
- `new_chat_post_revoke_check_20261003.txt`
- `admin_cross_user_check_20261003.txt`
- `post_restore_artifact_check_20261003.txt`
- `old_artifact_retention_check_20261003.txt`

See also [`generated-code.md`](generated-code.md) for representative generated Python and [`../../docs/POST-REVOCATION-FINDINGS.md`](../../docs/POST-REVOCATION-FINDINGS.md) for the consolidated interpretation.

## Governance conclusion
In this POC, source authorization and materialized-result lifecycle behaved independently. Revoking source Read stopped new governed queries but did not immediately make already-materialized Code Interpreter data unavailable to the same user.

Older artifacts from a previous test were no longer visible several days later, so this is not evidence of permanent storage or a retention SLA.
