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

## Repository structure

```text
.
├── docs/                      # methodology, findings, evidence maps and limitations
├── test-assets/               # TMDL and reproducible test plan
└── tests/
    ├── 01-ols-enforcement/
    ├── 02-governed-code-interpreter-flow/
    ├── 03-sandbox-persistence-and-isolation/
    ├── 04-runtime-and-guardrails/
    └── 05-post-revocation-artifact-lifecycle/
```

Each test folder contains a short README and an `evidence/` folder. Superseded duplicate captures and exploratory screenshots have been removed so the repository keeps the clearest evidence set.

## Test areas

1. [OLS enforcement](tests/01-ols-enforcement/README.md)
2. [Governed semantic-model → Code Interpreter flow](tests/02-governed-code-interpreter-flow/README.md)
3. [Sandbox persistence and cross-user isolation](tests/03-sandbox-persistence-and-isolation/README.md)
4. [Runtime/package metadata and guardrails](tests/04-runtime-and-guardrails/README.md)
5. [Post-revocation artifact lifecycle](tests/05-post-revocation-artifact-lifecycle/README.md)

Supporting material:
- [Post-revocation findings](docs/POST-REVOCATION-FINDINGS.md)
- [Article evidence map](docs/ARTICLE-EVIDENCE-MAP.md)
- [Test matrix](docs/TEST-MATRIX.md)
- [Methodology](docs/METHODOLOGY.md)
- [Limitations](docs/LIMITATIONS.md)
- [Evidence gaps](docs/EVIDENCE-GAPS.md)
- [Screenshot evidence manifest](docs/SCREENSHOT-MANIFEST.md)
- [Post-revocation test plan](test-assets/Fabric_Data_Agent_Post_Revocation_Artifact_Test_Plan.md)
- [OLS test role TMDL](test-assets/Apply_OLS_Test_Role.tmdl)

## Post-revocation evidence chain

Primary artifact:

```text
ci_post_revoke_test_20261003_v2.csv
Size: 104 bytes
SHA-256: f50ce54091d0e1898d53637c8bd42ff36aba75a25d7e859a574ae97df34bfede
```

The repository includes machine-readable evidence for:
- content read after source access revocation
- same-user/new-chat persistence after revocation
- different-user negative visibility check
- older-artifact retention check
- artifact state after source access was restored

## Important interpretation rules

- A failed Code Interpreter invocation is not proof an artifact was deleted.
- Source authorization and artifact cleanup are separate lifecycle events.
- Same-user persistence is not a retention SLA.
- Run Steps and generated Python must be inspected; the final natural-language response alone may hide reconstruction/write behavior.
- Cross-user negative search supports isolation in the tested environment but does not reveal Microsoft's internal isolation implementation.

## Screenshot evidence

The curated screenshots are stored directly under each test's `evidence/` directory. See [docs/SCREENSHOT-MANIFEST.md](docs/SCREENSHOT-MANIFEST.md) for the current evidence inventory and [docs/ARTICLE-EVIDENCE-MAP.md](docs/ARTICLE-EVIDENCE-MAP.md) for recommended article placement.

POC evidence captured during September–October 2026.
