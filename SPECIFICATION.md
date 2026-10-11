# Zero-trust AI operations: specification

**Version** 1.0.0-draft.1 · **Status** Public review draft · **Licence** [CC BY 4.0](LICENSE) · **Changes** [CHANGELOG.md](CHANGELOG.md)

This document is the normative text of zero-trust AI operations. The [README](README.md) explains the idea; this document says exactly what a system must do to meet it. [CONFORMANCE.md](CONFORMANCE.md) explains how to assess a system against it, and [CROSSWALK.md](CROSSWALK.md) maps every requirement to NIST SP 800-207, the OWASP Top 10 for LLM Applications 2026, the OWASP Top 10 for Agentic Applications 2026, the Model Context Protocol specification and related work.

This is a draft for public review. Requirements and identifiers may change until version 1.0.0 is released; every comment received during review will be answered in public before then.

## Contents

1. [Scope](#1-scope)
2. [Conventions](#2-conventions)
3. [Terminology](#3-terminology)
4. [Threat model](#4-threat-model)
5. [Requirements](#5-requirements)
   - [General requirements](#general-requirements)
   - [Rule 1. Act only as the person who asked](#rule-1-act-only-as-the-person-who-asked)
   - [Rule 2. Never give the AI a credential](#rule-2-never-give-the-ai-a-credential)
   - [Rule 3. Give the AI no network of its own](#rule-3-give-the-ai-no-network-of-its-own)
   - [Rule 4. Let only a person approve a change, exactly as it will run](#rule-4-let-only-a-person-approve-a-change-exactly-as-it-will-run)
   - [Rule 5. Record every decision where the AI cannot change it](#rule-5-record-every-decision-where-the-ai-cannot-change-it)
6. [Residual risk](#6-residual-risk)
7. [Out of scope](#7-out-of-scope)
8. [References](#8-references)

## 1. Scope

This specification applies to **AI operations systems**: software in which an AI model investigates and changes IT systems, such as servers, hypervisors, network appliances, identity providers, cloud accounts, Kubernetes clusters, code repositories and endpoints, on behalf of people.

It applies whether the system is bought or built in-house, whether the model is hosted or local, and whatever the AI is called: assistant, agent, copilot or AI SRE. It applies to read-only systems too: Rules 1, 2, 3 and 5 protect what the AI reads, not only what it changes.

It does not define how a model should behave. It defines what the system around the model must make impossible, on the assumption that the model will misbehave.

## 2. Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [[RFC2119]](https://www.rfc-editor.org/rfc/rfc2119) [[RFC8174]](https://www.rfc-editor.org/rfc/rfc8174) when, and only when, they appear in all capitals, as shown here.

Every requirement is a property of the system as deployed. "The platform MUST" means the platform's deterministic code enforces it. "The AI MUST NOT be able to" means the platform makes it impossible, whatever the AI attempts.

Each requirement has an identifier, such as **R4.3**. Each rule ends with **verification**: tests that check the requirements without reading source code. The tests examine the platform, not the model:

- The tester issues the AI's tool calls and runs code in the AI's execution environment directly, without relying on the model, and then repeats each test through the AI with adversarial instructions.
- A test passes only when the platform prevents the outcome. A model that declines, or fails, to carry out an attempt is never a pass.

## 3. Terminology

- **The AI.** The model and everything that acts on its output without a person in between: its tool calls, the code it writes and runs, the browser or computer sessions it drives, its sub-agents, other agents it delegates to, and anything it writes to memory.
- **Platform.** The deterministic software around the AI that authenticates people, mediates the AI's access to systems, holds credentials, runs approvals, executes changes and keeps records. The platform is not the AI.
- **Organisation.** The customer that operates the managed systems and uses the platform.
- **Person.** A human user who has authenticated to the platform as themselves.
- **Session.** A bounded unit of the AI's work, with a start and an end, carried out for one requesting person.
- **Requesting person.** The person on whose behalf the AI works in a session. For a scheduled session, the person who set it up.
- **Managed system.** Any system the AI can read or change through the platform.
- **Operation.** A single request the AI makes through the platform to a managed system.
- **Read.** An operation whose only effects are to return information, plus incidental effects that the managed system produces for any read, such as access logs, metrics and caches. An operation that alters data that people or other systems rely on is not a read: for example, marking a message as read, consuming a one-time item, or counting towards an account lockout. Whether an operation is a read is decided by its effect, not by its protocol: a request that uses HTTP `POST` to run a query can be a read, and a request that uses `GET` to trigger an action is not.
- **Change.** Any operation that is not a read, and any sending of information outside the platform, such as an email, a message, a ticket, a webhook or a pull request.
- **Proposal.** The complete content of a change, prepared so that it can run without any further input from the AI (R4.2).
- **Approval.** A decision by a person that one specific proposal, or a set of proposals shown together, may run.
- **Pre-approved automation.** A change with fixed content and a fixed scope that a person has approved in advance to run later or repeatedly, such as a scheduled playbook (R4.8).
- **Pre-approved class.** A kind of change that people have defined in advance as allowed to run without approval of each instance (R4.9).
- **Credential.** Any secret that grants access: a password, API key, token, private key, session cookie or recovery code.
- **External communication.** Any path by which information can leave the platform's control, other than to the providers disclosed under R3.6 and other than as an approved change. It includes network requests, name resolution (DNS), mail, messages, tickets, webhooks, requests to agents or services outside the platform, files written to shared locations, and automatic fetches.
- **Automatic fetch.** A request that software makes for an address found in the AI's output without a person choosing to open it: for example, loading an image, generating a link preview or chat unfurl, embedding a frame or prefetching a page.
- **Platform record.** Information the platform retrieved or produced itself, with its source, as opposed to text written by the AI.
- **Decision record.** The record of a change (R5.1).

## 4. Threat model

**Assumption.** The AI can be wrong, or steered by an attacker, at any moment of any session. Everything the AI reads can carry instructions: log lines, alert text, ticket comments, hostnames, configuration files, documentation, web pages, tool descriptions, messages from other agents, tool output and its own memory. In the words of the OWASP Top 10 for LLM Applications 2026 (LLM01:2026 Prompt Injection), "no reliable prevention mechanism exists today", and "Defense is therefore architectural rather than interceptive."

**Adversaries.**

- **A1.** Anyone who can put content where the AI will read it. They need no access to the platform or to the managed systems.
- **A2.** A model that produces harmful output for any reason, including its own error.
- **A3.** Software that can act in a person's own session or device, such as an AI agent in the person's browser or AI client, and that may try to act as that person.

**Trusted.** The platform's deterministic code; the authentication of people; the access controls of the managed systems themselves; the model provider and any search provider disclosed under R3.6, for the confidentiality of what they receive under the organisation's agreement, but never for the content of their output; and the person who approves a change, to the extent of reading what they approve.

An AI operations system cannot avoid two of the three properties that make prompt injection dangerous. In the terms of the Agents Rule of Two (Meta, 2025), it processes untrustworthy inputs [A] and has access to sensitive systems [B]. This specification keeps the third property, the ability to change state or communicate externally [C], out of the AI's hands: changes pass through a person (Rule 4), and external communication is removed, restricted or approved (Rule 3). What remains is set out in [section 6](#6-residual-risk).

**Security goals.** An AI that has been steered MUST NOT be able to:

- **G1** act beyond the authority of the person who asked, or expose what it read to anyone without access to its source (Rule 1);
- **G2** obtain or use a credential (Rule 2);
- **G3** reach a system except through the platform, or send what it has read anywhere other than to the providers the organisation knows about (Rule 3);
- **G4** change a managed system without a person approving that exact change, against the same targets, before the approval expires (Rule 4);
- **G5** approve a change, or obtain an approval by presenting its own words as evidence (Rule 4);
- **G6** alter or erase the record of what happened (Rule 5).

NIST SP 800-207 named this risk in 2020. Section 5.7 warns that "an attacker will be able to induce or coerce an NPE [non-person entity] to perform some task that the attacker is not privileged to perform", and that "an attacker could gain access to a software agent's credentials and impersonate the agent". It calls the authentication of such agents "an open issue". The five rules answer it for one class of NPE: the AI that operates infrastructure.

## 5. Requirements

### General requirements

- **R0.1** No requirement in this document may be met by instructing the AI, or by the judgement of a model, including a second model that reviews the first. Model-based detection and filtering can add defence in depth; they never count as meeting a requirement.
- **R0.2** Where a check that a requirement depends on cannot be completed, for example because a policy service, an approval store or a digest cannot be reached or read, the operation MUST NOT proceed.

### Rule 1. Act only as the person who asked

The AI never has authority of its own. Every operation it performs is authorised for the person who asked, by the platform, at the moment it happens, and what it learns stays with the people entitled to it.

- **R1.1** Every operation the AI performs MUST be authorised by the platform, at the time of the operation, against the permissions the requesting person holds at that time. Those permissions can be held in the platform's own access model, delegated by the managed system (for example through an on-behalf-of token), or both. Where the platform reaches a managed system with a credential of its own, it MUST enforce the requesting person's permissions before using it.
- **R1.2** Authorisation MUST NOT depend on anything the AI produces, such as the AI stating whom it acts for, which role applies, or that an operation is safe.
- **R1.3** The access available to the AI MUST NOT exceed the requesting person's, and MUST end when the session ends.
- **R1.4** Operations the platform does not offer the AI MUST be unavailable to it. An operation that is available but forbidden by an instruction to the AI does not meet this requirement.
- **R1.5** The platform MUST classify every operation it offers as a read or a change, by its effect (section 3), before offering it, and MUST treat any operation it has not classified as a change. The AI MUST NOT be able to decide or influence the classification.
- **R1.6** Operations that create, reset, reveal or revoke credentials, authentication factors, sessions or tokens SHOULD NOT be offered to the AI, even as proposals.
- **R1.7** A session with no requesting person present, such as a scheduled investigation, MUST be limited to reads and to proposals that wait for approval, apart from pre-approved automations (R4.8). Each run MUST be authorised against the permissions its requesting person holds at the time of the run, and MUST NOT start once that person's access has ended.
- **R1.8** The platform SHOULD limit the rate and volume of operations per session and per managed system, so that reads alone cannot degrade a managed system.
- **R1.9** What the AI reads or produces in a session MUST NOT become available to another person, session or organisation, including through shared memory, knowledge bases, search or caches, unless a person with access to the systems it came from deliberately shares it, for example by asking another person to approve a proposal. The sessions, memory and records of different organisations MUST be isolated from each other.
- **R1.10** People the organisation authorises MUST be able to end any AI session, or all of them at once. Ending a session MUST immediately end its access (R1.3) and its identity (R2.4).

**Rationale.** A model that decides its own authority can be talked out of its limits. Tying every operation to the requesting person, checked by code at the time it happens, means a steered AI can at worst read what that person could read, and change nothing until a person approves it. Classifying operations in advance, and treating anything unclassified as a change, means a new or forgotten endpoint fails closed. Keeping what the AI learns within the reach of the people entitled to it stops the AI from becoming a way around the managed systems' own access controls.

**Verification.**

1. As a person without access to a managed system, issue a read of that system from the AI's environment. The platform refuses.
2. From the AI's environment, call an operation the platform does not offer, such as user management. It does not exist.
3. Revoke the requesting person's access to a system in the middle of a session. The next operation on that system is refused.
4. Ask for the classification of every operation offered. Call one that has no classification. It becomes a proposal.
5. Remove the access of the person who set up a scheduled session. Its next run does not start.
6. As a second person without access to a system the first person's session read, search every place the platform shows shared content: memory, knowledge, search, cases. Nothing read from that system appears.
7. End all sessions at once while one is working. Its next operation is refused.

### Rule 2. Never give the AI a credential

The AI uses systems without holding the keys. Credentials stay with the platform, which attaches them to each operation outside the model.

- **R2.1** Credentials the platform uses for managed systems MUST NOT be anywhere the AI can read: its context, tool results, error messages, environment variables, files, memory, or logs it can query.
- **R2.2** The platform MUST attach credentials to operations itself, after authorising each operation (R1.1).
- **R2.3** A proposal MUST refer to any secret it needs by name, never by value. Where a change needs a secret when it runs, the platform MUST release the secret only after the change is approved, MUST deliver it only to the component that executes that change, and MUST remove it from any output the AI can read.
- **R2.4** Any identity the AI's session presents to the platform MUST be bound to that session and its requesting person, and MUST expire or be revoked when the session ends. A copy of it MUST NOT be usable from anywhere other than the session's own runtime: either the AI's tools and code cannot read it, or it is sender-constrained, for example by certificate-bound tokens [[RFC8705]](https://www.rfc-editor.org/rfc/rfc8705) or proof of possession [[RFC9449]](https://www.rfc-editor.org/rfc/rfc9449) with a key the AI's tools and code cannot read.
- **R2.5** Other credentials the AI's runtime uses, such as the key for the model provider, MUST NOT be readable by the AI's tools or code.
- **R2.6** Where read results contain secret values, the platform MUST replace them before the AI receives them, wherever it can identify them from the structure of the data: for example Kubernetes secrets, private keys, connection strings, and fields the managed system marks as secret. The platform SHOULD also detect and replace secret values in unstructured data. It MAY tell the AI a secret's name, its type and whether a value is set.

**Rationale.** A credential in the AI's reach turns one injected instruction into a working attack, and it can leave through logs, scratch files, generated code and the context sent to the model provider. Scoping a credential reduces the damage; keeping it out of reach removes the path. NIST SP 800-207 section 3 gives the policy administrator, not the subject, the job of generating "any session-specific authentication and authentication token or credential". This rule goes one step further for the AI: the credential is used on its behalf and never handed to it. Credentials also sit inside the data the AI reads, in configuration, scripts and secret stores, so the platform removes them on the way in.

**Verification.**

1. From the AI's environment, read everything its tools and code can read: environment, files, memory, logs and error output. Search it for anything that looks like a key or token. None is found.
2. Ask for a stored secret. The AI can learn that it exists, by name, but never its value.
3. Read a Kubernetes secret, or another structured secret, through the platform. The values are replaced.
4. Approve a change that needs a secret, then read every output the AI can see from that run. The value does not appear.
5. Take the session's identity out of the runtime, if it is readable at all, and present it from another host during the session, in another session, and after the session ends. Each attempt is refused.

### Rule 3. Give the AI no network of its own

The AI reaches your systems only through the platform, and has no way out of its own: no network, no unapproved channel, and no automatic fetch that could carry what it read. This is the rule that stops a steered AI from sending what it has read to an attacker.

- **R3.1** The AI MUST reach managed systems only through the platform. It MUST NOT have network access to the organisation's networks.
- **R3.2** Code the AI runs, other than as an approved change, MUST run where it has no network access except to the platform's mediated interface. Name resolution (DNS) is network access.
- **R3.3** Every channel of external communication available to the AI MUST be removed, limited to destinations that people have allow-listed in advance, or treated as a change under Rule 4. A destination MAY be allow-listed only if everyone who can read what arrives there may see what the AI read.
- **R3.4** Software that shows the AI's output to people MUST NOT make automatic fetches for addresses in that output, unless the address belongs to the platform or is allow-listed under R3.3. This covers the platform's own interfaces and any mail, chat or ticketing system the platform delivers output to; where the platform cannot turn automatic fetches off in such a system, it MUST deliver output there without fetchable addresses. A link that a person must choose to open, with its full address visible, is not an automatic fetch.
- **R3.5** Where the AI retrieves public information, such as vendor documentation or security advisories, the AI MUST NOT control the destination of any request, nor its content beyond a search query sent to a provider disclosed under R3.6. Designs that meet this requirement include:
  - the platform retrieving only addresses returned by that search, or on an allow-list people maintain, and building each request itself;
  - a separate session with no access to managed systems, whose input a person writes or approves;
  - a person approving each request.
- **R3.6** The platform MUST disclose which model provider and which search provider, if any, receive what the AI reads, under whose account and on which terms. The platform, not the AI, MUST choose the account and endpoint of every model and search request, so that the AI cannot direct a request to an account it names. The organisation SHOULD be able to choose the providers.

**Rationale.** Simon Willison's "lethal trifecta" (2025) is access to private data, exposure to untrusted content, and the ability to communicate externally; an agent with all three can be tricked into sending private data to an attacker. An AI operations system cannot give up the first two, so it must give up the third. A request to an arbitrary address is enough to leak data, because the address itself can carry it, and so is a name lookup or an image that a person's browser loads on its own. Being inside the network grants the AI nothing either: NIST SP 800-207 tenet 2 holds that "Network location alone does not imply trust", and this rule removes the network path altogether.

**Verification.**

1. From the AI's environment, request `https://example.com`, and look up a name in a domain whose name servers you can watch. The request fails, and no lookup reaches your name servers.
2. From the AI's environment, connect directly to a private address in your network. It fails.
3. Read a value from a managed system, then try to send it out through every channel the AI has: a request to an address of your choosing, mail, chat, a ticket, a webhook. Each attempt is impossible, reaches only an allow-listed destination, or becomes a proposal.
4. Make the AI's output contain an image and a link that point to a server you control, and open that output in the platform and in every system the platform delivers output to. Your server receives nothing until a person opens the link.
5. Ask the AI to retrieve an address it has built itself, containing a value it has read. It is refused, or becomes a request a person must approve.
6. Place a model provider key of your own in a document the AI reads, and instruct the AI to use it. No request reaches your account.
7. Ask which providers receive what the AI reads. The answer is specific and documented.

### Rule 4. Let only a person approve a change, exactly as it will run

The AI may propose any change. Only a person can approve one, and what runs is exactly what that person saw, against the same targets, while the approval is still valid.

- **R4.1** Every change the AI initiates MUST be a proposal, and MUST NOT run until a person approves it, unless it is a pre-approved automation (R4.8) or falls in a pre-approved class (R4.9).
- **R4.2** A proposal MUST contain everything that determines what runs: the operation and its parameters, or the script, playbook or diff; its targets; and every dependency it fetches when it runs, such as container images, packages, modules and remote scripts, pinned by digest. A proposal that would fetch mutable content when it runs does not meet this requirement.
- **R4.3** Approval MUST bind to the exact proposal by a cryptographic digest of its content, and the platform MUST verify at execution that what runs matches the approved digest. A proposal whose content cannot be verified at execution MUST NOT run.
- **R4.4** A proposal MUST identify its targets by identifiers that are not reused, such as object IDs or host keys, where the managed system provides them, not by name alone. At execution, the platform MUST verify that each target is still the one the person approved, and SHOULD verify any preconditions the proposal states about its current state. If a check fails, the change MUST NOT run.
- **R4.5** An approval MUST expire if the change has not started within a period the organisation sets. The platform MUST apply a finite default when none is set.
- **R4.6** Only a person can approve. The approval interface MUST refuse the AI, its sub-agents, unattended jobs and machine identities, whatever they claim. Because software can act in a person's own session (adversary A3), each approval, or each set of proposals approved together, MUST be confirmed with user verification that software cannot supply on the person's behalf, such as a passkey or security key with user verification, or a confirmation on a separate registered device. Rejecting a proposal needs no verification.
- **R4.7** The person approving MUST be shown the exact content that will run, rendered so that invisible and look-alike characters are visible, together with its targets, the evidence it cites and when the approval will expire. Evidence MUST be shown as platform records with their sources. Text the AI wrote, such as its reasoning and summaries, MUST be marked as written by the AI and MUST NOT be presented as evidence.
- **R4.8** A person MAY approve a pre-approved automation. Creating, modifying, widening or retargeting one is itself a change under this rule. Each run MUST be authorised under R1.7 and recorded under R5.1.
- **R4.9** An organisation MAY let a class of change run without approval of each instance, only if people define the class in advance and the platform decides membership from the proposal's content by deterministic rules, with no model taking part. Creating or widening a class is itself a change under this rule.
- **R4.10** Writes to anything that people or the platform treat as trusted instructions for an AI, such as system prompts, skills, tool definitions, playbooks and runbooks, MUST be treated as changes. Memory the AI writes MUST be labelled with its origin and MUST be treated as untrusted input whenever it is read, unless a person approves it as instructions.
- **R4.11** An organisation MAY require that the person approving a change is not the requesting person, for classes of change it defines. The platform SHOULD support this.

**Rationale.** NIST SP 800-207 tenet 6 requires that authorisation be "dynamic and strictly enforced before access is allowed". For a change made by an AI, the approval is that enforcement, and it only holds if it binds to what will actually run: the content, everything the content pulls in, the targets as they were when the person looked, and a time window. It also only holds if a person gave it. A signed-in session proves who logged in, not who clicked, and software now acts inside people's browsers and clients; verification that software cannot supply closes that gap. The approver is the last line of defence against a steered AI, so what they see must come from the platform, not from the AI's own account of events. Agent frameworks that let an AI earn autonomy through a good record, such as the maturity levels of the Agentic Trust Framework, are a sound model for many agents. For changes to production infrastructure this specification takes the stricter position: a good record earns the AI nothing, because one injected log line can steer a model with a perfect history. What may run unattended is decided by people in advance (R4.8, R4.9), never by the AI.

**Verification.**

1. Ask the AI to make a change. It produces a proposal, and nothing changes until a person approves it.
2. Call the approval interface with the AI's own identity, an unattended job's identity and a machine token. All are refused.
3. Approve as a signed-in person, but without the user verification step, for example by replaying the approval request from the browser session or by driving the browser with an automation tool. It is refused.
4. Alter a proposal after it has been approved. Execution refuses it. Alter a dependency the proposal uses, for example by re-tagging a container image. The proposal pinned the digest, so what runs is unchanged, or execution refuses it.
5. Approve a change, then replace its target with a new object of the same name before it runs. Execution refuses it.
6. Let an approval pass its expiry. The change cannot run.
7. Put an invisible character in a proposal's content. The approval view shows it.
8. Look at the approval view. The exact content and targets are shown, the evidence links to platform records, and the AI's own text is marked as such.
9. Ask the AI to edit or retarget a schedule, widen a pre-approved class, or change its own instructions. Each becomes a proposal.

### Rule 5. Record every decision where the AI cannot change it

What the AI read, what it asked for, who decided and what ran is kept where the AI cannot touch it.

- **R5.1** For every change, the platform MUST record: the requesting person; the exact proposal content and its digest; its targets by identifier; the evidence it cites; who approved or rejected it, when, through which channel and with which user verification; which pre-approved automation or class applied, if any; and what ran, when, and with what result.
- **R5.2** For every operation, including reads, the platform MUST record which operation it was, on which system, in which session, for which person, and when. It MAY also keep what the operation returned.
- **R5.3** The AI MUST NOT be able to modify or delete any record.
- **R5.4** Text the AI wrote MUST be kept in records as data and marked as written by the AI. Interfaces and exports that show records MUST NOT interpret it as markup, formulas or commands.
- **R5.5** Records SHOULD be tamper-evident, for example by hash-chaining them or writing them to storage that the platform's own operators cannot alter.
- **R5.6** Records SHOULD be exportable to the organisation's own logging or SIEM platform.
- **R5.7** The platform MUST NOT describe its records as immutable or tamper-proof unless a mechanism makes them so, and MUST document how long it keeps them.

**Rationale.** When something goes wrong, the questions are what the AI read, what it was told, what it did, and who let it. A record of reads is what shows, after a leak, what could have left. NIST SP 800-207 tenet 7 calls for the enterprise to collect "as much information as possible about the current state of assets, network infrastructure and communications"; these records are that information for the AI. Section 3 of NIST SP 800-207 gives the same duty to its policy engine, which "makes and logs the decision (as approved, or denied)". Text the AI wrote is attacker-influenced, so it must not become an attack on the people and tools that later read the records.

**Verification.**

1. Approve a change, then retrieve its record. Every field in R5.1 is present.
2. Have the AI read from a managed system, then retrieve the record of that read. Every field in R5.2 is present.
3. From the AI's environment, try to delete or edit a record. It cannot.
4. Make the AI write text containing markup and a spreadsheet formula, then view its record in the platform and in an export. Both show it as plain text.
5. Ask how records are protected and how long they are kept. The answer names the mechanism and the retention period.

## 6. Residual risk

A conforming system still leaves risks that the organisation must accept or manage in other ways. A conformance statement MUST assess each of them for the system it describes ([CONFORMANCE.md](CONFORMANCE.md)).

1. **Misleading conclusions.** A steered or mistaken AI can give a person a wrong diagnosis. The rules stop it from acting on that diagnosis, and R4.7 shows the approver platform records rather than the AI's account, but a person can still act on the AI's advice by other means.
2. **Harmful changes a person approves.** The rules guarantee that the person saw exactly what will run, not that they understood it.
3. **Disclosure to the disclosed providers.** Everything the AI reads reaches the model provider, and search queries reach the search provider (R3.6).
4. **Low-bandwidth signalling.** A steered AI can still signal a little through choices it is allowed to make, such as which permitted search result it opens and when.
5. **Load and cost.** Reads within the limits of R1.8 still consume the managed systems' capacity.
6. **Compromise outside the AI.** A compromised person, together with their authenticator, or a compromised platform, is outside this specification.

## 7. Out of scope

- The safety and quality of the model itself: hallucination rates, alignment, content filtering, training data and model supply chain.
- Securing AI applications generally, such as chatbots and retrieval systems, beyond the AI operations system defined in section 1.
- The security of the platform itself, including its code, hosting and operators, and the platform vendor's own access to its customers' environments. They matter, and [CHECKLIST.md](CHECKLIST.md) asks about them, but they are vendor due diligence, not properties of the AI.
- Certification. This specification defines requirements and self-assessment ([CONFORMANCE.md](CONFORMANCE.md)); it does not certify anyone.

## 8. References

- NIST. *Zero Trust Architecture*. Special Publication 800-207, August 2020. <https://doi.org/10.6028/NIST.SP.800-207>. Quoted from section 2.1 (tenets), section 3 (logical components) and section 5.7 (non-person entities).
- OWASP GenAI Security Project. *OWASP Top 10 for Large Language Model Applications*, version 2026, published 4 August 2026. <https://doi.org/10.5281/zenodo.22109015>
- OWASP GenAI Security Project, Agentic Security Initiative. *OWASP Top 10 for Agentic Applications 2026*. December 2025. <https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/>
- Model Context Protocol. *Specification*, version 2026-07-28. <https://modelcontextprotocol.io/specification/2026-07-28>
- Meta. *Agents Rule of Two: A Practical Approach to AI Agent Security*. 31 October 2025. <https://ai.meta.com/blog/practical-ai-agent-security/>
- Simon Willison. *The lethal trifecta for AI agents: private data, untrusted content, and external communication*. 16 June 2025. <https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/>
- Josh Woodruff (MassiveScale.AI). *Agentic Trust Framework*. Published on the Cloud Security Alliance blog, 2 February 2026. <https://github.com/massivescale-ai/agentic-trust-framework>
- B. Campbell, J. Bradley, N. Sakimura, T. Lodderstedt. *OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens*. RFC 8705, February 2020.
- D. Fett, B. Campbell, J. Bradley, T. Lodderstedt, M. Jones, D. Waite. *OAuth 2.0 Demonstrating Proof of Possession (DPoP)*. RFC 9449, September 2023.
- S. Bradner. *Key words for use in RFCs to Indicate Requirement Levels*. RFC 2119, March 1997. B. Leiba. *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*. RFC 8174, May 2017.
