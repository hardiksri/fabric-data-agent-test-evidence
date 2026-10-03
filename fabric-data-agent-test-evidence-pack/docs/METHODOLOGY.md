# Methodology

## Identities

Two test identities were used:
- an administrator/control identity
- a restricted consumer/test identity

The restricted user was removed from workspace roles and granted only the permissions required for the Data Agent and semantic model tests. OLS was applied to `Customer[Company Name]`.

## Evidence standard

Where possible, every claim is supported by:
- prompt/result screenshot
- Run Steps
- generated DAX/Python
- file metadata
- SHA-256 hash
- explicit permission-state transition

## Artifact hash

The primary post-revocation artifact was 104 bytes and the observed SHA-256 was:

`f50ce54091d0e1898d53637c8bd42ff36aba75a25d7e859a574ae97df34bfede`

## Caution

Some exploratory runs were intentionally marked inconclusive when the generated Python recreated a requested filename despite a read-only instruction. Those runs are preserved because they demonstrate why generated code must be reviewed.
