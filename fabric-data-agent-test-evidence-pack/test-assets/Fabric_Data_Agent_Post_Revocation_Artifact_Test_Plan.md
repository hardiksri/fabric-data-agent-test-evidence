# Fabric Data Agent Code Interpreter — Post-Revocation Artifact Test Plan

## Objective
Validate what happens to a Code Interpreter artifact created from an **authorized semantic-model result** after the same user's access to the source is revoked.

Keep these four questions separate:
1. Can the user still query the source?
2. Can Code Interpreter still execute?
3. Does the old artifact still exist?
4. Can the user still read the old artifact?

> Run only with a dedicated test user and non-sensitive test data.

## Test setup
Use one test user (User A). Keep the **Data Agent itself accessible** while revoking only the **underlying semantic-model/source access**, if your permission model allows it.

Use this artifact name:

`ci_post_revoke_test_20261002.csv`

Capture a SHA-256 hash when the file is created. That lets us prove whether a later file is the same artifact.

## Evidence folder structure

```text
evidence/
└── code-interpreter/
    └── post-revocation-access/
        ├── README.md
        ├── 01-baseline/
        ├── 02-artifact-creation/
        ├── 03-access-revoked/
        ├── 04-source-access-check/
        ├── 05-same-chat-artifact-check/
        ├── 06-new-chat-artifact-check/
        └── 07-access-restored-check/
```

Each folder should contain the prompt, response screenshot, Run Steps screenshot, generated code when available, timestamp, and notes.

---

## Test 1 — Baseline source access

### Prompt
```text
Using the semantic model, return Total Sales by Region.

Return only:
- Region
- Total Sales
```

Capture the user identity, workspace role, source permissions, RLS/OLS role membership, final response, Run Steps, generated DAX, and timestamp.

Expected result: source query succeeds.

---

## Test 2 — Create authorized artifact

### Prompt
```text
First retrieve Total Sales by Region from the semantic model.

Then invoke Code Interpreter.

Using only the returned semantic-model result:

1. Save the result as:
   /mnt/data/ci_post_revoke_test_20261002.csv

2. Do not add any additional business data.

3. Calculate and return:
   - exact filename
   - full path
   - row count
   - file size in bytes
   - SHA-256 hash

4. Confirm whether the file exists.

Do not access the network.
Do not inspect environment variables.
```

Capture the DAX, source result, generated Python, path, size, row count, SHA-256, response, Run Steps, and timestamp.

---

## Test 3 — Same-chat pre-revocation check

### Prompt
```text
Invoke Code Interpreter.

Do not query the semantic model again.

Check only for this existing file:

/mnt/data/ci_post_revoke_test_20261002.csv

Do not create, copy, rename, or modify it.

Return only:
- File visible: Yes/No
- Full path
- File size in bytes
- SHA-256 hash
```

Expected: visible = Yes and SHA-256 matches Test 2.

---

## Test 4 — Revoke source access

Revoke User A's **underlying semantic-model/source access** while preserving Data Agent access if possible.

Capture before/after evidence of:
- workspace role
- semantic-model/source permission
- Entra/AD group membership if used
- exact revocation timestamp

Do not clear chat or delete the artifact.

---

## Test 5 — Confirm source access is revoked

### Prompt
```text
Using the semantic model, return Total Sales by Region.

Do not use any previously generated file or cached result.
```

Expected: source query fails or is denied.

If it still succeeds, stop and wait for permission propagation.

---

## Test 6 — Same-chat artifact access after revocation

### Prompt
```text
Do not query the semantic model.

Invoke Code Interpreter only to inspect the existing local file:

/mnt/data/ci_post_revoke_test_20261002.csv

Do not create, copy, rename, or modify any file.

If Code Interpreter runs, return only:
- Code Interpreter executed: Yes/No
- File visible: Yes/No
- Full path
- File size in bytes
- SHA-256 hash

If Code Interpreter cannot run, return:
- Code Interpreter executed: No
- reason shown by the system
```

Interpretation:
- CI runs + file visible + hash matches → artifact remains accessible after revocation.
- CI runs + file not visible → artifact unavailable in that execution context.
- CI does not run because source retrieval is required → **inconclusive for artifact deletion**.

---

## Test 7 — New-chat artifact access after revocation

Open a **new Data Agent chat as the same User A**.

### Prompt
```text
Do not query the semantic model.

Invoke Code Interpreter only to search the current /mnt/data working area for this exact filename:

ci_post_revoke_test_20261002.csv

Do not create, copy, rename, or modify any file.

If Code Interpreter runs, return only:
- Code Interpreter executed: Yes/No
- File visible: Yes/No
- Matching filename
- File size in bytes
- SHA-256 hash

If Code Interpreter cannot run, return:
- Code Interpreter executed: No
- reason shown by the system
```

Use the same interpretation rules as Test 6.

---

## Test 8 — Restore source access

Restore User A's original source permissions.

Capture the restoration timestamp and confirm that a fresh semantic-model query works again.

---

## Test 9 — Check artifact after access restoration

### Prompt
```text
First retrieve Total Sales by Region so Code Interpreter can run.

Then invoke Code Interpreter.

Search for this existing file:

/mnt/data/ci_post_revoke_test_20261002.csv

Do not create, copy, rename, overwrite, or modify it.

Return only:
- File visible: Yes/No
- Full path
- File size in bytes
- SHA-256 hash
```

Why this matters:
- If the artifact was inaccessible during revocation but the same SHA-256 appears after restoration, it likely persisted while execution/access was gated.
- If it does not reappear, a cleanup, sandbox replacement, or other lifecycle event may have occurred. Do not infer the exact mechanism without evidence.

---

## Optional Test 10 — Time-based retention

If the artifact survives, repeat the exact hash check after:
- 1 hour
- 6 hours
- 24 hours
- 48 hours
- 7 days

Record only observed persistence. Do not turn POC behavior into a retention SLA.

---

## Results matrix

| Test | Source access | Chat | CI executes? | Artifact visible? | Hash matches? |
|---|---|---|---|---|---|
| Baseline | Yes | Existing | N/A | N/A | N/A |
| Artifact creation | Yes | Existing | Yes | Yes | Baseline |
| Pre-revoke check | Yes | Same | Yes | Yes | Yes |
| Source check after revoke | No | Same | N/A | N/A | N/A |
| Artifact after revoke | No | Same | TBD | TBD | TBD |
| Artifact after revoke | No | New | TBD | TBD | TBD |
| Restored-access check | Yes | New | TBD | TBD | TBD |

## Evidence metadata template

```markdown
- Date:
- Time:
- Test user:
- Workspace role:
- Source permission before:
- Source permission after:
- RLS/OLS configuration:
- Chat type: Same / New
- Code Interpreter invoked: Yes / No
- Artifact filename:
- Artifact SHA-256:
- Result:
- Notes:
```

## Do not commit to GitHub
- access tokens
- client secrets
- cookies
- Authorization headers
- tenant secrets
- connection strings
- real sensitive business data

Redact screenshots before publishing if needed.

## Final reporting rule
A failed Code Interpreter invocation is **not proof that the artifact was deleted**. Report source authorization, CI execution, file visibility, and hash comparison as separate observations.
