# Conformance

How to assess an AI operations system against [SPECIFICATION.md](SPECIFICATION.md), and how to state the result.

This document is normative for anyone who claims conformance. The key words are used as section 2 of the specification defines them.

Conformance is self-assessed. Nobody certifies it, including the maintainers of this specification. What makes a claim credible is that it is public, specific and testable: for each requirement, the statement says what was done and how it was verified, so that anyone can check.

## What "conforms" means

A system conforms to a given version of zero-trust AI operations when its published statement shows all of the following:

- Every MUST and MUST NOT requirement of that version is met, in the configuration the statement names.
- Every SHOULD requirement that is not met is listed, with the reason.
- Every value the specification asks the system to document is given (see [Documented values](#documented-values)).
- Each residual risk in [section 6](SPECIFICATION.md#6-residual-risk) of the specification is assessed for this system.
- The statement says how a customer can run the verification steps against the deployed system or an equivalent test environment, and, for any step that needs control of the vendor's own infrastructure, gives the vendor's evidence and how the customer can check it.

A requirement MUST NOT be marked **Met** without evidence from a verification step, or an equivalent test, run against the deployed system or an equivalent test environment. A design document alone is not enough. A system that meets some but not all MUST requirements does not conform; its statement SHOULD say which requirements it meets instead. An honest partial statement is worth more than a claim that is not.

### The conformance target

The statement MUST name the conformance target as section 2 of the specification defines it: the platform, the agent runtime that sends context to the model and acts on its output, and every interface the platform provides for people to see the AI's output. A vendor whose component works with agent runtimes it does not provide MUST name the runtime and configuration it assessed.

### Configurable systems

A system conforms only in the configurations its statement names. The statement MUST list every setting that takes the system out of conformance (R0.3), such as giving the AI open internet access or letting it change systems without Rule 4.

### Test environments

Where a customer cannot test against production, an equivalent test environment MAY be used. The statement MUST explain how the test environment is kept equivalent to the deployed system.

### Draft versions

A statement against a draft, such as 1.0.0-draft.1, MUST name the draft. A statement against a draft does not carry over to the released version.

### Claims about records

A statement, and any description of the system, MUST NOT describe its records as immutable or tamper-proof unless a mechanism makes them so, and MUST name that mechanism.

### Corrections and withdrawal

If someone reports a reproducible failure of a requirement the statement marks as Met, the statement MUST be corrected or withdrawn within 30 days.

## How to assess

1. **Fix the scope.** Name the system, its version, its configuration, the runtime and interfaces it includes, and the managed systems it can reach.
2. **Run the verification steps.** Every rule in the specification ends with them. Run them against the system as deployed, or an equivalent test environment, in the way section 2 sets out: make the AI's requests and run code in its environment directly, then repeat each step through the AI with adversarial instructions where possible. The direct run decides the result. A step that the model declines is inconclusive, never a pass.
3. **Record evidence for each requirement.** A test you ran and its result is the evidence. Passing the steps is necessary, not sufficient: where a requirement asks for more than its steps check, say how you verified the rest.
4. **Give the documented values** listed below.
5. **Assess the residual risks.** For each risk in section 6, say how it applies to this system and what limits it.
6. **Publish the statement.** The statement MUST use the template below, MUST be dated, and SHOULD be linked wherever the claim is made.
7. **Reassess.** The statement MUST be reassessed when the system changes in a way that touches a requirement, and at least once a year.

## How to state it

Allowed, when the system conforms and the statement is published:

> Conforms to zero-trust AI operations 1.0.0-draft.1 (self-assessed, 2026-10-11, configuration "standard"). Statement: <https://example.com/your-conformance-statement>

A claim MUST NOT use "certified", "approved", "endorsed", or anything implying that the maintainers of this specification have reviewed or vouched for the system.

You MAY use this badge, with the version you conform to, linked to your statement:

```markdown
[![Zero-trust AI operations 1.0.0-draft.1: self-assessed](https://img.shields.io/badge/zero--trust%20AI%20operations-1.0.0--draft.1%20self--assessed-107d98)](https://example.com/your-conformance-statement)
```

## Statement template

### Requirements

Copy the table into your statement and fill in every row. Status is one of:

- **Met**, with evidence;
- **Not met**;
- **Not used**, for MAY provisions the system does not use;
- **Not applicable**, only where the condition the requirement depends on cannot arise in the named configuration, with evidence that the capability is absent. For example, R4.10 is not applicable to a system that offers no automations.

Requirements with both a MUST and a SHOULD part have a row for each.

| ID | Level | Requirement, in short | Status | Evidence |
| --- | --- | --- | --- | --- |
| R0.1 | MUST NOT | No requirement met by instructing the AI or by a model's judgement | | |
| R0.2 | MUST NOT | Nothing starts when a check it depends on, or recording, cannot complete | | |
| R0.3 | MUST | Settings that leave conformance documented, out of the AI's reach, shown when in effect | | |
| R1.1 | MUST | Every read authorised against current permissions, never above the mapped account; revocation within one hour | | |
| R1.2 | MUST NOT | Authorisation does not depend on assertions by the AI | | |
| R1.3 | MUST | Access never exceeds the requesting person's and ends with the session; changes within the approver's authority | | |
| R1.4 | MUST | Operations not offered are unavailable, not merely forbidden by instruction | | |
| R1.5 | MUST | Every request classified by deterministic rules; unclassified or free-form means change | | |
| R1.6 | MUST | Credential and factor creation or reset approved individually; values never reach the AI | | |
| R1.7 | MUST | Unattended sessions have a requesting person, a stated scope, current permissions, and stop when the person leaves | | |
| R1.7 (SHOULD) | SHOULD | Alert when a trigger stops; transfer of triggers | | |
| R1.8 | MUST | The organisation can restrict the AI below the requesting person's permissions | | |
| R1.8 (SHOULD) | SHOULD | The requesting person can restrict it too | | |
| R1.9 | SHOULD | Rate and volume of requests limited | | |
| R1.10 | MUST | Information reaches only people with access to it, unless deliberately shared; shared sessions use common permissions | | |
| R1.11 | MUST | Organisations isolated; content not sent to third parties on terms that allow training for others | | |
| R1.12 | MUST | One action, and separate actions, stop sessions, proposals, changes, automations and classes within 60 seconds | | |
| R1.13 | MUST | Sessions end within 24 hours or a shorter set maximum | | |
| R2.1 | MUST NOT | No managed-system credential anywhere the AI can read | | |
| R2.2 | MUST | The platform attaches credentials, after authorising each request | | |
| R2.3 | MUST | Credentials referenced by name, released after approval to the executor only; only success or failure returned | | |
| R2.3 (SHOULD) | SHOULD | The platform attaches credentials to changes itself | | |
| R2.4 | MUST | Session identity bound, accepted only by the platform, never forwarded, ends with the session, unusable if copied | | |
| R2.4 (SHOULD) | SHOULD | Session identity sender-constrained even when unreadable | | |
| R2.5 | MUST NOT | The AI's runtime credentials not readable by its tools or code | | |
| R2.6 | MUST | Credential values replaced: platform-held values and encodings, marked fields, key material, listed formats, created credentials | | |
| R2.6 (SHOULD) | SHOULD | Credential values detected and replaced in other data | | |
| R3.1 | MUST | The platform is the only route to managed systems; no access to the organisation's networks | | |
| R3.2 | MUST | The AI's code and browser sessions have no network, including DNS, beyond the platform | | |
| R3.3 | MUST | Every external channel removed, allow-listed with stated audience and checked, or approved per message | | |
| R3.4 | MUST | No automatic fetch of addresses in the AI's output; links shown safely; delivery defanged | | |
| R3.5 | MUST | Information from managed systems cannot determine public retrieval beyond a query to a disclosed service and a choice among permitted results | | |
| R3.6 | MUST | Every third party documented; accounts and endpoints chosen by the platform | | |
| R3.6 (SHOULD) | SHOULD | Organisation chooses third parties and can use its own accounts | | |
| R3.7 | MUST | Platform-run change code limited to targets and declared destinations; no credential to code on targets | | |
| R3.7 (SHOULD) | SHOULD | Target network restricted where possible | | |
| R3.8 | MUST | Text from the AI and managed systems rendered and delivered as inert data | | |
| R4.1 | MUST | Every change the AI initiates is a proposal that waits for approval, except within automations and classes | | |
| R4.2 | MUST | Proposals complete, including timing, retries and safe steps; artefacts pinned or captured; plans verified | | |
| R4.3 | MUST | Approval bound to a SHA-256 or stronger digest, rendered from it, single use, retries bounded, verified at execution | | |
| R4.4 | MUST | Targets bound by identifier or platform-computed fingerprint and verified at execution with stated preconditions | | |
| R4.4 (SHOULD) | SHOULD | Destructive changes required to state preconditions | | |
| R4.5 | MUST | Approvals expire within 30 days or a shorter set period | | |
| R4.6 | MUST | Only a person with permission to make the change authorises it; viewers and approvers have access to the session's sources | | |
| R4.7 | MUST | User verification bound to each proposal's digest and a fresh value; authenticators restrictable; enrolment protected | | |
| R4.7 (SHOULD) | SHOULD | Attestation checked; separate-device confirmation that shows the proposal | | |
| R4.8 | MUST | Sets shown in full and individually rejectable; sensitive and high-impact changes approved alone | | |
| R4.8 (SHOULD) | SHOULD | Size of sets limited | | |
| R4.9 | MUST | Approval view shows everything R4.9 lists, from the digested bytes and the platform's records, with AI text marked | | |
| R4.10 | MAY / MUST | Automations started only by fixed, AI-independent triggers, rate-limited, with people-set attributes, expiring within a year | | |
| R4.10 (SHOULD) | SHOULD | Alert when an automation stops | | |
| R4.11 | MUST NOT | No change without per-instance approval outside a bounded, enumerated, deterministic class that expires within a year | | |
| R4.12 | MUST | Instruction stores documented; outside writes need approval; AI memory marked and kept out | | |
| R4.13 | MAY / SHOULD | Separate or multiple approvers for kinds of change the organisation defines | | |
| R4.14 | MUST | AI-initiated configuration changes are approved proposals; direct ones need user verification | | |
| R4.15 | SHOULD | Freeze periods | | |
| R5.1 | MUST | A complete decision record for every proposal and every automated or class change | | |
| R5.2 | MUST | A record of every request made or refused, with parameters | | |
| R5.2 (SHOULD) | SHOULD | Content returned to the AI kept for a documented period | | |
| R5.3 | MUST | Every instruction, output, instruction-store version and model version recorded | | |
| R5.4 | MUST NOT | The AI cannot modify or delete records | | |
| R5.5 | MUST | Text kept inert in records, interfaces and exports, with AI text marked | | |
| R5.6 | MUST | Records kept at least one year, never removed by volume; legal exceptions recorded | | |
| R5.7 | MUST | Access to records limited as R1.10 requires | | |
| R5.8 | SHOULD | Records tamper-evident | | |
| R5.9 | MUST | Records exportable to the organisation's logging or SIEM | | |

### Documented values

| Requirement | Value |
| --- | --- |
| R0.3 | Every setting that takes the system out of conformance |
| R1.1 | The method used to determine permissions for each kind of managed system, and the maximum delay between a revocation at its source and its enforcement |
| R1.5 | The tool servers the organisation has designated as trusted, in the assessed configuration |
| R2.6 | The encodings and data formats in which credential values are replaced |
| R3.3 | The allow-list entries whose readers the platform cannot check |
| R3.6 | Every third party, with account, region, retention, use for training, and whether the account is shared |
| R4.4 | The fingerprint rules for each operation whose targets lack stable identifiers |
| R4.7 | The out-of-band process for registering authenticators, if one is used |
| R4.12 | Every instruction store |
| R5.2 | The period for which content returned to the AI is kept, if it is |
| R5.6 | Any shorter retention period set to meet a legal obligation, and the volume of records the system has been tested to hold |

The statement SHOULD also give the values the organisation has set in the assessed configuration for R1.13, R4.5, R4.10, R4.11 and R5.6.

### Residual risk

For each residual risk in section 6 of the specification:

| Residual risk | How it applies to this system | What limits it |
| --- | --- | --- |
| 1. Misleading conclusions and forged evidence | | |
| 2. Harm a person approves | | |
| 3. Harm within automations and classes | | |
| 4. Breadth of the requesting person's access | | |
| 5. Disclosure to the disclosed third parties | | |
| 6. Signalling through permitted choices | | |
| 7. Links a person opens | | |
| 8. Software in the person's own session | | |
| 9. Code that runs on a managed system | | |
| 10. Credentials the platform cannot recognise | | |
| 11. What has already been read | | |
| 12. Load, cost, volume and availability | | |
| 13. Compromise outside the AI | | |

## Buying instead of building

If you are evaluating a product rather than building one, ask the vendor for their conformance statement, and run the verification steps yourself during a trial, using the test access the statement describes. [CHECKLIST.md](CHECKLIST.md) turns the requirements into questions for a vendor call, each with what a good answer and a red flag sound like.
