# Generated Code Snippets

This file preserves representative generated code from the POC.

## Artifact creation
The Data Agent queried `Total Sales by Region`, then Code Interpreter wrote:

`/mnt/data/ci_post_revoke_test_20261003_v2.csv`

and calculated its size and SHA-256.

## Post-revocation read-only inspection

```python
from pathlib import Path
import hashlib

root = Path('/mnt/data')
needle = 'ci_post_revoke_test_20261003_v2.csv'
matches = sorted(
    (p for p in root.iterdir() if p.is_file() and needle in p.name),
    key=lambda p: p.name
)

# Matching target files were read only.
```

## Cross-user check

The administrator/control identity separately searched `/mnt/data` for the same filename suffix. The observed result was `MATCH_COUNT: 0`.
