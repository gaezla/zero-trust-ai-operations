# Conformance

How to assess an AI operations system against [SPECIFICATION.md](SPECIFICATION.md), and how to say what you found.

Conformance is self-assessed. Nobody certifies it, including the maintainers of this specification. What makes a claim credible is that it is public, specific and testable: each requirement says what you did and how you verified it, so anyone can check.

## What "conforms" means

A system **conforms to zero-trust AI operations 1.0** when it meets every MUST and MUST NOT requirement in the specification.

- **SHOULD** requirements that are not met must be listed, each with the reason.
- **MAY** provisions (pre-approved automations under R4.7, pre-approved classes under R4.8) need only be described if the system uses them.
- A system that meets some but not all MUST requirements does not conform. Say which requirements it meets instead; a partial statement that is honest is worth more than a claim that is not.

## How to assess

1. **Fix the scope.** Name the system, its version, and the managed systems it can reach.
2. **Run the verification steps.** Every rule in the specification ends with tests. Run them against the system as deployed, not against documentation.
3. **Record evidence for each requirement.** A test you ran and its result is the strongest evidence; a design document is weaker; a statement with nothing behind it is not evidence.
4. **Publish the statement.** Use the template below, dated, and link to it wherever you make the claim.
5. **Reassess** when the system changes in a way that touches a requirement, and at least once a year.

## How to state it

Allowed, when every MUST requirement is met and the statement is published:

> Conforms to zero-trust AI operations 1.0 (self-assessed, 2026-10-10). Statement: <https://example.com/your-conformance-statement>

Not allowed: "certified", "approved", "endorsed", or anything implying that the maintainers of this specification have reviewed or vouched for the system.

You may use this badge, linked to your statement:

```markdown
[![Zero-trust AI operations 1.0: self-assessed](https://img.shields.io/badge/zero--trust%20AI%20operations-1.0%20self--assessed-107d98)](https://example.com/your-conformance-statement)
```

## Statement template

Copy the table into your statement and fill in every row. Status is one of **Met**, **Not met** or **Not used** (for MAY provisions).

| ID | Level | Requirement, in short | Status | Evidence |
| --- | --- | --- | --- | --- |
| R1.1 | MUST | Every operation authorised against the requesting person, by the platform, at the time | | |
| R1.2 | MUST NOT | Authorisation does not depend on anything the AI produces | | |
| R1.3 | MUST NOT | No access that outlasts the session or exceeds the person's | | |
| R1.4 | MUST | Operations not offered are unavailable, not merely forbidden by instruction | | |
| R1.5 | SHOULD NOT | Authentication-management endpoints not offered to the AI | | |
| R1.6 | MUST | Sessions with no person present limited to reads, apart from pre-approved automations | | |
| R2.1 | MUST NOT | No managed-system credential anywhere the AI can read | | |
| R2.2 | MUST | The platform attaches credentials, after authorising each operation | | |
| R2.3 | MUST | Secrets for a change released after approval, only to its executor, removed from output | | |
| R2.4 | MUST | The AI's session identity bound to the session and person, not reusable, ends with the session | | |
| R2.5 | SHOULD NOT | The AI's own runtime credentials not readable by its tools or code | | |
| R3.1 | MUST | Managed systems reached only through the platform; no access to private networks | | |
| R3.2 | MUST | The AI's code has no network beyond the platform's interface | | |
| R3.3 | MUST | Every external-communication channel removed, allow-listed or approved as a change | | |
| R3.4 | MUST | Public retrieval cannot carry information from managed systems out | | |
| R3.5 | MUST / SHOULD | Model provider disclosed (MUST); chosen by the organisation (SHOULD) | | |
| R4.1 | MUST | Every change the AI initiates is a proposal that waits for a person | | |
| R4.2 | MUST | Approval bound to the exact content and verified at execution | | |
| R4.3 | MUST | The approval interface refuses the AI, sub-agents, unattended jobs and machine identities | | |
| R4.4 | MUST | The approver sees the exact content, not only a summary | | |
| R4.5 | MUST / SHOULD | Channel and assurance recorded (MUST); second factor offered for client-side approval (SHOULD) | | |
| R4.6 | MUST | Read or change decided by the platform from the effect, never by the AI | | |
| R4.7 | MAY / MUST NOT | Pre-approved automations allowed; the AI cannot create, modify, widen or retarget them | | |
| R4.8 | MAY | Pre-approved classes, only if people define them and the platform decides membership | | |
| R4.9 | SHOULD | Writes to what the AI later reads as instructions treated as changes | | |
| R5.1 | MUST | A complete decision record for every change | | |
| R5.2 | SHOULD | Reads recorded: what, from where, for whom | | |
| R5.3 | MUST NOT | The AI cannot modify or delete records | | |
| R5.4 | SHOULD | Records exportable to the organisation's own logging or SIEM | | |
| R5.5 | MUST NOT / MUST | No "immutable" claim without a mechanism; retention documented | | |

## Buying instead of building

If you are evaluating a product rather than building one, ask the vendor for their conformance statement and run the verification steps yourself during a trial. [CHECKLIST.md](CHECKLIST.md) turns the requirements into questions for a vendor call, each with what a good answer and a red flag sound like.
