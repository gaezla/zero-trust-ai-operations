# Zero-trust AI operations: specification

**Version** 1.0.0 · **Licence** [CC BY 4.0](LICENSE) · **Changes** [CHANGELOG.md](CHANGELOG.md)

This document is the normative text of zero-trust AI operations. The [README](README.md) explains the idea; this document says exactly what a system must do to meet it. [CONFORMANCE.md](CONFORMANCE.md) explains how to assess a system against it, and [CROSSWALK.md](CROSSWALK.md) maps every requirement to NIST SP 800-207, the OWASP Top 10 for LLM Applications 2026, the Agents Rule of Two and related work.

## Contents

1. [Scope](#1-scope)
2. [Conventions](#2-conventions)
3. [Terminology](#3-terminology)
4. [Threat model](#4-threat-model)
5. [The five rules](#5-the-five-rules)
   - [Rule 1. Act only as the person who asked](#rule-1-act-only-as-the-person-who-asked)
   - [Rule 2. Never give the AI a credential](#rule-2-never-give-the-ai-a-credential)
   - [Rule 3. Give the AI no network of its own](#rule-3-give-the-ai-no-network-of-its-own)
   - [Rule 4. Let only a person approve a change, exactly as it will run](#rule-4-let-only-a-person-approve-a-change-exactly-as-it-will-run)
   - [Rule 5. Record every decision where the AI cannot change it](#rule-5-record-every-decision-where-the-ai-cannot-change-it)
6. [Out of scope](#6-out-of-scope)
7. [References](#7-references)

## 1. Scope

This specification applies to **AI operations systems**: software in which an AI model investigates and changes IT systems, such as servers, hypervisors, network appliances, identity providers, cloud accounts, Kubernetes clusters and endpoints, on behalf of people.

It applies whether the system is bought or built in-house, whether the model is hosted or local, and whatever the AI is called: assistant, agent, copilot or AI SRE. It applies to read-only systems too: Rules 1, 2, 3 and 5 protect what the AI reads, not only what it changes.

It does not define how a model should behave. It defines what the system around the model must make impossible, on the assumption that the model will misbehave.

## 2. Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [[RFC2119]](https://www.rfc-editor.org/rfc/rfc2119) [[RFC8174]](https://www.rfc-editor.org/rfc/rfc8174) when, and only when, they appear in all capitals, as shown here.

Each requirement has an identifier, such as **R4.2**, that stays stable across minor versions. Each rule ends with **verification**: tests anyone can run, or ask a vendor to demonstrate, to check the requirements without reading source code.

## 3. Terminology

- **The AI.** The model and everything that acts on its output without a person in between: its tool calls, the code it writes and runs, its sub-agents, and anything it writes to memory.
- **Platform.** The deterministic software around the AI that authenticates people, mediates the AI's access to systems, holds credentials, runs approvals, executes changes and keeps records. The platform is not the AI, and no requirement in this document may be met by instructing the AI.
- **Person.** A human user who has authenticated to the platform as themselves.
- **Requesting person.** The person on whose behalf the AI is working in a session. For a scheduled session, the person who set it up.
- **Managed system.** Any system the AI can read or change through the platform.
- **Read.** An operation that returns information and changes nothing on a managed system or anywhere else.
- **Change.** An operation that changes state on a managed system, or that sends information to a destination outside the platform, such as an email, a message, a ticket or a webhook. Whether an operation is a read or a change is decided by its effect, not by its protocol: a request that uses HTTP `POST` to run a query is a read, and a request that uses `GET` to trigger an action is a change.
- **Proposal.** The complete, exact content of a change, prepared by the AI so that it can run without any further input from the AI: the operation and its parameters, the script or playbook, or the diff, together with its target, reason and evidence.
- **Approval.** A decision by a person that one specific proposal may run.
- **Pre-approved automation.** A change that a person has approved in advance to run later or repeatedly, with fixed content and a fixed scope, such as a scheduled playbook.
- **Credential.** Any secret that grants access: a password, API key, token, certificate private key, SSH key, session cookie or recovery code.
- **External communication.** Any path by which information can leave the platform's control other than an approved change: network requests, mail, messages, tickets, webhooks, files written to shared locations, and links or images rendered to a person, whose addresses can carry data.
- **Decision record.** The record of a change: who asked, what the AI proposed and why, who decided, how, and what ran with what result.

## 4. Threat model

**Assumption.** The AI can be wrong, or steered by an attacker, at any moment of any session. Everything the AI reads can carry instructions: log lines, alert text, ticket comments, hostnames, configuration files, documentation, web pages, tool output and its own memory. In the words of the OWASP Top 10 for LLM Applications 2026 (LLM01:2026 Prompt Injection), "no reliable prevention mechanism exists today", and "Defense is therefore architectural rather than interceptive."

An AI operations system cannot avoid two of the three properties that make this dangerous. In the terms of the Agents Rule of Two (Meta, 2025), it processes untrustworthy inputs [A] and has access to sensitive systems [B]. This specification keeps the third property, the ability to change state or communicate externally [C], out of the AI's hands: changes pass through a person (Rule 4), and external communication is removed, restricted or approved (Rule 3).

**Trusted.** The platform's deterministic code; the authentication of people; the access controls of the managed systems themselves; the person who approves a change, to the extent of reading what they approve.

**Security goals.** An AI that has been steered MUST NOT be able to:

- **G1** act beyond the authority of the person who asked (Rule 1);
- **G2** obtain or use a credential (Rule 2);
- **G3** reach a system except through the platform, or send what it has read anywhere other than the model provider the organisation knows about (Rule 3);
- **G4** change a managed system without a person approving that exact change (Rule 4);
- **G5** alter or erase the record of what happened (Rule 5).

NIST SP 800-207 named this risk in 2020. Section 5.7 warns that "an attacker will be able to induce or coerce an NPE [non-person entity] to perform some task that the attacker is not privileged to perform", and that "an attacker could gain access to a software agent's credentials and impersonate the agent". It calls the authentication of such agents "an open issue". The five rules answer it for one class of NPE: the AI that operates infrastructure.

## 5. The five rules

### Rule 1. Act only as the person who asked

The AI never has authority of its own. Every operation it performs is authorised for the person who asked, by the platform, at the moment it happens.

- **R1.1** Every operation the AI performs on a managed system MUST be authorised by the platform against the requesting person's own permissions, at the time of the operation.
- **R1.2** Authorisation MUST NOT depend on anything the AI produces, such as the AI stating whom it acts for, which role applies, or that an operation is safe.
- **R1.3** The AI MUST NOT hold access that outlasts the session or exceeds the requesting person's.
- **R1.4** Operations the platform does not offer the AI MUST be unavailable to it. An operation that is available but forbidden by an instruction to the AI does not meet this requirement.
- **R1.5** Endpoints that manage authentication itself, such as password resets, user and group management, session and token administration, SHOULD NOT be offered to the AI at all.
- **R1.6** A session with no requesting person present, such as a scheduled investigation, MUST be limited to reads, apart from pre-approved automations (R4.7).

**Rationale.** A model that decides its own authority can be talked out of its limits. Tying every operation to the requesting person, checked by code, means a steered AI can at worst do what that person could do through reads, and nothing more until a person approves a change.

**Verification.**

1. As a person without access to a given system, ask the AI to read it. The platform, not the model, refuses.
2. Ask the AI to call an operation the platform does not offer, such as a user-management endpoint. It does not exist for the AI.
3. Revoke the requesting person's access in the middle of a session. The AI's next operation on that system is refused.

### Rule 2. Never give the AI a credential

The AI uses systems without holding the keys. Credentials stay with the platform, which attaches them to each operation outside the model.

- **R2.1** Credentials for managed systems MUST NOT be anywhere the AI can read: its context, tool results, environment variables, files, memory, or logs it can query.
- **R2.2** The platform MUST attach credentials to operations itself, after authorising each operation (R1.1).
- **R2.3** Where a change needs a secret when it runs, the platform MUST release the secret only after the change is approved, MUST deliver it only to the component that executes that change, and MUST remove it from any output the AI can read.
- **R2.4** Any identity the AI's session presents to the platform MUST be bound to that session and its requesting person, MUST NOT be usable outside it, and MUST expire or be revoked when the session ends.
- **R2.5** Credentials the AI's own runtime needs, such as a key for the model provider, SHOULD NOT be readable by the AI's tools or code.

**Rationale.** A credential in the AI's reach turns one injected instruction into a working attack, and it can leave through logs, scratch files, generated code and the context sent to the model provider. Scoping a credential reduces the damage; keeping it out of reach removes the path. NIST SP 800-207 section 3 gives the policy administrator, not the subject, the job of generating "any session-specific authentication and authentication token or credential". This rule goes one step further for the AI: the credential is used on its behalf and never handed to it.

**Verification.**

1. Ask the AI to print its environment, list its files and search its context for anything that looks like a key or token. None is found.
2. Ask the AI to retrieve a stored secret. It can learn that the secret exists, by name, but never its value.
3. Approve a change that needs a secret, then read every output the AI can see from that run. The value does not appear.
4. Take the identity from one AI session and present it in another, or after the session ends. It is refused.

### Rule 3. Give the AI no network of its own

The AI reaches your systems only through the platform, and has no other way in or out. This is the rule that stops a steered AI from sending what it has read to an attacker.

- **R3.1** The AI MUST reach managed systems only through the platform. It MUST NOT have network access to the organisation's private networks.
- **R3.2** Code the AI runs, other than as an approved change, MUST run where it has no network access except the platform's own mediated interface.
- **R3.3** Every means of external communication available to the AI MUST be removed, restricted to destinations that people have allow-listed, or treated as a change under Rule 4. An allow-list that people define is a pre-approved class in the sense of R4.8.
- **R3.4** Where the AI needs public information, such as vendor documentation or security advisories, retrieval MUST NOT be able to carry information from managed systems out. Acceptable designs include a separate session with no access to managed systems, retrieval limited to sources that people have allow-listed, or a person approving each request.
- **R3.5** The platform MUST disclose which model provider receives what the AI reads, and SHOULD let the organisation choose that provider.

**Rationale.** Simon Willison's "lethal trifecta" (2025) is access to private data, exposure to untrusted content, and the ability to communicate externally; an agent with all three can be tricked into sending private data to an attacker. An AI operations system cannot give up the first two, so it must give up the third. A request to an arbitrary URL is enough to leak data, because the URL itself can carry it. Being inside the network grants the AI nothing either: NIST SP 800-207 tenet 2 holds that "Network location alone does not imply trust", and this rule removes the network path altogether.

**Verification.**

1. Ask the AI to run code that requests `https://example.com`. The request fails.
2. Ask the AI to read something from a managed system and then fetch a URL that contains it. The fetch is impossible, or it becomes a proposal for a person to approve.
3. Ask the AI to connect to a private address in your network directly, bypassing the platform. It cannot.
4. Ask which model provider receives what the AI reads. The answer is specific and documented.

### Rule 4. Let only a person approve a change, exactly as it will run

The AI may propose any change. Only a person can approve one, and what runs is exactly what that person saw.

- **R4.1** Every change the AI initiates MUST be a proposal, and MUST NOT run until a person approves it, unless it is a pre-approved automation (R4.7) or falls in a pre-approved class (R4.8).
- **R4.2** Approval MUST bind to the exact content of the proposal, for example by a cryptographic hash, and the platform MUST verify at execution that what runs is what was approved. An approval of a description, with details filled in afterwards, does not meet this requirement.
- **R4.3** The approval interface itself MUST refuse approvals from anyone but a person: the AI, its sub-agents, unattended jobs and machine identities are refused, whatever they claim.
- **R4.4** The person approving MUST be shown the exact content that will run, not only a summary written by the AI.
- **R4.5** The platform MUST record the channel and the assurance of every approval. Where an approval can be given through software that also runs the AI, such as an approval prompt in an MCP client, the platform SHOULD offer a second factor bound to the approving person, and MUST record whether it was used.
- **R4.6** Whether an operation is a read or a change MUST be decided by the platform from the operation's effect (section 3), never by the AI.
- **R4.7** A person MAY approve a pre-approved automation. The AI MUST NOT create, modify, widen or retarget one without a new approval.
- **R4.8** An organisation MAY let a class of change run without approval of each instance, only if people define the class in advance, the platform decides membership from the proposal's content, and nothing the AI produces can influence that decision.
- **R4.9** Writes to anything the AI will later read as instructions, such as its memory, prompts, skills or tool descriptions, SHOULD be treated as changes.

**Rationale.** NIST SP 800-207 tenet 6 requires that authorisation be "dynamic and strictly enforced before access is allowed". For a change made by an AI, the approval is that enforcement, and it only holds if it binds to what will actually run. Agent frameworks that let an AI earn autonomy through a good record, such as the maturity levels of the CSA Agentic Trust Framework, are a sound model for many agents. For changes to production infrastructure this specification takes the stricter position: a good record earns the AI nothing, because one injected log line can steer a model with a perfect history. What may run unattended is decided by people in advance (R4.7, R4.8), never by the AI.

**Verification.**

1. Ask the AI to make a change. It produces a proposal, and nothing changes until a person approves it.
2. Call the approval interface with the AI's own identity, an unattended job's identity and a machine token. All are refused.
3. Alter a proposal after it has been approved. Execution refuses to run it.
4. Look at what the approver sees. It is the exact content, not a paraphrase.
5. Ask the AI to edit or retarget an existing schedule. The result is a new proposal.

### Rule 5. Record every decision where the AI cannot change it

What the AI asked, who decided and what ran is kept where the AI cannot touch it.

- **R5.1** For every change, the platform MUST record the requesting person, the proposal content or a hash that identifies it, the evidence the proposal cites, who approved or rejected it, when, over which channel, with which assurance (R4.5), what ran, and the result.
- **R5.2** For reads, the platform SHOULD record what the AI read, from which system, and on whose behalf.
- **R5.3** The AI MUST NOT be able to modify or delete any record.
- **R5.4** Records SHOULD be exportable to the organisation's own logging or SIEM platform.
- **R5.5** The platform MUST NOT describe its records as immutable or tamper-proof unless a mechanism makes them so, and MUST document how long it keeps them.

**Rationale.** When something goes wrong, the questions are what the AI was told, what it did, and who let it. NIST SP 800-207 tenet 7 calls for the enterprise to collect "as much information as possible about the current state of assets, network infrastructure and communications"; a decision record is that information for the AI. Section 3 of NIST SP 800-207 gives the same duty to its policy engine, which "makes and logs the decision (as approved, or denied)".

**Verification.**

1. Approve a change, then ask for its record. Every field in R5.1 is present.
2. Ask the AI to delete or edit that record. It cannot.
3. Ask how the records are protected and how long they are kept. The answer names the mechanism and the retention period, and an export to your own platform exists.

## 6. Out of scope

- The safety and quality of the model itself: hallucination rates, alignment, content filtering, training data and model supply chain.
- Securing AI applications generally, such as chatbots and retrieval systems, beyond the AI operations system defined in section 1.
- The platform vendor's own access to its customers' environments. It matters, and [CHECKLIST.md](CHECKLIST.md) asks about it, but it is vendor due diligence, not a property of the AI.
- Certification. This specification defines requirements and self-assessment ([CONFORMANCE.md](CONFORMANCE.md)); it does not certify anyone.

## 7. References

- NIST. *Zero Trust Architecture*. Special Publication 800-207, August 2020. <https://doi.org/10.6028/NIST.SP.800-207>. Quoted from section 2.1 (tenets), section 3 (logical components) and section 5.7 (non-person entities).
- OWASP GenAI Security Project. *OWASP Top 10 for LLM Applications 2026*: LLM01:2026 Prompt Injection and LLM03:2026 Excessive Agency. <https://github.com/GenAI-Security-Project/GenAI-LLM-Top10/tree/main/2026/final>
- Meta. *Agents Rule of Two: A Practical Approach to AI Agent Security*. 31 October 2025. <https://ai.meta.com/blog/practical-ai-agent-security/>
- Simon Willison. *The lethal trifecta for AI agents: private data, untrusted content, and external communication*. 16 June 2025. <https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/>
- Cloud Security Alliance. *The Agentic Trust Framework: Zero Trust Governance for AI Agents*. 2 February 2026. <https://cloudsecurityalliance.org/blog/2026/02/02/the-agentic-trust-framework-zero-trust-governance-for-ai-agents>
- S. Bradner. *Key words for use in RFCs to Indicate Requirement Levels*. RFC 2119, March 1997. B. Leiba. *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*. RFC 8174, May 2017.
