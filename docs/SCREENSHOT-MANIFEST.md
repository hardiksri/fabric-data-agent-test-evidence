# Screenshot Evidence Manifest

This manifest lists the curated screenshots retained in the repository after removing superseded duplicates and exploratory captures.

## 01 — OLS enforcement

Path: `tests/01-ols-enforcement/evidence/`

- `11-semantic-model-share-link-read.png` — semantic-model sharing link grants Read.
- `12-workspace-restricted-user-absent.png` — restricted test user is not a workspace role member.
- `13-semantic-model-direct-read.png` — test user has semantic-model Read.
- `14-ols-role-membership.png` — test user is a member of `DataAgent_OLS_Test`.
- `15-company-name-ols-block.png` — `Company Name` is unavailable to the restricted identity.
- `16-authorized-region-query.png` — authorized `Total Sales by Region` query succeeds.

## 02 — Governed semantic-model → Code Interpreter flow

Path: `tests/02-governed-code-interpreter-flow/evidence/`

- `20-artifact-creation-run-steps.png` — semantic-model retrieval followed by Code Interpreter execution.
- `21-artifact-created-hash-result.png` — created CSV metadata, size and SHA-256.
- `22-generated-dax.png` — generated DAX for `Total Sales by Region`.
- `23-generated-python-governed-input.png` — generated Python reading governed tool-result artifacts.

## 03 — Sandbox persistence and isolation

Path: `tests/03-sandbox-persistence-and-isolation/evidence/`

- `04-same-chat-artifact-visible.png`
- `06-new-chat-first-check-inconclusive.png`
- `07-new-chat-artifact-found.png`
- `08-new-chat-governed-dax.png`
- `09-user-a-isolation-artifact-created.png`
- `10-user-a-isolation-check.png`

These are retained because they document the earlier persistence/isolation POC that motivated the later October 3 lifecycle test.

## 04 — Runtime and guardrails

Path: `tests/04-runtime-and-guardrails/evidence/`

- `13-network-egress-policy-test.png`
- `15-environment-variable-enumeration-refused.png`
- `16-runtime-platform-versions.png`
- `17-installed-package-summary.png`
- `18-controlled-keyerror-test.png`

## 05 — Post-revocation artifact lifecycle

Path: `tests/05-post-revocation-artifact-lifecycle/evidence/`

- `20-fresh-source-query-blocked.png` — fresh semantic-model retrieval fails after Read revocation.
- `21-permission-removal-confirmed.png` — permission state after source Read removal.
- `22-post-revoke-ci-executes.png` — Code Interpreter remains callable in the existing context.
- `23-post-revoke-readonly-run-steps.png` — read-only artifact inspection Run Steps.
- `24-post-revoke-content-check.png` — previously materialized CSV remains readable.
- `25-semantic-model-read-restored.png` — semantic-model Read restored.

Machine-readable evidence in the same folder provides the stronger deterministic checks for same-user/new-chat persistence, cross-user negative visibility, artifact content, size and SHA-256.

## Primary artifact

```text
ci_post_revoke_test_20261003_v2.csv
Size: 104 bytes
SHA-256: f50ce54091d0e1898d53637c8bd42ff36aba75a25d7e859a574ae97df34bfede
```
