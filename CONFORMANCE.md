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

### Who issues a statement

A vendor MAY issue a statement for a product in a named configuration, and an organisation MAY issue one for its deployment. Requirements that depend on facts of a deployment, such as allow-list entries (R3.3), trusted tool servers (R1.5), designated just-in-time processes and shortened retention (R5.6), are completed by the organisation for its deployment; a vendor's statement says how the product supports them.

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
- **Organisation**, in a vendor's statement, for a requirement that the organisation completes for its deployment, with how the product supports it;
- **Not applicable**, only where the condition the requirement depends on cannot arise in the named configuration, with evidence that the capability is absent. For example, R3.5 is not applicable to a system that offers no public retrieval.

Requirements with both a MUST and a SHOULD part have a row for each.

| ID | Level | Requirement, in short | Status | Evidence |
| --- | --- | --- | --- | --- |
| R0.1 | MUST NOT | No requirement met by instructing the AI or by a model's judgement | | |
| R0.2 | MUST NOT | Nothing starts when a check it depends on, or recording, cannot complete | | |
| R0.3 | MUST | Settings that leave conformance documented, changed only by approved proposal, shown when in effect | | |
| R1.1 | MUST | Every request other than a change authorised against current permissions, never above the mapped account; revocation within one hour | | |
| R1.2 | MUST NOT | Authorisation does not depend on assertions by the AI | | |
| R1.3 | MUST | Permissions never exceed the requesting person's, end with the session and with the person's account; changes within the approver's authority | | |
| R1.4 | MUST | Operations not offered are unavailable, not merely forbidden by instruction | | |
| R1.5 | MUST | Every request classified by deterministic rules; unclassified or free-form means change | | |
| R1.5 (SHOULD) | SHOULD | Read-only credentials for reads | | |
| R1.6 | MUST | People's credential and factor creation or reset approved individually; values never reach the AI | | |
| R1.7 | MUST | Unattended sessions have a requesting person, a stated scope, current permissions, and stop when the account is disabled | | |
| R1.7 (SHOULD) | SHOULD | Alert when a trigger stops; transfer of triggers | | |
| R1.8 | MUST | The organisation can restrict the AI below the requesting person's permissions, down to resources | | |
| R1.8 (SHOULD) | SHOULD | The requesting person can restrict it too | | |
| R1.9 | SHOULD | Rate and volume of requests limited | | |
| R1.10 | MUST | Information reaches only people with access to it, unless deliberately shared; cross-organisation sharing permitted by the source; shared sessions use common permissions | | |
| R1.11 | MUST | Organisations isolated; content not sent to third parties on terms that allow training for others | | |
| R1.12 | MUST | Stop actions for everyone, per person, per provider and per operation, system or resource; new requests refused at once, actions complete within 60 seconds, except documented operations and named safe steps | | |
| R1.13 | MUST | Sessions end within 24 hours or a shorter set maximum | | |
| R2.1 | MUST NOT / MUST | No credential the platform uses placed where the AI can read; echoed credentials replaced | | |
| R2.2 | MUST | The platform attaches credentials to the requests it makes, after authorising each | | |
| R2.3 | MUST | Credentials referenced by name, released after approval to the executor only; only success or failure reaches the AI | | |
| R2.3 (SHOULD) | SHOULD | The platform makes a change's connections itself; released credentials per-execution, or rotated afterwards | | |
| R2.4 | MUST | Session identity bound, accepted only by the platform, never forwarded, ends with the session, unusable if copied | | |
| R2.4 (SHOULD) | SHOULD | Session identity sender-constrained even when unreadable | | |
| R2.5 | MUST NOT | The AI's runtime credentials not readable by its tools or code | | |
| R2.6 | MUST | Credential values replaced: values the platform holds, attaches or releases in listed encodings, marked fields, key material, headers, tokens, hashes, listed formats per kind of system, created credentials | | |
| R2.6 (SHOULD) | SHOULD | Credential values detected and replaced in other data | | |
| R3.1 | MUST | The platform is the only route to managed systems; no access to the organisation's networks | | |
| R3.2 | MUST | The AI's code and browser sessions have no network, including DNS, beyond the platform | | |
| R3.3 | MUST | Every external channel removed, allow-listed with stated audience and checked, or approved per message; destinations that act on content get only fixed content | | |
| R3.4 | MUST | No automatic fetch of addresses in the AI's output; links shown safely; delivery defanged; notifications carry no session content | | |
| R3.5 | MUST | Information from managed systems cannot determine public retrieval beyond a query to a disclosed, non-sharing service and a choice among permitted results; no AI-chosen host names | | |
| R3.6 | MUST | Every third party documented, including onward disclosure; accounts and endpoints chosen by the platform | | |
| R3.6 (SHOULD) | SHOULD | Organisation chooses third parties and can use its own accounts | | |
| R3.7 | MUST | Platform-run change code limited to targets and declared destinations; unchecked destinations marked; only per-execution credentials to code on targets | | |
| R3.7 (SHOULD) | SHOULD | Target network restricted where possible | | |
| R3.8 | MUST | Text the platform did not write rendered and delivered as inert data | | |
| R4.1 | MUST | Every change the AI initiates is a proposal that waits for approval, except within automations and classes | | |
| R4.2 | MUST | Proposals complete, including timing, retries and safe steps; every artefact pinned by digest; plans verified, and computing them governed | | |
| R4.3 | MUST | Approval bound to a SHA-256 or stronger digest of an unambiguous serialisation, rendered from it, single use, retries re-verified | | |
| R4.4 | MUST | Targets and the state the change depends on bound and verified at execution | | |
| R4.4 (SHOULD) | SHOULD | Destructive changes required to state preconditions | | |
| R4.5 | MUST | Approvals and their retries expire within 30 days or a shorter set period | | |
| R4.6 | MUST | Only a person who holds permission to make the change authorises it; viewers and approvers have access to the session's sources, or were deliberately shared with | | |
| R4.7 | MUST | User verification bound to each proposal's digest and a fresh value; authenticators restrictable; enrolment protected; disabled with the account | | |
| R4.7 (SHOULD) | SHOULD | Attestation checked; separate-device confirmation that shows the proposal | | |
| R4.8 | MUST | Sets shown in full and individually rejectable; sensitive and high-impact changes approved alone | | |
| R4.8 (SHOULD) | SHOULD | Size of sets limited | | |
| R4.9 | MUST | Approval view shows everything R4.9 lists, from the digested bytes and the platform's records, with AI text marked | | |
| R4.10 | MAY / MUST | Automations started only by fixed schedules or named event sources the AI cannot reach, rate-limited, with controlled attributes, expiring within a year, run under a person who holds permission | | |
| R4.10 (SHOULD) | SHOULD | Expiry and stop alerts; transfer of automations | | |
| R4.11 | MUST NOT | No change without per-instance approval outside a bounded, enumerated, deterministic class with controlled attributes that expires within a year | | |
| R4.11 (SHOULD) | SHOULD | Expiry alerts for classes | | |
| R4.12 | MUST | Instruction stores documented; outside writes need approval; AI memory marked, kept out and never evidence | | |
| R4.13 | MAY / SHOULD | Separate or multiple approvers for kinds of change the organisation defines | | |
| R4.14 | MUST | The AI changes configuration only by approved proposal; direct and repository changes confirmed by an eligible person; only people may disable, and it is recorded | | |
| R4.15 | SHOULD | Freeze periods | | |
| R5.1 | MUST | A complete decision record for every proposal, automated or class change, configuration change and disabling | | |
| R5.2 | MUST | A record of every request made or refused, with parameters | | |
| R5.2 (SHOULD) | SHOULD | Content returned to the AI kept for a documented period | | |
| R5.3 | MUST | Every instruction, output, instruction-store version and content, and model version recorded | | |
| R5.4 | MUST NOT | The AI cannot modify or delete records | | |
| R5.5 | MUST | Text the platform did not write kept inert in records, interfaces and exports, with AI text marked | | |
| R5.6 | MUST | Records kept at least one year, never removed by volume; legal and collective-agreement exceptions recorded; holds; export or deletion at end of service | | |
| R5.7 | MUST | Access to records limited as R1.10 requires; a person's sessions private by default | | |
| R5.8 | SHOULD | Records tamper-evident | | |
| R5.9 | MUST | Continuous, machine-readable export to the organisation's logging or SIEM within a documented delay | | |

### Documented values

| Requirement | Value |
| --- | --- |
| R0.3 | Every setting that takes the system out of conformance |
| R1.1 | The method used to determine permissions and just-in-time eligibility for each kind of managed system, and the maximum delay between a revocation at its source and its enforcement |
| R1.5 | The tool servers the organisation has designated as trusted, in the assessed configuration |
| R1.12 | The operations whose processes the platform cannot terminate on targets |
| R1.6 | The operations treated as creating or resetting people's credentials or factors, or weakening authentication |
| R2.6 | The encodings in which credentials the platform holds, attaches or releases are replaced, the forms of key material recognised, and the files and formats parsed for each kind of managed system |
| R3.3 | The allow-list entries whose readers the platform cannot check |
| R3.6 | Every third party, with account, region, retention, use for training, fine-tuning or evaluation, onward disclosure, and whether the account is shared |
| R4.3 | The canonical serialisation used for digests |
| R4.4 | The fingerprint rules for each operation whose targets lack stable identifiers |
| R4.7 | The process for registering authenticators other than with an existing one, if one is used |
| R4.4 | The state-binding rules for each operation |
| R4.10 | The rules that match trigger events to requests the platform made for the AI |
| R4.11 | The record of authorship from which the platform establishes who set selecting attributes held on managed systems |
| R4.12 | Every instruction store |
| R5.2 | The period for which content returned to the AI is kept, if it is |
| R5.6 | The volume of records the system has been tested to hold |
| R5.9 | The maximum delay of the continuous export |
| Holds permission (section 3) | The just-in-time processes the organisation designates |

The statement SHOULD also give the values the organisation has set in the assessed configuration for R1.9, R1.13, R4.5, R4.8, the R4.9 period, R4.10, R4.11 and R5.6, and its high-impact designations.

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
| 11. What has already been read, and changing audiences | | |
| 12. Load, cost, volume and availability | | |
| 13. Compromise outside the AI | | |
| 14. Systems that act on what they receive or observe | | |
| 15. Instructions carried to other AI software | | |

## Buying instead of building

If you are evaluating a product rather than building one, ask the vendor for their conformance statement, and run the verification steps yourself during a trial, using the test access the statement describes. [CHECKLIST.md](CHECKLIST.md) turns the requirements into questions for a vendor call, each with what a good answer and a red flag sound like.
