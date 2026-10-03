# Permission State and Source Revocation

## Baseline
- Restricted user: no workspace role
- Data Agent: Read access retained
- Semantic model: Read access granted
- OLS role: `DataAgent_OLS_Test`
- `Company Name` unavailable through the Data Agent
- `Total Sales by Region` query succeeded

## Revoked state
Semantic-model Read was removed while Data Agent access was kept.

A fresh semantic-model query returned that access to the Superstore semantic model was not currently granted and instructed the user to ask the workspace owner to grant access.

This confirmed that new source retrieval was blocked before testing the already-materialized Code Interpreter artifact.

## Restored state
Semantic-model Read was restored. A fresh `Total Sales by Region` semantic-model query succeeded again, and the previously materialized artifact was still present with the same size and SHA-256.
