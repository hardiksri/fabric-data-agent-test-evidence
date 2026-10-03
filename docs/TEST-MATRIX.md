# Test Matrix

| Test | Result |
|---|---|
| OLS protected field unavailable to restricted user | PASS |
| Authorized Region query succeeds | PASS |
| Governed DAX executed before Python | PASS |
| Semantic-model result materialized under `/mnt/data` | PASS |
| Generated Python visible in Run Steps | PASS |
| Local CSV/TXT artifact creation | PASS |
| Same-user artifact persistence across chats | PASS in tested context |
| Cross-user artifact visibility | Not found by second user |
| Environment-variable enumeration | Refused |
| Outbound HTTPS | Treated as disabled; request not issued |
| Runtime/package metadata | Visible |
| Controlled Python error | No sensitive leakage observed |
| Source Read revoked → fresh query blocked | PASS |
| Same-chat artifact after revocation | Readable |
| New-chat artifact after revocation | Readable |
| Artifact after Read restored | Same size + SHA-256 |
| Older artifacts from earlier POC | Not found several days later |
| Exact retention interval | Unknown |
