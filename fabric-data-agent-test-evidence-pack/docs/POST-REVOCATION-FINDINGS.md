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

## Conclusion

In the tested environment, **source authorization and the lifecycle of already-materialized Code Interpreter data behaved independently**.

Revoking the user's semantic-model Read permission prevented fresh governed queries, but it did not immediately remove the already-materialized Code Interpreter artifact for that same user. The artifact remained readable in the existing chat and was also found in a new chat for the same user. A different user/control identity did not find the artifact.

The result should not be interpreted as a documented retention guarantee. Older artifacts from the earlier POC were not present several days later, and the exact cleanup interval was not established.
