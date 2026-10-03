# 05 — Post-Revocation Artifact Lifecycle

## Objective
Determine whether already-materialized Code Interpreter data remains available after the same user's semantic-model Read permission is revoked.

## Result

```text
Read granted
→ source query succeeds
→ artifact created
→ Read revoked
→ fresh source query blocked
→ same-chat artifact readable
→ new-chat artifact readable
→ different user cannot find artifact
→ Read restored
→ source query succeeds
→ original artifact still present
```

Primary artifact:
- filename suffix: `ci_post_revoke_test_20261003_v2.csv`
- size: 104 bytes
- SHA-256: `f50ce54091d0e1898d53637c8bd42ff36aba75a25d7e859a574ae97df34bfede`

## Governance conclusion
In this POC, source authorization and materialized-result lifecycle behaved independently. Revoking source Read stopped new governed queries but did not immediately make already-materialized Code Interpreter data unavailable to the same user.

This does not establish a retention SLA.
