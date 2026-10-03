# 02 — Governed Semantic Model → Code Interpreter Flow

## Goal
Validate the execution path from governed semantic-model retrieval into Code Interpreter.

## Observed path

```text
Power BI Semantic Model
    ↓
Generated DAX
    ↓
Authorized result
    ↓
Tool-result artifact under /mnt/data
    ↓
Code Interpreter
    ↓
Generated Python
    ↓
Derived CSV/TXT artifact
```

## Observed
- Semantic-model retrieval occurred before Python processing.
- Generated DAX was visible in Run Steps.
- Generated Python read the governed result artifacts under `/mnt/data`.
- Code Interpreter created a CSV and returned deterministic file metadata including size and SHA-256.

## Evidence
See [`evidence/`](evidence/) for the curated October 3 execution screenshots:
- artifact creation Run Steps
- artifact filename/size/hash result
- generated DAX
- generated Python using governed input

The artifact used by the later lifecycle test is documented in Test 05.
