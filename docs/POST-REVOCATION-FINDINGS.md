# Post-Revocation Code Interpreter Findings

## Question

What happens to data already materialized into Fabric Data Agent Code Interpreter after the same user's semantic-model Read permission is revoked?

## Test sequence

```text
Semantic Model Read granted
        ↓
Authorized Total Sales by Region query succeeds
        ↓
Code Interpreter creates ci_post_revoke_test_20261003_v2.csv
        ↓
Semantic Model Read removed
        ↓
Fresh semantic-model query blocked
        ↓
Existing chat: Code Interpreter can still read prior artifact
        ↓
New chat, same user: prior artifact still found
        ↓
Different user/admin: artifact not found
        ↓
Semantic Model Read restored
        ↓
Fresh semantic-model query succeeds again
        ↓
Original artifact still found with same size and SHA-256
```

## Primary artifact

- Filename suffix: `ci_post_revoke_test_20261003_v2.csv`
- Size: `104` bytes
- SHA-256: `f50ce54091d0e1898d53637c8bd42ff36aba75a25d7e859a574ae97df34bfede`

## Machine-readable evidence

- [`post_revoke_artifact_check_20261003.txt`](../tests/05-post-revocation-artifact-lifecycle/evidence/post_revoke_artifact_check_20261003.txt) — matching artifact found after source access revocation.
- [`post_revoke_content_check_20261003.txt`](../tests/05-post-revocation-artifact-lifecycle/evidence/post_revoke_content_check_20261003.txt) — verifies the five CSV rows, file size and SHA-256 after revocation.
- [`new_chat_post_revoke_check_20261003.txt`](../tests/05-post-revocation-artifact-lifecycle/evidence/new_chat_post_revoke_check_20261003.txt) — same artifact found from a new chat for the same user after revocation.
- [`admin_cross_user_check_20261003.txt`](../tests/05-post-revocation-artifact-lifecycle/evidence/admin_cross_user_check_20261003.txt) — `MATCH_COUNT: 0` for a different user/control identity.
- [`post_restore_artifact_check_20261003.txt`](../tests/05-post-revocation-artifact-lifecycle/evidence/post_restore_artifact_check_20261003.txt) — same artifact/hash found after source access was restored.
- [`old_artifact_retention_check_20261003.txt`](../tests/05-post-revocation-artifact-lifecycle/evidence/old_artifact_retention_check_20261003.txt) — older artifacts from the prior POC were no longer visible several days later.

## Conclusion

In the tested environment, **source authorization and the lifecycle of already-materialized Code Interpreter data behaved independently**.

Revoking the user's semantic-model Read permission prevented fresh governed queries, but it did not immediately remove the already-materialized Code Interpreter artifact for that same user. The artifact remained readable in the existing chat and was also found in a new chat for the same user. A different user/control identity did not find the artifact.

The result should not be interpreted as a documented retention guarantee. Older artifacts from the earlier POC were not present several days later, and the exact cleanup interval was not established.

## Governance implication

Permission revocation should be evaluated separately for:

1. **New source retrieval** — can the user still query the governed source?
2. **Execution access** — can Code Interpreter still execute?
3. **Previously materialized data** — can the user still read an artifact created while access was valid?
4. **Artifact retention** — when is the sandbox artifact eventually cleaned up?

A production governance review should not assume that revoking source permissions immediately invalidates all data previously materialized into a downstream analytical execution environment.
