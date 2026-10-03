# 01 — OLS Enforcement

## Goal
Validate that a restricted user can still query authorized model data while an OLS-protected field is unavailable.

## Observed
- `Company Name` was unavailable to the restricted user.
- `Total Sales by Region` succeeded.
- This supports OLS enforcement for the tested Fabric Data Agent → Power BI semantic model path.
