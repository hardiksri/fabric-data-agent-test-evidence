# Methodology

Two test identities were used: one administrator/control identity and one restricted consumer/test identity.

The restricted user had only the permissions required for the Data Agent and semantic model tests. OLS was applied to `Customer[Company Name]`.

Evidence captured where available:
- prompt/result screenshot
- Run Steps
- generated DAX/Python
- file metadata
- SHA-256 hash
- permission-state transition

Primary artifact size: 104 bytes.
Primary SHA-256: `f50ce54091d0e1898d53637c8bd42ff36aba75a25d7e859a574ae97df34bfede`

Exploratory runs that recreated a requested filename despite a read-only instruction are marked inconclusive. They are retained because they show why generated code must be reviewed.
