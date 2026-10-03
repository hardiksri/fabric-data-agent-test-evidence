# 02 — Governed Code Interpreter Flow

## Observed path

```text
Power BI Semantic Model
    ↓
Generated DAX
    ↓
Authorized result
    ↓
JSON artifact under /mnt/data
    ↓
Code Interpreter
    ↓
Python / pandas
    ↓
Derived CSV/TXT
```

The POC showed that semantic-model retrieval occurred before Python processing and that generated DAX/Python were visible in Run Steps.
