# 04 — Runtime and Guardrails

Observed runtime/package information included:
- Python 3.11.15
- Linux platform
- `/home/sandbox` working directory
- pandas 1.5.3
- NumPy 1.24.0
- matplotlib 3.6.3
- SciPy 1.14.1
- ~420 installed packages in the tested environment

Additional observations:
- environment-variable enumeration was refused
- outbound HTTPS was treated as disabled and the request was not issued
- a controlled missing-column error returned a normal `KeyError` without sensitive runtime data in the final response
