# 04 — Runtime and Guardrails

## Goal
Inspect non-sensitive runtime metadata and observe Code Interpreter guardrails without attempting to bypass them.

## Observed runtime metadata
- Python 3.11.15
- Linux platform
- `/home/sandbox` working directory
- pandas 1.5.3
- NumPy 1.24.0
- matplotlib 3.6.3
- SciPy 1.14.1
- approximately 420 installed packages in the tested environment

## Guardrail observations
- environment-variable enumeration was refused
- outbound HTTPS was treated as disabled and the requested external call was not issued
- a controlled missing-column test returned a normal `KeyError` without sensitive runtime data in the final response

## Evidence
See [`evidence/`](evidence/) for the network, environment-variable, runtime/package, and controlled-error screenshots.

## Boundary
The network result is observed sandbox-policy behavior, not packet-level proof of network-layer enforcement.
