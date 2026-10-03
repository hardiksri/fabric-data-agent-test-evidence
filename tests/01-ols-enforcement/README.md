# 01 — OLS Enforcement

## Goal
Validate that a restricted user can query authorized model data while an OLS-protected field remains unavailable.

## Test setup
- Restricted user is not assigned a workspace role.
- Data Agent access remains available.
- Semantic model access is Read-only.
- User is assigned to `DataAgent_OLS_Test`.
- OLS protects `Customer[Company Name]`.

## Observed
- `Company Name` was unavailable to the restricted identity.
- `Total Sales by Region` succeeded.
- This supports OLS enforcement for the tested Fabric Data Agent → Power BI semantic model path.

## Evidence
See [`evidence/`](evidence/) for the curated permission-state, OLS-role, protected-field, and authorized-query screenshots.

## Scope boundary
This is evidence for the tested semantic-model query path only. It should not be generalized to separate Lakehouse, Warehouse, or other source routes without testing those paths independently.
