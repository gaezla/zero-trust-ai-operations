# Conformance

How to assess an AI operations system against [SPECIFICATION.md](SPECIFICATION.md), and how to say what you found.

Conformance is self-assessed. Nobody certifies it, including the maintainers of this specification. What makes a claim credible is that it is public, specific and testable: each requirement says what you did and how you verified it, so anyone can check.

## What "conforms" means

A system **conforms to zero-trust AI operations** at a given version when all of the following hold:

- It meets every MUST and MUST NOT requirement in that version of the specification.
- Every SHOULD requirement it does not meet is listed, each with the reason.
- Its statement assesses each residual risk in [section 6](SPECIFICATION.md#6-residual-risk) of the specification for this system.
- Its statement says how a customer can run the verification steps themselves, including the test access to the AI's execution environment that section 2 of the specification relies on.

MAY provisions, such as pre-approved automations (R4.8), pre-approved classes (R4.9) and separation of approver and requester (R4.11), need only be described if the system uses them.

A system that meets some but not all MUST requirements does not conform. Say which requirements it meets instead; a partial statement that is honest is worth more than a claim that is not.

### Configurable systems

A system that can be configured conforms only in the configurations its statement names. The statement MUST list every setting that takes the system out of conformance, such as enabling open internet access for the AI, and the system SHOULD show people when it is running outside conformance.

### Draft versions

Statements against a draft, such as 1.0.0-draft.1, MUST name the draft. A draft statement does not carry over to the released version.

## How to assess

1. **Fix the scope.** Name the system, its version, its configuration, and the managed systems it can reach.
2. **Run the verification steps.** Every rule in the specification ends with tests. Run them against the system as deployed, not against documentation, and in the way section 2 of the specification sets out: issue the AI's tool calls and run code in its environment directly, then repeat each test through the AI with adversarial instructions. A model declining or failing to carry out an attempt is never a pass.
3. **Record evidence for each requirement.** A test you ran and its result is the strongest evidence; a design document is weaker; a statement with nothing behind it is not evidence.
4. **Assess the residual risks.** For each risk in section 6, say how it applies to this system and what limits it.
5. **Publish the statement.** Use the template below, dated, and link to it wherever you make the claim.
6. **Reassess** when the system changes in a way that touches a requirement, and at least once a year.

## How to state it

Allowed, when the system conforms and the statement is published:

> Conforms to zero-trust AI operations 1.0.0-draft.1 (self-assessed, 2026-10-11). Statement: <https://example.com/your-conformance-statement>

Not allowed: "certified", "approved", "endorsed", or anything implying that the maintainers of this specification have reviewed or vouched for the system.

You may use this badge, linked to your statement:

```markdown
[![Zero-trust AI operations: self-assessed](https://img.shields.io/badge/zero--trust%20AI%20operations-self--assessed-107d98)](https://example.com/your-conformance-statement)
```

## Statement template

Copy the table into your statement and fill in every row. Status is one of **Met**, **Not met** or **Not used** (for MAY provisions).

| ID | Level | Requirement, in short | Status | Evidence |
| --- | --- | --- | --- | --- |
| R0.1 | MUST | No requirement met by instructing the AI or by a model's judgement | | |
| R0.2 | MUST NOT | Nothing proceeds when a check it depends on cannot complete | | |
| R1.1 | MUST | Every operation authorised by the platform against the requesting person's current permissions | | |
| R1.2 | MUST NOT | Authorisation does not depend on anything the AI produces | | |
| R1.3 | MUST NOT / MUST | Access never exceeds the person's and ends with the session | | |
| R1.4 | MUST | Operations not offered are unavailable, not merely forbidden by instruction | | |
| R1.5 | MUST | Every operation classified as read or change by effect; unclassified means change | | |
| R1.6 | SHOULD NOT | Credential, factor, session and token management not offered to the AI | | |
| R1.7 | MUST | Unattended sessions limited to reads and proposals, re-authorised each run, stopped when the person's access ends | | |
| R1.8 | SHOULD | Rate and volume of operations limited per session and per system | | |
| R1.9 | MUST NOT / MUST | What the AI reads stays with people entitled to it; organisations isolated | | |
| R1.10 | MUST | Any session, or all of them, can be ended at once, with their access and identity | | |
| R2.1 | MUST NOT | No managed-system credential anywhere the AI can read | | |
| R2.2 | MUST | The platform attaches credentials, after authorising each operation | | |
| R2.3 | MUST | Secrets referenced by name; released after approval, only to the executor, removed from output | | |
| R2.4 | MUST | Session identity bound to session and person, ends with it, unusable if copied | | |
| R2.5 | MUST NOT | The AI's runtime credentials not readable by its tools or code | | |
| R2.6 | MUST / SHOULD | Secret values in structured read results replaced (MUST); in unstructured data (SHOULD) | | |
| R3.1 | MUST | Managed systems reached only through the platform; no access to the organisation's networks | | |
| R3.2 | MUST | The AI's code has no network, including DNS, beyond the platform's interface | | |
| R3.3 | MUST | Every external channel removed, allow-listed or approved as a change | | |
| R3.4 | MUST NOT / MUST | No automatic fetch of addresses in the AI's output, in any interface or delivery system | | |
| R3.5 | MUST NOT | The AI does not control the destination or content of public retrieval | | |
| R3.6 | MUST / SHOULD | Providers disclosed and accounts chosen by the platform (MUST); chosen by the organisation (SHOULD) | | |
| R4.1 | MUST | Every change the AI initiates is a proposal that waits for a person | | |
| R4.2 | MUST | Proposals complete, with every fetched dependency pinned by digest | | |
| R4.3 | MUST | Approval bound to a digest of the content and verified at execution | | |
| R4.4 | MUST / SHOULD | Targets by stable identifier and re-verified at execution (MUST); preconditions verified (SHOULD) | | |
| R4.5 | MUST | Approvals expire; a finite default applies | | |
| R4.6 | MUST | Only a person approves, confirmed by user verification that software cannot supply | | |
| R4.7 | MUST | Approver sees exact content, visible hidden characters, targets, expiry, platform evidence; AI text marked | | |
| R4.8 | MAY / MUST | Pre-approved automations allowed; changing one is a change; each run authorised and recorded | | |
| R4.9 | MAY | Pre-approved classes, defined by people, membership decided deterministically | | |
| R4.10 | MUST | Writes to trusted instructions are changes; AI memory labelled and treated as untrusted | | |
| R4.11 | MAY / SHOULD | Approver separate from requester, where the organisation requires it | | |
| R5.1 | MUST | A complete decision record for every change | | |
| R5.2 | MUST | A record of every operation, including reads | | |
| R5.3 | MUST NOT | The AI cannot modify or delete records | | |
| R5.4 | MUST | AI-written text kept as data and never interpreted by viewers or exports | | |
| R5.5 | SHOULD | Records tamper-evident | | |
| R5.6 | SHOULD | Records exportable to the organisation's logging or SIEM | | |
| R5.7 | MUST NOT / MUST | No "immutable" claim without a mechanism; retention documented | | |

Then, for each residual risk in section 6 of the specification:

| Residual risk | How it applies to this system | What limits it |
| --- | --- | --- |
| 1. Misleading conclusions | | |
| 2. Harmful changes a person approves | | |
| 3. Disclosure to the disclosed providers | | |
| 4. Low-bandwidth signalling | | |
| 5. Load and cost | | |
| 6. Compromise outside the AI | | |

## Buying instead of building

If you are evaluating a product rather than building one, ask the vendor for their conformance statement, and run the verification steps yourself during a trial with the test access the statement describes. [CHECKLIST.md](CHECKLIST.md) turns the requirements into questions for a vendor call, each with what a good answer and a red flag sound like.
