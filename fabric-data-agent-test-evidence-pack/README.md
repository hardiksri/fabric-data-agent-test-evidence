# Microsoft Fabric Data Agent — Test Evidence

Hands-on POC evidence for Microsoft Fabric Data Agent security, Code Interpreter execution, sandbox persistence/isolation, runtime guardrails, and post-revocation artifact lifecycle behavior.

> These are empirical observations from the tested Fabric tenant/runtime. They are not contractual Microsoft guarantees or retention SLAs.

## Key lifecycle finding

```text
Authorized user
    ↓
Semantic-model query succeeds
    ↓
Code Interpreter materializes CSV
    ↓
Semantic-model Read permission revoked
    ↓
Fresh source query blocked
    ↓
Previously materialized artifact remains readable by same user
    ↓
New chat for same user can still read artifact
    ↓
Different user cannot see artifact
    ↓
Semantic-model Read restored
    ↓
Original artifact remains with same SHA-256
```

The governance distinction is:

```text
SOURCE AUTHORIZATION
Can the user retrieve new data?

vs.

MATERIALIZED ARTIFACT LIFECYCLE
Can the same user still access data already copied into Code Interpreter?
```

## Test areas

1. OLS enforcement
2. Governed semantic-model → Code Interpreter flow
3. Sandbox persistence and cross-user isolation
4. Runtime/package metadata and guardrails
5. Post-revocation artifact lifecycle

## Important interpretation rules

- A failed Code Interpreter invocation is not proof an artifact was deleted.
- Source authorization and artifact cleanup are separate lifecycle events.
- Same-user persistence is not a retention SLA.
- Run Steps and generated Python must be inspected; the final natural-language response alone may hide reconstruction/write behavior.
- Cross-user negative search supports isolation in the tested environment but does not reveal Microsoft's internal isolation implementation.

POC evidence captured during September–October 2026.
