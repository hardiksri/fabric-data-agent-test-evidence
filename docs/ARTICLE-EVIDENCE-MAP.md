# Part 2 Article — Evidence Placement Map

This map connects the Fabric Data Agent governance article to the POC evidence in this repository.

## Figure 1 — Governed source retrieval before Python

Use evidence showing:
- generated DAX for `Total Sales by Region`
- semantic-model result
- generated Python reading the governed result artifact under `/mnt/data`

Article section: **Governed data was retrieved before Python executed**

Suggested caption:

> Governed semantic-model data is retrieved first and then passed to Code Interpreter for Python processing.

## Figure 2 — Same-user artifact persistence

Use evidence showing the previously created Code Interpreter artifact being found from another chat for the same user.

Article section: **Artifacts persisted across chats for the same user**

Suggested caption:

> A Code Interpreter artifact remained visible across a new chat for the same test user.

## Figure 3 — Cross-user isolation

Use the separate-user/admin check and `admin_cross_user_check_20261003.txt`.

Article section: **A second user did not see the first user's test artifact**

Suggested caption:

> In our test, a separate user could execute Code Interpreter but did not find the first user's artifact.

## Figure 4 — Runtime and package metadata

Use runtime/platform and installed-package screenshots.

Article section: **Runtime details were visible, but sensitive environment introspection was constrained**

Suggested caption:

> Basic runtime/package metadata was visible, while direct environment-variable enumeration was refused.

## Figure 5 — Outbound network policy

Use the HTTPS-request test.

Article section: **Outbound internet access was treated as disabled**

Suggested caption:

> Code Interpreter treated outbound internet access as disabled and did not issue the requested HTTPS call.

## Figure 6 — Controlled error handling

Optional. Use the missing-column `KeyError` evidence.

Suggested caption:

> A controlled Python failure returned a normal KeyError without exposing sensitive runtime details.

## Figure 7 — Post-revocation lifecycle

This is the strongest governance evidence in the POC.

Recommended sequence:
1. Semantic-model Read permission removed.
2. Fresh semantic-model query blocked.
3. Post-revocation content check shows the earlier CSV is still readable.
4. New-chat same-user check finds the same 104-byte artifact and SHA-256.
5. Cross-user/admin check returns `MATCH_COUNT: 0`.
6. Semantic-model Read restored.

Primary artifact SHA-256:

`f50ce54091d0e1898d53637c8bd42ff36aba75a25d7e859a574ae97df34bfede`

Suggested caption:

> Revoking semantic-model Read blocked new source queries, but the same user's previously materialized Code Interpreter artifact remained readable across a new chat. A different user did not find the artifact.

## Article wording boundary

Use:

> In our POC...

Do not convert the observed behavior into a Fabric-wide retention guarantee. The earlier-artifact check returned `MATCH_COUNT: 0`, so the exact cleanup interval remains unknown.
