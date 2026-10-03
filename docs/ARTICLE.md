# Microsoft Fabric Data Agent Governance: From POC to Production

*Securing the data is only the beginning. Production Data Agents also need controlled identities, lifecycle management, evaluation, privacy, networking, and change governance.*

**Co-Authors — praful potphode & Natarajan Manivasagan**

In Part 1, the focus was simple:

> **Who can access what?**

We looked at identity, source permissions, RLS, CLS, OLS, query-path security, and why AI instructions should never become the authorization layer.

But a Data Agent can be perfectly secure and still be poorly governed.

Once we move from a POC/Development environment into production, a different set of questions appears:

- **Who can modify the agent?**
- **Which identity is used when the agent is called from another application?**
- **How do we test an instruction change before releasing it?**
- **What happens when the runtime changes?**
- **What changes when Code Interpreter is enabled?**
- **Where does conversation history live?**
- **How do networking and data residency affect the design?**
- **What happens when the Data Agent leaves Fabric?**

That is where the conversation moves from **security** into **governance**.

---

## Security and Governance Are Not the Same Thing

A useful way to separate them is:

```text
SECURITY

Who can access the agent?
Who can access the data?
What rows, columns, or objects can they see?


GOVERNANCE

Who can change the agent?
How are changes tested?
Which runtime is used?
How is quality measured?
Where is conversation data stored?
How is the agent consumed outside Fabric?
```

Security defines the boundary.

Governance controls how the system is operated inside that boundary.

Both are required for production.

---

## Data Agent Querying Is Read-Oriented — With One Important Nuance

The normal Fabric Data Agent source-query path is designed for analytical retrieval.

Its SQL, DAX, and KQL tools are used to retrieve and analyze information rather than modify the underlying data source.

So a prompt such as:

```text
Delete all customers with zero revenue.
```

should not turn the Data Agent into a transactional database administrator.

That is an important characteristic because it reduces the risk compared with an autonomous agent that has unrestricted write-capable database tools.

But there is an important nuance.

---

## Code Interpreter Introduces Another Execution Surface

Fabric Data Agent can optionally use **Code Interpreter**.

This allows the agent to generate and execute Python in a Microsoft-managed sandbox for tasks such as:

```text
Statistics
Advanced calculations
Data transformations
Charts
Analytical processing
```

That does **not** mean the underlying Fabric source suddenly becomes write-enabled.

A better mental model is:

```text
Underlying source
      │
      │ Governed retrieval
      ▼
Fabric Data Agent
      │
      ▼
Sandboxed Python execution
      │
      ▼
Calculated / transformed / visualized result
```

The distinction matters.

> **The source-query path remains governed and read-oriented, while Code Interpreter introduces an additional analytical execution environment.**

Rather than treating the word *sandbox* as sufficient evidence, we tested the execution path directly in our POC.

### What We Observed in Our POC

All screenshots, prompts, generated DAX/Python, hashes, and permission-transition evidence referenced below are published in the [Fabric Data Agent test evidence repository](https://github.com/hardiksri/fabric-data-agent-test-evidence).


#### 1. Governed data was retrieved before Python executed

We first asked the Data Agent to retrieve **Total Sales by Region**.

The Run Steps showed the semantic model being queried with DAX before Code Interpreter ran:

```dax
EVALUATE
SUMMARIZECOLUMNS(
    'Location'[Region],
    "Total Sales", [Total Sales]
)
ORDER BY [Total Sales] DESC
```

The authorized result was then materialized as a local JSON artifact under `/mnt/data`, and the generated Python read that result for further processing with pandas.

Conceptually, the observed path was:

```text
Power BI Semantic Model
        ↓
Governed DAX query
        ↓
Authorized result
        ↓
/mnt/data/...result_*.json
        ↓
Code Interpreter
        ↓
Python / pandas
        ↓
Derived result
```

This was an important governance finding: in the execution path we tested, Code Interpreter was not independently querying the semantic model. The Data Agent retrieved governed data first and then passed the result into the Python environment.

**POC evidence:** [Generated DAX](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/02-governed-code-interpreter-flow/evidence/22-generated-dax.png) · [Generated Python reading governed result artifacts](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/02-governed-code-interpreter-flow/evidence/23-generated-python-governed-input.png) · [Full governed execution test](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/02-governed-code-interpreter-flow/README.md)

*Evidence caption: Governed semantic-model data is retrieved first and then passed to Code Interpreter for Python processing.*

#### 2. The sandbox could create local artifacts

The generated Python created files such as:

```text
/mnt/data/total_sales_region_summary.csv
/mnt/data/total_sales_input_file_inspection.txt
```

The same Run Steps also exposed the generated Python used to read, transform, and write those artifacts.

That gives us visibility into both the calculation and the intermediate processing path rather than only the final natural-language answer.

#### 3. Artifacts persisted across chats for the same user

We created a CSV in one Code Interpreter run and then checked for it again.

The file was visible:

```text
Same user
Chat 1 → file created
Chat 1 → file still visible
New chat → previous file still visible
```

So in our POC, starting a new Data Agent chat did **not** necessarily mean starting with an empty `/mnt/data` working area for that user.

We do not infer a permanent retention period from this test—the exact lifetime remains unknown.

**POC evidence:** [Same-chat artifact visibility](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/03-sandbox-persistence-and-isolation/evidence/04-same-chat-artifact-visible.png) · [New-chat artifact found](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/03-sandbox-persistence-and-isolation/evidence/07-new-chat-artifact-found.png) · [Persistence/isolation test notes](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/03-sandbox-persistence-and-isolation/README.md)

*Evidence caption: A Code Interpreter artifact remained visible across a new chat for the same test user.*

#### 4. A second user did not see the first user's test artifact

We then tested user isolation.

User A created:

```text
ci_isolation_user_a_9282026.txt
```

User B ran Code Interpreter separately, created:

```text
ci_isolation_user_b_9282026.txt
```

and enumerated files whose names contained `ci_isolation_user_`.

The result for User B contained only:

```text
ci_isolation_user_b_9282026.txt
Match count: 1
```

User A's artifact was not returned.

This provides **POC evidence consistent with user-level isolation in the environment we tested**. It should not be generalized into a claim about every tenant, future runtime, or Microsoft's internal isolation implementation.

**POC evidence:** [User A isolation artifact created](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/03-sandbox-persistence-and-isolation/evidence/09-user-a-isolation-artifact-created.png) · [User B sees only its own artifact](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/02-governed-code-interpreter-flow/evidence/12-user-b-own-artifact-only.png) · [Cross-user post-revocation check: MATCH_COUNT 0](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/admin_cross_user_check_20261003.txt)

*Evidence caption: In our tests, a separate user could execute Code Interpreter but did not find the first user's artifact.*

#### 5. Runtime details were visible, but sensitive environment introspection was constrained

Code Interpreter exposed normal runtime metadata, including:

```text
Python: 3.11.15
Platform: Linux 6.6.150.1-azl3-x86_64-with-glibc2.36
Working directory: /home/sandbox
pandas: 1.5.3
NumPy: 1.24.0
Installed package count: 420
matplotlib: 3.6.3
SciPy: 1.14.1
```

However, when we requested **environment-variable names only**—explicitly excluding values, credentials, tokens, and secrets—the interaction was refused before Code Interpreter execution.

That is useful evidence that general runtime information can be inspected while direct environment-variable enumeration was constrained in our tested experience.

**POC evidence:** [Runtime and platform versions](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/04-runtime-and-guardrails/evidence/16-runtime-platform-versions.png) · [Installed package summary](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/04-runtime-and-guardrails/evidence/17-installed-package-summary.png) · [Environment-variable enumeration refused](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/04-runtime-and-guardrails/evidence/15-environment-variable-enumeration-refused.png)

*Evidence caption: Basic runtime/package metadata was visible, while direct environment-variable enumeration was refused.*

#### 6. Outbound internet access was treated as disabled

We explicitly instructed Code Interpreter to perform one HTTPS request to `https://example.com`.

The generated execution did **not** issue the request. Instead, the environment reported that outbound internet access was disabled.

So our observed result was:

```text
External HTTPS request requested
        ↓
Code Interpreter invoked
        ↓
Request not issued
        ↓
Environment reports outbound internet disabled
```

This is useful sandbox-policy evidence, but it is important to be precise: because no connection was actually attempted, this is **not independent packet-level verification of a network-layer block**.

**POC evidence:** [Network-egress policy test](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/04-runtime-and-guardrails/evidence/13-network-egress-policy-test.png)

*Evidence caption: Code Interpreter treated outbound internet access as disabled and did not issue the requested HTTPS call.*

#### 7. A controlled Python error did not expose sensitive details

Finally, we deliberately referenced a nonexistent DataFrame column:

```text
__TEST_MISSING_COLUMN__
```

The final result exposed only:

```text
exception class: KeyError
exception message: '__TEST_MISSING_COLUMN__'
```

In this controlled test, we did not observe credentials, connection strings, environment values, internal source secrets, or unrelated file contents in the returned error.

**POC evidence:** [Controlled KeyError test](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/04-runtime-and-guardrails/evidence/18-controlled-keyerror-test.png)

*Evidence caption: A controlled Python failure returned a normal KeyError without exposing sensitive runtime details in the returned result.*


#### 8. Revoking source access did not immediately remove already-materialized data

One follow-up question from the community was especially useful:

> What happens if a user exports an authorized result into Code Interpreter, and then that user's access to the semantic model is revoked?

We tested that lifecycle directly.

First, the restricted test user had **Read** access to the semantic model and successfully retrieved **Total Sales by Region**. Code Interpreter then created:

```text
ci_post_revoke_test_20261003_v2.csv
```

The artifact contained five data rows, was **104 bytes**, and had this SHA-256:

```text
f50ce54091d0e1898d53637c8bd42ff36aba75a25d7e859a574ae97df34bfede
```

We then removed that user's semantic-model **Read** permission while leaving Data Agent access intact.

A fresh source query was correctly blocked:

```text
Semantic Model Read removed
        ↓
Fresh Total Sales by Region query
        ↓
Source access denied
```

But when we returned to Code Interpreter, the previously materialized artifact was still readable.

The same user could read it in the existing chat, and a **new Data Agent chat for the same user** also found the artifact with the same file size and SHA-256.

We then tested a separate administrator/control identity. Code Interpreter executed successfully for that user, but the search returned:

```text
MATCH_COUNT: 0
```

Finally, we restored the original user's semantic-model Read permission. Fresh source queries worked again, and the earlier Code Interpreter artifact was still present with the same size and SHA-256.

The observed lifecycle was:

```text
Read granted
    ↓
Authorized source query
    ↓
Artifact materialized
    ↓
Read revoked
    ↓
New source query blocked
    ↓
Existing chat → artifact still readable
    ↓
New chat, same user → artifact still readable
    ↓
Different user → artifact not found
    ↓
Read restored
    ↓
Source query works again
    ↓
Same artifact still present
```

This exposed an important governance distinction:

> **Source authorization and the lifecycle of data already materialized into Code Interpreter are not the same control.**

In our tested environment, revoking source access prevented **new governed retrieval**, but it did not immediately invalidate the user's previously materialized sandbox artifact.

We also checked artifacts created during an earlier POC several days before this test. Those older files were no longer present. So this should **not** be interpreted as permanent storage or a retention guarantee.

The exact cleanup interval remains unknown.

**POC evidence:** [Read permission removed](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/21-permission-removal-confirmed.png) · [Fresh source query blocked](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/20-fresh-source-query-blocked.png) · [Post-revocation artifact content check](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/24-post-revoke-content-check.png) · [Same-user/new-chat hash check](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/new_chat_post_revoke_check_20261003.txt) · [Cross-user check: MATCH_COUNT 0](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/admin_cross_user_check_20261003.txt) · [Read restored](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/25-semantic-model-read-restored.png) · [Post-restore hash check](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/post_restore_artifact_check_20261003.txt) · [Older-artifact retention check](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/old_artifact_retention_check_20261003.txt) · [Full findings](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/docs/POST-REVOCATION-FINDINGS.md)

*Evidence caption: Revoking semantic-model Read blocked new source queries, but the same user's previously materialized Code Interpreter artifact remained readable across a new chat. A different user did not find the artifact.*

### What This Means for Production Governance

Our POC gives us a more useful model than simply saying *"Code Interpreter runs in a sandbox."*

For the environment and identities we tested, we observed:

| Area | POC observation |
| --- | --- |
| Semantic-model access | Governed query executed before Python |
| Generated DAX | [Visible in Run Steps](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/02-governed-code-interpreter-flow/evidence/22-generated-dax.png) |
| Generated Python | [Visible in Run Steps](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/02-governed-code-interpreter-flow/evidence/23-generated-python-governed-input.png) |
| Intermediate input | Materialized under `/mnt/data` |
| Local artifact creation | Confirmed |
| Same-chat persistence | Confirmed |
| Same-user, new-chat persistence | [Observed](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/03-sandbox-persistence-and-isolation/evidence/07-new-chat-artifact-found.png) |
| Source Read revoked | [Fresh semantic-model query blocked](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/20-fresh-source-query-blocked.png) |
| Previously materialized artifact after revocation | [Still readable by same user](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/24-post-revoke-content-check.png) |
| New chat after revocation | [Same artifact found with same size and SHA-256](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/new_chat_post_revoke_check_20261003.txt) |
| Cross-user visibility | [Different user did not find the artifact](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/admin_cross_user_check_20261003.txt) |
| Read restored | [Fresh source query worked; prior artifact still present](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/25-semantic-model-read-restored.png) |
| Older artifacts from earlier POC | [Not found several days later](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/05-post-revocation-artifact-lifecycle/evidence/old_artifact_retention_check_20261003.txt) |
| Runtime/package metadata | [Visible](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/04-runtime-and-guardrails/evidence/16-runtime-platform-versions.png) |
| Environment-variable enumeration | [Refused](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/04-runtime-and-guardrails/evidence/15-environment-variable-enumeration-refused.png) |
| Outbound internet | [Treated as disabled; request not issued](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/04-runtime-and-guardrails/evidence/13-network-egress-policy-test.png) |
| Controlled error leakage | [No sensitive leakage observed](https://github.com/hardiksri/fabric-data-agent-test-evidence/blob/main/tests/04-runtime-and-guardrails/evidence/18-controlled-keyerror-test.png) |
| Exact artifact retention duration | Not established |
| Internal sandbox/isolation technology | Not established |

These are **POC observations**, not universal guarantees.

The exact retention lifetime, internal sandbox technology, and implementation details should not be inferred beyond what the tests actually demonstrated.

For production, Code Interpreter should therefore be treated as a separate governed execution surface with its own validation checklist:

```text
What data reaches Python?
What code is generated?
What artifacts can be created?
How long can those artifacts remain available?
What happens to already-materialized data when source access is revoked?
Are users isolated from one another?
Can the environment access external networks?
What runtime information is exposed?
What appears in errors and diagnostics?
```

The key point remains:

> **Read-only source access does not mean there is only one execution surface.**

The reproducible prompts, hashes, permission transitions, screenshots, and machine-readable evidence from this POC are preserved in the public [fabric-data-agent-test-evidence GitHub repository](https://github.com/hardiksri/fabric-data-agent-test-evidence).

---

## The Calling Identity Becomes Critical Outside Fabric

When someone uses a Data Agent directly in Fabric, identity is relatively easy to reason about.

External applications make things more interesting.

A published Data Agent can participate in several patterns:

```text
Fabric user
     ↓
User identity


Custom application
     ↓
Delegated user identity


MCP client
     ↓
User token or service identity


Automation
     ↓
Service Principal
```

That means one architecture question should always be documented:

> **Whose identity ultimately reaches Fabric?**

Because Fabric security is evaluated against the effective identity that arrives at the service.

---

## Service Principals Are Useful — But They Change the Model

Service principals can be useful for:

```text
Automation
Background processing
Integration services
CI/CD
Custom applications
```

But a service principal has its **own permissions**.

Consider this pattern:

```text
User A
User B
User C
   │
   ▼
Shared Application
   │
   ▼
Single Service Principal
   │
   ▼
Fabric Data Agent
```

If every request runs using the same service identity, the underlying data platform evaluates that identity.

It does not automatically inherit User A, User B, or User C's individual data permissions.

That is very different from:

```text
User
 ↓
Application
 ↓
Delegated / On-Behalf-Of identity
 ↓
Fabric Data Agent
```

For interactive applications where per-user entitlements matter, delegated identity is usually much easier to reason about.

For unattended automation, a service principal may be exactly the right choice.

The key is not that one pattern is universally better.

The key is:

> **The identity model must be intentional.**

---

## Governance Starts Before the Agent Is Published

Who can change the Data Agent is almost as important as who can use it.

A creator can work with a draft version and refine things such as:

```text
Schema selection
Instructions
Example questions
Source configuration
AI behavior
```

Consumers can then use a published version.

That creates an important separation:

```text
Draft
 ↓
Development and testing


Published
 ↓
Approved consumer experience
```

This matters because instructions increasingly define analytical behavior.

A normal consumer should not need permission to modify:

```text
Revenue means Net Revenue excluding intercompany transactions.
```

just because they need to ask:

```text
What was revenue last month?
```

Production governance should separate:

```text
Who can consume?

Who can inspect?

Who can edit?

Who can publish?
```

Those are different responsibilities.

---

## Treat Data Agent Configuration Like Code

Once instructions define important business behavior, changing them is effectively a production change.

This:

```text
Change instruction
→ Save
→ Hope nothing broke
```

is not a sustainable governance process.

A stronger lifecycle looks like:

```text
Development
     ↓
Source control
     ↓
Peer review
     ↓
Test workspace
     ↓
Regression validation
     ↓
Production
     ↓
Publish
```

The goal is to be able to answer:

**Who changed this instruction?**

**What changed from the previous version?**

**Which version was tested?**

**Which version is running in production?**

**Which configuration produced the behavior we are investigating?**

Once a Data Agent becomes business-facing, its instructions, examples, source selections, and other configuration should be treated as governed application assets.

---

## Standard vs Preview Runtime Is a Governance Decision

Runtime selection should also be intentional.

A POC may use Preview capabilities because we want access to newer functionality.

A production environment may prefer the more stable runtime path.

But there is an important point:

> **Choosing a standard runtime does not mean the underlying AI stack will remain frozen forever.**

AI systems evolve.

That means governance cannot depend on:

```text
We tested it once.
Therefore it will behave exactly the same forever.
```

Regression testing has to become part of normal operation.

---

## Security Is Not the Same as Accuracy

A response can be perfectly secure and still be wrong.

Suppose the user asks:

```text
What did we spend on marketing last quarter?
```

Even if the user is fully authorized, the Data Agent still has to interpret:

- what "spend" means,
- what "last quarter" means,
- which accounts represent marketing,
- which source is authoritative,
- which filters should apply,
- and how the answer should be explained.

So production governance needs two separate questions:

```text
SECURITY

Was this user allowed to see the answer?


QUALITY

Was the answer actually correct?
```

Both matter.

---

## Native Evaluation Should Be Part of the Release Process

Testing should not stop at:

```text
We asked five questions.
The answers looked okay.
Ship it.
```

A better pattern is to maintain expected-answer test cases.

Conceptually:

```text
Business Question
      │
      ├──────────► Expected Answer
      │
      ▼
Fabric Data Agent
      │
      ▼
Generated Answer
      │
      ▼
Evaluation
      │
      ├── Pass
      ├── Incorrect
      └── Investigate
```

A useful evaluation set should include more than easy questions.

For example:

```text
Core KPI questions
Time-intelligence questions
Synonym variations
Ambiguous prompts
Cross-source questions
Expected failures
Restricted-data questions
Known edge cases
```

This changes testing from:

> "The demo looked good."

to:

> "The agent passed the scenarios we defined as important."

That is a much stronger production standard.

---

## Run Steps and Diagnostics Help Explain Failures

Not every incorrect answer is simply an AI hallucination.

A failure could happen because:

```text
Wrong source selected

Correct source
+
wrong query

Correct query
+
wrong business interpretation

Correct calculation
+
misleading explanation
```

Those are different problems.

Run Steps and diagnostics help us understand where the behavior came from.

That becomes particularly valuable when an agent can reason across multiple sources.

Instead of saying:

> "The AI got it wrong."

we can ask:

> **Where did the execution path diverge from what we expected?**

That is a much more useful operational question.

---

## Conversation History Is Part of the Data Estate

The conversation itself can contain sensitive information.

Consider:

```text
Why is revenue falling for Customer ABC
while we are negotiating their confidential renewal?
```

The revenue result may be ordinary business information.

The prompt may reveal something much more sensitive.

That means governance cannot focus only on the underlying tables.

We also need to think about:

```text
User prompts
Conversation context
Generated responses
Follow-up questions
Retention
Investigation access
```

Organizations should decide:

- What are users allowed to put into prompts?
- How long should conversational context persist?
- Who should be allowed to investigate interactions?
- Which information should never enter a conversational interface?

This becomes even more important once centralized auditing is introduced.

---

## Data Residency Needs an Explicit Decision

For regulated environments, the architecture review should document more than the location of the Fabric capacity.

It should ask:

```text
Where does the Fabric capacity live?

Where can AI processing occur?

Where can conversation history be stored?

Where will telemetry be stored?

Where will downstream applications process the response?
```

The full AI processing path matters.

It is not enough to say:

> "The data is in Fabric."

Governance needs to account for every service participating in the interaction.

---

## Networking Needs to Be Evaluated Per Source

Private Link and outbound-access controls are related, but they solve different problems.

Think of them like this:

```text
Private Link
      ↓
How do we privately reach a supported resource?


Outbound Access Protection
      ↓
Which external destinations may this workspace call?
```

Those controls should not be treated as interchangeable.

And not every Data Agent source follows exactly the same networking path.

A better architecture document looks like:

```text
Data Agent
   │
   ├── Semantic Model
   │       ↓
   │   Security / network path A
   │
   ├── Warehouse
   │       ↓
   │   Security / network path B
   │
   └── External SQL
           ↓
       Security / network path C
```

The same principle from Part 1 applies here:

> **Validate the actual path, not just the existence of a security feature.**

---

## What Happens When the Data Agent Leaves Fabric?

A Data Agent does not have to stay inside the Fabric user interface.

It can participate in broader architectures through services and patterns such as:

```text
Microsoft Foundry
Copilot Studio
Microsoft 365 Copilot
MCP clients
Custom applications
```

Once the response leaves Fabric, the receiving service introduces its own:

```text
Identity model
Security model
Retention policy
Logging behavior
Compliance boundary
Geographic processing
```

So:

```text
Secure inside Fabric
```

does not automatically mean:

```text
Secure everywhere downstream
```

The downstream service becomes part of the governance boundary.

---

## External Identity Can Change the Security Model

This becomes especially important when another application calls Fabric.

Compare these two patterns.

### User identity

```text
User
 ↓
External application
 ↓
Fabric as User
 ↓
User's Fabric + source permissions
```

### Shared or maker identity

```text
User
 ↓
External application
 ↓
Fabric as Shared Identity
 ↓
Shared identity's source permissions
```

Those are not equivalent architectures.

The second pattern may be intentional.

But it needs to be documented as a security and governance decision because the answer may be based on the shared identity's permissions rather than the individual consumer's permissions.

The question remains:

> **Whose identity reaches Fabric?**

---

## A Production Governance Model

After combining these controls, the broader architecture starts to look like this:

```text
IDENTITY
        ↓
AUTHORIZATION
        ↓
DATA SECURITY
        ↓
QUERY-PATH SECURITY
        ↓
AI SCOPE
        ↓
SEMANTIC GOVERNANCE
        ↓
EXECUTION GOVERNANCE
        ↓
QUALITY GOVERNANCE
        ↓
LIFECYCLE GOVERNANCE
        ↓
FABRIC DATA AGENT
```

Part 1 focused primarily on the upper security layers.

Part 2 adds the controls required to operate the agent as a production analytical product.

---

## Before Production, I Would Review These Areas

### Identity and authorization

- Confirm which identity reaches Fabric.
- Document user, delegated, and service-principal paths separately.
- Avoid accidentally replacing per-user security with one overly privileged shared identity.

### Data security

- Validate RLS, CLS, and OLS with real restricted identities.
- Validate every available query path independently.

### AI scope

- Expose only the sources, tables, and fields genuinely required.
- Do not use AI instructions as an authorization mechanism.

### Execution

- Document the selected runtime.
- Review Code Interpreter separately if enabled.
- Validate what data is passed into Python and what generated code does.
- Review artifact persistence and user-isolation behavior.
- Validate network and runtime-introspection restrictions.
- Treat Preview capabilities according to organizational risk policy.

### Quality

- Maintain expected-answer test cases.
- Regression-test critical business questions.
- Use Run Steps and diagnostics when behavior changes.

### Lifecycle

- Separate consumers from creators.
- Use draft and published versions.
- Promote changes through Development, Test, and Production.
- Version configuration and instructions.

### Privacy

- Review conversation-history behavior.
- Define what users may enter into prompts.

### Residency

- Document AI processing, storage, and telemetry locations.
- Include downstream services in the assessment.

### Network

- Review Private Link per source.
- Review outbound-access controls where appropriate.

### External consumption

- Re-run the governance review for Foundry, Copilot Studio, Microsoft 365 Copilot, MCP clients, and custom applications.
- Confirm the effective identity for every channel.

---

# Governance Still Does Not Tell Us What Happened

At this point, the Data Agent may be secure and well governed.

But another set of production questions remains:

**Who asked what?**

**What did the agent answer?**

**Which data source did it choose?**

**How long did the request take?**

**Where did execution fail?**

**Can compliance teams investigate an interaction?**

**Can operations teams monitor the agent over time?**

Those questions move us into three different disciplines:

```text
AUDIT

Who asked what,
and what did the agent answer?


TRACE

How did this particular request execute?


OBSERVE

Is the Data Agent healthy and behaving
as expected in production?
```

That became the next phase of our POC.

---

# Next: Part 3 — Auditing, Tracing and Observing Microsoft Fabric Data Agents in Production

Part 3 will focus on how we can combine capabilities such as:

```text
Microsoft Purview
DSPM for AI
Microsoft Foundry
Application Insights
Fabric diagnostics
Capacity telemetry
MCP / application telemetry
```

to understand not just:

> **What is the Data Agent allowed to do?**

but also:

> **What did it actually do?**

That is where security and governance start turning into **operational accountability**.
