# Zero-trust AI operations: specification

**Version** 1.0.0-draft.1 · **Status** Public review draft · **Licence** [CC BY 4.0](LICENSE) · **Changes** [CHANGELOG.md](CHANGELOG.md)

This document is the normative text of zero-trust AI operations. The [README](README.md) explains the idea; this document says exactly what a system must do to meet it. [CONFORMANCE.md](CONFORMANCE.md) explains how to assess a system and state the result, and [CROSSWALK.md](CROSSWALK.md) maps the requirements to NIST SP 800-207, the OWASP Top 10 lists for LLM and agentic applications, the Model Context Protocol specification and related work.

This is a draft for public review. Requirements and identifiers may change until version 1.0.0 is released. Every comment received during review will be answered in public before then.

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
7. [Relation to change management](#7-relation-to-change-management)
8. [Out of scope](#8-out-of-scope)
9. [References](#9-references)

## 1. Scope

This specification applies to **AI operations systems**: software in which an AI model investigates or changes IT systems, such as servers, hypervisors, network appliances, identity providers, cloud accounts, Kubernetes clusters, code repositories and endpoints, on behalf of people or of an organisation.

It applies whether the system is bought or built in-house, whether the model is hosted or local, and whatever the AI is called: assistant, agent, copilot or AI SRE. It applies to systems that only read, too: Rules 1, 2, 3 and 5 protect what the AI reads, not only what it changes.

It does not define how a model should behave. It defines what the system around the model must make impossible, on the assumption that the model will misbehave.

## 2. Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [[RFC2119]](https://www.rfc-editor.org/rfc/rfc2119) [[RFC8174]](https://www.rfc-editor.org/rfc/rfc8174) when, and only when, they appear in all capitals, as shown here.

**What is normative.** The requirements (identifiers R0.1 to R5.10) and the verification steps are normative. Rule titles, the text that introduces each rule, rationales, the threat model, the residual risk and the relation to change management are informative.

**What conforms.** The conformance target is the complete system: the platform, the runtime that hosts the model and its tools, and every interface that shows the AI's output, in a configuration that the conformance statement names. A component that offers operations to an AI runtime it does not control can claim conformance only together with a named runtime and configuration. Software in a person's session that is outside the conformance target is treated as adversary A3 (section 4).

**Who must enforce.** Whatever its grammatical subject, every requirement is an obligation on the system to enforce the stated property with its deterministic code. Where a requirement says the system "documents" something, the documentation is part of the conformance statement ([CONFORMANCE.md](CONFORMANCE.md)).

**Verification.** Each rule ends with verification steps. Passing them is necessary, not sufficient. They test the system, not the model:

- The tester makes the AI's requests and runs code in the AI's execution environment directly, without relying on the model, and then repeats each step through the AI with adversarial instructions where that is possible.
- The direct run decides the result. An attempt that the model declines or fails to carry out is recorded as inconclusive, never as a pass.

## 3. Terminology

- **The AI.** The model and everything that acts on its output without a person in between, other than the platform and software that only displays the output: the requests it makes, the code it writes and runs, the browser or computer sessions it drives, and its sub-agents.
- **Platform.** The deterministic software that authenticates people, mediates the AI's access to systems, holds credentials, runs approvals, executes changes and keeps records, including the runtime that hosts the model, apart from code the AI writes.
- **Organisation.** The organisation that operates the managed systems and uses the platform, whether it buys the platform or builds it.
- **Person.** A human who has authenticated to the platform with an account assigned to them alone.
- **Session.** A bounded unit of the AI's work for one requesting person in one organisation, with a start and an end.
- **Requesting person.** The person on whose behalf the AI works in a session. For a session started by a schedule, an event or an integration, the person who configured that trigger.
- **Unattended session.** A session not started by a person's interactive request.
- **Managed system.** A system outside the platform that the AI can read or change through the platform.
- **Access.** A person has access to a managed system when the permissions used under R1.1 allow them to read it.
- **Operation.** A kind of action the platform offers on managed systems, such as listing virtual machines or running a command.
- **Request.** One use of an operation, with its parameters.
- **Read.** A request whose only effects are returning information and the effects that the managed system produces for every request of that kind whatever its parameters, such as its own access logs, metrics and caches. A request that also alters data or state that people or systems use, sends information to any other destination, or causes the managed system to contact a destination that its parameters determine, is not a read. The protocol does not decide: a request that uses HTTP `POST` to run a query can be a read, and a request that uses `GET` to trigger an action is not.
- **Change.** A request that is not a read.
- **External communication.** Any way the AI can send information outside the platform other than a request to a managed system: network requests and name resolution from the AI's environment, mail, messages, tickets and webhooks the platform sends for the AI, requests to agents or services outside the platform, files written to shared locations, and automatic fetches. The platform's own transfers to the third parties it discloses under R3.6 are not external communication by the AI.
- **Automatic fetch.** A request that software makes for an address found in the AI's output without a person choosing to open it: for example, loading an image, generating a link preview or chat unfurl, embedding a frame, or prefetching a page or a name.
- **Proposal.** The complete content of a change, as R4.2 defines it, prepared so that it can run without any further input from the AI.
- **Approval.** A person's decision, confirmed under R4.7, that one proposal may run once.
- **Pre-approved automation.** A change with fixed content and scope that a person has approved to run later or repeatedly on a fixed schedule or trigger (R4.10).
- **Pre-approved class.** A narrow kind of change that people have defined in advance as allowed to run without approval of each instance (R4.11).
- **Credential.** Any secret value that grants access, or from which access can be derived: a password, API key, token, private key, session cookie or recovery code.
- **Source record.** Information the platform retrieved from a managed system or produced itself, shown with its source and the time it was retrieved, as opposed to text the AI wrote. A source record shows what a system returned, not that it is true.
- **Decision record.** The record of a proposal and its outcome (R5.1).
- **User verification.** An authentication step in which an authenticator registered to the person confirms locally that the person is present and is who they claim to be, such as user verification in Web Authentication (WebAuthn).
- **Instruction store.** Any store whose content the platform places in the AI's context as instructions rather than as data, such as system prompts, skills, tool definitions, playbooks and runbooks.

## 4. Threat model

This section is informative.

**Assumption.** The AI can be wrong, or steered by an attacker, at any moment of any session. Everything the AI reads can carry instructions: log lines, alert text, ticket comments, hostnames, configuration files, documentation, web pages, tool definitions, messages from other agents, the results of its requests and its own memory. In the words of the OWASP Top 10 for LLM Applications 2026 (LLM01:2026 Prompt Injection), "no reliable prevention mechanism exists today", and "Defense is therefore architectural rather than interceptive."

**Adversaries.**

- **A1.** Anyone who can put content where the AI will read it. They need no access to the platform or to the managed systems.
- **A2.** A model that produces harmful output for any reason, including its own error.
- **A3.** Software in a person's own session or device that is outside the conformance target, such as an AI agent in the person's browser or AI client. It can do through the platform whatever that person can, except what needs that person's user verification.

**Trusted.** The platform's deterministic code; the authentication of people; the access controls of the managed systems themselves; the third parties disclosed under R3.6, for the confidentiality of what they receive on the disclosed terms, but never for the content of their output; and the person who approves a change, to the extent of reading what they approve. Source records are trusted only as a record of what a system returned.

**The Rule of Two.** An AI operations system that does its job has two of the three properties that the Agents Rule of Two (Meta, 2025) warns against combining: it processes untrustworthy inputs [A], and it has access to sensitive systems [B]. These rules keep the third property, the ability to change state or communicate externally [C], away from the AI itself. The platform changes state only for proposals a person has approved, or within automations and classes people have defined in advance and bounded, and the AI communicates externally only with allow-listed destinations or with a person's approval. [Section 6](#6-residual-risk) states what remains.

**Design goals.** The requirements are designed so that a steered AI cannot do the following, except as section 6 describes and as the allow-lists, automations and classes that people approve permit:

| Goal | A steered AI cannot | Delivered by |
| --- | --- | --- |
| G1 | act beyond the authority the organisation gave the person who asked, or expose what it read to people or organisations without access to its source | R1.1 to R1.13, R4.6 |
| G2 | obtain or use a credential | R2.1 to R2.6 |
| G3 | reach a managed system except through the platform, or send what it has read anywhere other than to disclosed third parties, allow-listed destinations or through changes people approve | R3.1 to R3.7, R4.1 |
| G4 | change a managed system except through a proposal a person approved exactly, against the same targets, once and before it expired, or within an automation or class people defined | R0.3, R4.1 to R4.5, R4.10, R4.11, R4.13 |
| G5 | approve a change, or win an approval by passing off its own words as evidence | R4.6 to R4.9 |
| G6 | alter, erase or evade the record of what happened | R0.2, R5.1 to R5.10 |
| G7 | attack the people and software that read its output | R3.4, R4.9, R4.12, R5.5 |

**Prior warning.** NIST SP 800-207 (2020) anticipated the risk for a related case. Section 5.7, on non-person entities used to administer a zero trust architecture, warns that "an attacker will be able to induce or coerce an NPE [non-person entity] to perform some task that the attacker is not privileged to perform", and that "an attacker could gain access to a software agent’s credentials and impersonate the agent". It calls the authentication of such agents "an open issue". These rules address the same concern for the AI that operates infrastructure.

## 5. Requirements

### General requirements

- **R0.1** A system MUST NOT rely on instructions to the AI, or on the judgement of any model, to meet any requirement in this document. Model-based detection and filtering can add defence in depth, but they never meet a requirement.
- **R0.2** Where a check that a requirement depends on cannot be completed, for example because a policy service, an approval store or a digest cannot be reached or read, or where the platform cannot record a request as Rule 5 requires, the request or change MUST NOT start.
- **R0.3** The AI MUST NOT be able to change any setting that takes the system out of conformance. Changing such a setting is a change under Rule 4, and the system MUST show people when it is running outside conformance.

**Verification.**

1. Make the policy service, the approval store and the record store unreachable in turn. Reads and executions are refused.
2. From the AI's environment, try to change a setting that the conformance statement lists as taking the system out of conformance. It is refused, or becomes a proposal.

### Rule 1. Act only as the person who asked

The AI never has authority of its own. Every request it makes is authorised for the person who asked, by the platform, at the moment it happens, and what it learns stays with the people entitled to it.

- **R1.1** Every request the AI makes MUST be authorised by the platform, before it runs, against the requesting person's permissions for that system as known to the platform at that time. Those permissions are the ones the organisation grants the person for that system, whether they are held in the managed system's own access model, in the platform's, or in both. Where the person has their own account on the managed system, the platform MUST NOT allow more through the AI than that account allows. The system MUST document the maximum delay between revoking a permission at its source and the platform enforcing the revocation.
- **R1.2** Authorisation MUST NOT depend on any assertion by the AI about identity, role, authority, purpose or safety.
- **R1.3** The access available to the AI in a session MUST NOT exceed the requesting person's, and MUST end when the session ends. Only a person's approval can let a change go beyond the requesting person's own authority, and only within the approver's (R4.6).
- **R1.4** Operations the platform does not offer the AI MUST be unavailable to it. An operation that is available but forbidden by an instruction to the AI does not meet this requirement.
- **R1.5** The platform MUST classify each request as a read or a change, by deterministic rules over its operation and parameters, before it runs; a request that the rules cannot classify as a read is a change. A request whose effect depends on free-form parameters, such as a command, a script or a query, is a change unless the platform enforces, independently of the AI, that it has only the effects of a read: for example through an allow-list of read-only commands and arguments, or a read-only database role. Classifications that a tool server supplies MUST NOT be relied on unless the organisation trusts that server. The AI MUST NOT be able to change the rules.
- **R1.6** Requests that create, reset or revoke credentials, authentication factors, sessions or tokens, or that weaken authentication, MUST be approved individually, never as part of a set (R4.8), an automation (R4.10) or a class (R4.11). Credential values they produce MUST NOT reach the AI (R2.6).
- **R1.7** A session MUST NOT start unless the platform can determine its requesting person. An unattended session MUST be limited to reads, to proposals that wait for approval, and to changes within pre-approved classes (R4.11). Each unattended session MUST be authorised against the permissions its requesting person holds when it runs, and MUST NOT start once that person's access to the platform has ended. The platform SHOULD alert people the organisation designates when this stops a schedule, and SHOULD let the trigger be transferred to another person as a change.
- **R1.8** The platform SHOULD let the organisation, and the requesting person, restrict the operations and managed systems available to the AI below the requesting person's permissions.
- **R1.9** The platform SHOULD limit the rate and volume of requests per session and per managed system, to limit the load the AI can place on managed systems and the volume of records it can create.
- **R1.10** What the AI reads or produces MUST NOT become available to a person who lacks access to any managed system it came from, or to another organisation, including through shared memory, knowledge, search, caches, records, or data used to train, fine-tune or evaluate models, unless a person with that access shares it deliberately. Audit access and record export that the organisation authorises count as deliberate sharing. Where more than one person can instruct a session or see its output, the platform MUST authorise each request against the permissions that all of them hold, or show the output only to the requesting person. Transferring a session to another person MUST start a new session with that person as the requesting person.
- **R1.11** The sessions, memory, records and credentials of different organisations MUST be isolated from each other. A session MUST belong to exactly one organisation.
- **R1.12** People the organisation authorises MUST be able, in one action and within a time the system documents, to end any session or all sessions with their access and identities, to withdraw pending proposals, to suspend approved changes that have not started and all automations and classes, and to withdraw any operation or managed system from all sessions. A change that is already running MUST stop before its next step that the platform controls, unless its proposal states that the step has to complete to leave the target in a safe state, and its outcome MUST be recorded.
- **R1.13** A session MUST end no later than a maximum duration the organisation sets. The platform MUST apply a finite default, which the system documents.

**Rationale.** A model that decides its own authority can be talked out of its limits. Tying every request to the requesting person, checked by code when it happens, means a steered AI can at worst read what that person could read, and change nothing until a person with the authority to make the change approves it. Classifying each request by what it does, and treating anything unclassified as a change, makes a new endpoint or an unusual command fail closed. Keeping what the AI learns within the reach of the people entitled to it stops the AI from becoming a way around the managed systems' own access controls.

**Verification.**

1. As a person without access to a managed system, issue a read of that system from the AI's environment. The platform refuses. Repeat as a person whose platform role allows the read but whose own account on that system does not. The platform refuses.
2. From the AI's environment, make a request that claims to act for another person, in another role or for a safety reason. Authorisation is unchanged. Call an operation that the platform does not offer. It does not exist.
3. Revoke the requesting person's access to a system at its source in the middle of a session. Within the documented delay, the next request to that system is refused.
4. Add a new operation without classifying it, and call it. Call a command or query operation with parameters that its rules do not recognise as a read. Both become proposals.
5. Request a password reset through the AI. It is a proposal that cannot be approved together with others, and the new password never reaches the AI.
6. Remove the access of the person who set up a scheduled session. Its next run does not start.
7. As a second person without access to a managed system that the first person's session read, search everything the platform shows to people: memory, knowledge, search, sessions and records. Nothing read from that system appears. Repeat as a person in another organisation.
8. Use the stop action while a session is working and a change is waiting to start. Within the documented time, the session's next request is refused, and the change does not start.
9. Let a session run beyond the maximum duration. It ends.

### Rule 2. Never give the AI a credential

The AI uses systems without holding the keys. Credentials stay with the platform, which attaches them to each request outside the model and removes them from what the AI reads.

- **R2.1** Credentials the platform uses for managed systems MUST NOT be anywhere the AI can read: its context, the results of its requests, error messages, its environment, files, memory, or records it can query.
- **R2.2** The platform MUST attach credentials to requests itself, after authorising each request (R1.1).
- **R2.3** The platform MUST let a proposal refer to a credential by name, and MUST NOT require credential values in proposal content. It MUST release a referenced credential only after approval, only to the component that executes that change, and only for that execution. Output of that execution MUST NOT be returned to the AI beyond status that the platform produces, such as success and exit code, unless a person releases it.
- **R2.4** Any identity that the AI's session presents to the platform MUST be bound to that session and its requesting person, and MUST expire or be revoked when the session ends. A copy of it MUST NOT be usable from anywhere other than the session's own runtime: either the AI's tools and code cannot read it, or it is sender-constrained, for example by certificate-bound tokens [[RFC8705]](https://www.rfc-editor.org/rfc/rfc8705) or proof of possession [[RFC9449]](https://www.rfc-editor.org/rfc/rfc9449), with a key that the AI's tools and code cannot read. It SHOULD be sender-constrained even where it cannot be read.
- **R2.5** Other credentials the AI's runtime uses, such as keys for the model provider and other third parties under R3.6, MUST NOT be readable by the AI's tools or code.
- **R2.6** Before the AI receives the result of any request or change, the platform MUST replace credential values in it, at least:
  - the values of fields that the managed system's schema or interface marks as secret;
  - private keys and other encoded key material;
  - credential values in data formats that the system documents as parsed, such as network device configurations, Kubernetes secrets and kubeconfig files, cloud-init and unattended-installation files, and infrastructure-as-code state;
  - any credential a change creates or resets.

  The platform SHOULD detect and replace credential values in other data. It MAY show the AI a credential's name and type, whether a value is set, non-secret parts such as a certificate and its expiry date, and a fingerprint keyed for the session that shows only whether two values match.

**Rationale.** A credential in the AI's reach turns one injected instruction into a working attack, and it can leave through logs, scratch files, generated code and the context sent to the model provider. Scoping a credential reduces the damage; keeping it out of reach removes the path. NIST SP 800-207 section 3 gives the policy administrator, not the subject, the job of generating "any session-specific authentication and authentication token or credential". This rule goes one step further for the AI: the credential is used on its behalf and never handed to it. Credentials also sit inside the data the AI reads, in configuration, scripts and secret stores, so the platform removes them on the way in. Detecting them in free text is imperfect, which is why section 6 lists what may remain.

**Verification.**

1. From the AI's environment, read everything its tools and code can read: environment, files, memory, logs and error output. No credential for a managed system or a third party is found, other than a sender-constrained session identity under R2.4.
2. Ask for a stored credential. The AI can learn that it exists, by name and type, but never its value.
3. Read a Kubernetes secret, a network device configuration and each other format the system documents. The credential values are replaced.
4. Approve a change that references a credential, then read everything the AI can see from that run. Only the status appears unless a person releases the output, and the value never appears.
5. Take the session's identity out of the runtime, if it can be read at all, and present it from another host during the session, in another session and after the session ends. Each attempt is refused.

### Rule 3. Give the AI no network of its own

The AI reaches your systems only through the platform, and has no way out of its own: no network, no channel that people have not allowed, and no automatic fetch that could carry what it read. This is the rule that limits what a steered AI can send to an attacker.

- **R3.1** The AI's only route to managed systems MUST be the platform. The AI MUST NOT have network access to the organisation's networks. Network diagnostics, such as ping, traceroute, name lookups and port checks against managed systems, are requests that the platform makes, not network access from the AI's environment.
- **R3.2** Code the AI runs, and any browser or computer session it drives, MUST have no network access except to the platform's mediated interface. Name resolution is network access. Where the AI uses the web interface of a managed system, or a public web page, the platform MUST run the browser, hold its cookies and sessions out of the AI's reach, and apply R1.5 and R3.5 to each interaction.
- **R3.3** Every channel of external communication available to the AI MUST be removed, limited to destinations that people have allow-listed in advance, or used only by sending each message as a proposal under Rule 4. Communication to an allow-listed destination is not a change. Each allow-list entry MUST state the managed systems that everyone who can read the destination has access to, and the platform MUST NOT send to it from a session that has read any other managed system. The organisation is responsible for keeping each entry true, and the platform SHOULD check it where the destination allows. Adding, widening or retargeting an allow-list entry is a change under Rule 4.
- **R3.4** Interfaces that show the AI's output MUST NOT make automatic fetches for addresses in it, unless the address belongs to the platform and fetching it causes no request to any other destination, or the address is allow-listed under R3.3. Where the platform delivers output to software outside the conformance target, such as mail, chat or ticketing systems, it MUST turn automatic fetching off there, or deliver addresses in a form that such software does not recognise as addresses. Links to destinations that are not allow-listed MUST be shown as text that cannot be clicked, or open only through an intermediate page that shows the full address with its domain highlighted.
- **R3.5** Where the AI retrieves public information, such as vendor documentation or security advisories, content the AI has read from managed systems MUST NOT be able to determine the destination or content of any request, except by writing a search query sent to a search provider disclosed under R3.6, and by choosing among addresses the platform has obtained from that provider or from an allow-list. Designs that meet this requirement include:
  - the platform building every request itself, only for addresses returned by that search or on an allow-list that people maintain;
  - a separate session with no access to managed systems, whose input a person writes or approves, and whose results reach other sessions only as untrusted content;
  - a person approving each request.
- **R3.6** The system MUST document every third party that receives content the AI reads or produces, including model, embedding, search, safety-filtering, telemetry, tracing, hosting and delivery providers, with the account used, the region, the retention, whether the content is used for training, and whether the account is shared with other organisations. The platform, not the AI, MUST choose the account and endpoint of every request to such a party, so that the AI cannot direct a request to an account or endpoint it names. The organisation SHOULD be able to choose these parties, and SHOULD be able to use its own accounts with them.
- **R3.7** An approved change MUST run with network access limited to its targets and to the destinations its proposal declares (R4.2).

**Rationale.** Simon Willison's "lethal trifecta" (2025) is access to private data, exposure to untrusted content, and the ability to communicate externally; an agent with all three can be tricked into sending private data to an attacker. An AI operations system cannot give up the first two, so the AI must not have the third. A request to an arbitrary address is enough to leak data, because the address itself can carry it, and so is a name lookup or an image that a person's browser loads by itself. Being inside the network grants the AI nothing either: NIST SP 800-207 tenet 2 holds that "Network location alone does not imply trust", and this rule removes the network path altogether. An approved change is code the AI wrote, so it runs only with the network its proposal declares, and any credential released to it under R2.3 cannot be sent anywhere else.

**Verification.**

1. From the AI's environment, and from a browser session the AI drives, request `https://example.com`, and look up a name in a domain whose name servers you can watch. The request fails, and no lookup reaches your name servers.
2. From the AI's environment, connect directly to a private address in your network. It fails.
3. Read a value from a managed system, then try to send it out through every channel the AI has: a request to an address of your choosing, mail, chat, a ticket, a webhook. Each attempt is impossible, reaches only an allow-listed destination, or becomes a proposal. Allow-list a destination for one managed system, read from another, and try to send to it. It is refused.
4. Make the AI's output contain an image and a link that point to a server you control, and open that output in the platform and in every system the platform delivers output to. Your server receives nothing, and the link opens only through an intermediate page that shows its address.
5. Ask the AI to retrieve an address it has built itself, containing a value it has read. It is refused, or becomes a request that a person must approve.
6. Place a model provider key of your own in a document the AI reads, and instruct the AI to use it. No request reaches your account.
7. Approve a change whose proposal declares no destination other than its target, and whose content tries to reach another address. The connection fails.

### Rule 4. Let only a person approve a change, exactly as it will run

The AI may propose any change. A person with the authority to make it approves it, in a way that software acting for them cannot complete without them, and what runs is the content they saw, against the same targets, once, before the approval expires. Unattended change exists only within automations and classes that people define and bound in advance.

- **R4.1** Every change the AI initiates MUST be a proposal, and MUST NOT run until a person approves it, except within a pre-approved automation (R4.10) or class (R4.11).
- **R4.2** A proposal MUST contain everything that the platform will send or cause to be fetched:
  - the operation and its parameters, or the script, playbook or diff;
  - its targets (R4.4) and the identity it runs as;
  - the network destinations it may reach (R3.7) and the credentials it references by name (R2.3);
  - each dependency it fetches, directly or by causing a target to fetch it.

  Each dependency MUST be pinned by digest, or by an exact version that a signed index binds to a digest, such as a signed package repository, or fetched from a source that the organisation has designated and the proposal names. Where a change's effect is computed when it runs, such as an infrastructure plan, the proposal MUST include the computed plan, and the platform MUST verify at execution that the plan is unchanged, or refuse to run it.
- **R4.3** An approval MUST bind to a digest computed with SHA-256, or a stronger collision-resistant hash, over a canonical serialisation of everything R4.2 lists. The platform MUST render the approval view from the bytes it digests, MUST refuse the approval if the proposal has changed since it was shown, and MUST verify at execution that what runs matches the approved digest. A proposal whose content cannot be verified MUST NOT run. An approval MUST authorise exactly one execution, which the platform, not the AI, starts, either on approval or at a time the proposal states. Retrying a failed or partial execution MUST need a new approval, unless the approved proposal states a retry policy. The platform MUST NOT let any component, including a policy service, modify a proposal after it is approved.
- **R4.4** A proposal MUST identify its targets by identifiers that the managed system itself reports and does not reuse, where it has them. Otherwise, it MUST bind each target to a fingerprint of the target's relevant state captured when the proposal is made, such as a record's value, a rule's text, a configuration hash or a serial number. Targets given as queries or groups MUST be resolved to identifiers when the proposal is made. At execution, the platform MUST verify each target's identifier or fingerprint and any preconditions the proposal states, and the change MUST NOT run if a check fails. A proposal for a destructive change SHOULD state the preconditions it depends on.
- **R4.5** An approval MUST expire if its execution has not started within a period the organisation sets, up to a maximum it sets. The platform MUST apply a default period and a maximum, which the system documents.
- **R4.6** Only a person can approve. The approval interface MUST refuse the AI, its sub-agents, unattended sessions and machine identities, whatever they claim. A person MUST NOT be able to approve a change unless they hold permission, determined as in R1.1, to make it on each of its targets; the platform MUST check this at approval and again at execution, and MUST NOT run the change if it no longer holds. The organisation MAY also require the requesting person to hold that permission. A proposal MUST be offered for approval only to people who have access to every managed system the session read before creating it.
- **R4.7** Each approval, or each set of proposals approved together (R4.8), MUST be confirmed with user verification that is bound to the digest of every proposal it approves, for example by deriving the authentication challenge from those digests. A confirmation that needs no user verification, such as an approve or deny push notification, does not meet this requirement. Registering, replacing or removing an authenticator MUST require an existing registered authenticator or an out-of-band process that the organisation defines. The platform SHOULD verify authenticator attestation against a list the organisation controls, and SHOULD offer confirmation on a separate registered device that itself shows each proposal's operation, targets and the start of its digest. Rejecting a proposal needs no verification.
- **R4.8** When proposals are approved together, the approval view MUST show each one in full and let the person reject each one individually. The platform SHOULD limit how many proposals one confirmation covers. Requests under R1.6, and kinds of change that the organisation marks as high impact, MUST be approved individually.
- **R4.9** The person approving MUST be shown, rendered from the bytes the approval binds to (R4.3):
  - the exact content that will run, as inert text that is never interpreted as markup, links, formulas or control sequences, with invisible, bidirectional-control and non-ASCII characters made visible;
  - the targets and how each is bound (R4.4), the identity the change runs as, the credentials it references, the network destinations it may reach, and any dependency not pinned by digest;
  - the requesting person, the managed systems the session read, the session's other proposals that are pending or were recently approved, the proposal's age, and when the approval will expire;
  - the evidence the proposal cites, as source records with their source and retrieval time, with records from sources that outsiders can write to, such as logs, tickets, alerts, web pages and the AI's memory, marked as such.

  Text the AI wrote, such as its reasoning and summaries, MUST be marked as written by the AI, and MUST NOT be presented as evidence.
- **R4.10** A person MAY approve a pre-approved automation. An automation MUST start only by the schedule or deterministic trigger fixed in its approval, and the AI MUST NOT be able to start, parameterise or time it. Its targets and parameters MUST come from its approval, or be derived by deterministic rules from facts that the platform verifies about managed systems, never from free text it reads, such as log lines, alert text or tickets, nor from the AI. It MUST expire, or be approved again, within a period the organisation sets. Each run MUST be authorised against the current permissions of the person who approved it, and MUST NOT start once that person's access has ended. Creating, modifying, widening, retargeting, enabling, disabling, renewing or deleting an automation is a change under this rule.
- **R4.11** The platform MUST NOT let a change run without approval of each instance, unless it falls in a pre-approved class that:
  - people defined in advance, limited to named operations, and to named targets or targets selected by attributes the platform verifies;
  - states a maximum scope and rate, such as targets per change and changes per period, which the platform enforces;
  - expires, or is approved again, within a period the organisation sets;
  - excludes requests under R1.6 and kinds of change the organisation marks as high impact;
  - limits any external communication it includes to destinations that meet the condition in R3.3.

  The platform MUST decide membership by deterministic rules over the proposal's operation, over parameters checked against values the class allows, and over attributes of its targets that the platform resolves itself; never over descriptions, labels or other text the AI supplies, and with no model taking part. A class that admits every change to a managed system does not meet this requirement. Creating, modifying, widening, retargeting, renewing or deleting a class is a change under this rule.
- **R4.12** The system MUST document every instruction store. A write to an instruction store by the AI, or by software outside the platform, such as a tool server updating its tool definitions, MUST NOT take effect until a person approves it as a change under this rule. Memory the AI writes MUST be stored with its origin, MUST be marked as written by the AI wherever it is shown or placed in the AI's context, and MUST NOT enter an instruction store or be shown as a source record unless a person approves it as a change.
- **R4.13** The organisation MAY require that kinds of change it defines are approved by someone other than the requesting person, or by more than one person. The platform SHOULD support this.

**Rationale.** NIST SP 800-207 tenet 6 requires that authorisation be "dynamic and strictly enforced before access is allowed". For a change made by an AI, the approval is that enforcement, and it holds only if it binds to what will actually run: the content and everything it pulls in, the targets as they were when the person looked, one execution, and a time window. It also holds only if a person with the authority to make the change gave it. A signed-in session proves who opened it, not who pressed the button, and software now acts inside people's browsers and AI clients; user verification bound to the proposal's digest is something that such software cannot complete without the person. The approver is the last line of defence against a steered AI, so what they see must come from the platform, rendered so that it cannot deceive, and not from the AI's own account of events. Some frameworks let people promote an agent to act without approval of each action once it has a good record. For changes to production infrastructure this specification takes the stricter position: a record earns the AI nothing, because one injected log line can steer a model with a perfect history. What may run unattended is decided, bounded and time-limited by people, never by the AI (R4.10, R4.11).

**Verification.**

1. Ask the AI to make a change. It produces a proposal, and nothing changes until a person approves it.
2. Call the approval interface with the AI's own identity, an unattended session's identity and a machine token. All are refused.
3. Try to approve as a signed-in person who lacks permission to make the change on its target, and as a person without access to a system the session read. Both are refused. Approve as a person who holds the permission, then remove it before the change runs. It does not run.
4. Try to approve without user verification: by replaying the approval request from the browser session, by driving the browser with an automation tool, and with an authenticator registered without an existing one. Each is refused. Start the verification for one proposal and present it for another. It is refused.
5. Alter a proposal after it has been shown for approval, and again after it has been approved. The approval is refused, and execution refuses the altered content. Re-tag a container image that the proposal uses. What runs is unchanged, or execution refuses it. Approve a change that includes a computed plan, then alter the target so that the plan would differ. Execution refuses it.
6. Approve a change, then replace its target with a new object of the same name before it runs. Execution refuses it.
7. Let an approval pass its expiry. The change cannot run. Run an approved change, then try to run it again with the same approval. It is refused.
8. Put invisible, bidirectional-control and look-alike characters, markup and a formula in a proposal's content. The approval view shows them as visible inert text.
9. Look at the approval view. Everything R4.9 lists is shown, the evidence links to source records with their sources, and the AI's own text is marked as such.
10. Approve a set that contains a credential reset. It is refused until the reset is approved on its own.
11. Ask the AI to start, edit, retarget or widen an automation or a class, or to change an instruction store. Each becomes a proposal. Submit a proposal that a class would admit only because of its description. The class does not admit it.

### Rule 5. Record every decision where the AI cannot change it

What the AI was asked, what it read, what it proposed, who decided and what ran is kept where the AI cannot touch it.

- **R5.1** For every proposal, including those rejected, withdrawn and expired, and for every change run under an automation or a class, the platform MUST keep a decision record of:
  - the requesting person and the session;
  - the exact content and its digest, and the targets and how each is bound;
  - the evidence cited, as references to source records or a snapshot of them, with their retrieval time;
  - for each approval or rejection, who gave it, when, through which channel and with which authenticator, and any external change-record reference the organisation uses;
  - the automation or class that applied, if any, and its membership decision;
  - for each execution, when it started and ended, the steps it ran, and its result.
- **R5.2** For every request the platform makes or refuses on the AI's behalf, including reads, changes, refused requests, public retrievals, search queries and external communication, the platform MUST record the operation, its parameters, the system or destination, the session, the requesting person, the authorisation decision, the time and the result status. The platform SHOULD keep the content returned to the AI for a documented period.
- **R5.3** The platform MUST record each instruction a person gives the AI and each output the AI returns.
- **R5.4** The AI MUST NOT be able to modify or delete any record.
- **R5.5** Text from the AI or from managed systems MUST be kept in records as data. Interfaces MUST NOT interpret it as markup, formulas or control sequences, and exports MUST encode it so that common consumers do not either, for example by neutralising a leading `=`, `+`, `-` or `@` in CSV and escaping control characters and line breaks.
- **R5.6** Records MUST be kept for at least a period the organisation sets, and never for less than a default the system documents. They MUST NOT be removed because of volume before that period ends.
- **R5.7** Access to records that contain content read from managed systems MUST be limited as R1.10 requires.
- **R5.8** Records SHOULD be tamper-evident, for example by chaining their hashes or writing them to storage that the people who run the platform cannot alter.
- **R5.9** Records SHOULD be exportable to the organisation's own logging or SIEM platform.
- **R5.10** The system MUST NOT be described as keeping immutable or tamper-proof records unless a mechanism makes them so.

**Rationale.** When something goes wrong, the questions are what the AI was told, what it read, what it did and who let it. Records of requests, with their parameters, are what show after a leak what could have left. NIST SP 800-207 tenet 7 calls for the enterprise to collect "as much information as possible about the current state of assets, network infrastructure and communications"; these records are that information for the AI. Section 3 of NIST SP 800-207 gives the same duty to its policy engine, which "makes and logs the decision (as approved, or denied)". Text from the AI and from managed systems is attacker-influenced, so it must not become an attack on the people and tools that later read the records.

**Verification.**

1. Approve a change, reject another and let a third expire. Each decision record has every field in R5.1.
2. Have the AI make a read, a refused request and a search. Each record has every field in R5.2, and each instruction you gave and each output the AI returned is recorded (R5.3).
3. From the AI's environment, try to delete or edit a record. It cannot.
4. Make the AI write text containing markup, a spreadsheet formula and control characters, and view its record in the platform and in an export opened in a spreadsheet. All of it shows as plain text.
5. Generate records beyond the volume the documentation states, and check that records inside the retention period are still there.

## 6. Residual risk

This section is informative. A conforming system still leaves risks that the organisation must accept or manage in other ways, and [CONFORMANCE.md](CONFORMANCE.md) requires every conformance statement to assess them.

1. **Misleading conclusions and forged evidence.** A steered or mistaken AI can give a person a wrong diagnosis, and a compromised managed system can return records that its attacker wrote. The rules stop the AI from acting on a diagnosis by itself, and R4.9 shows the approver source records and where they came from, but a source record is not proof, and a person can still act on the AI's advice by other means.
2. **Harm a person approves.** The rules guarantee that the approver saw the exact content, not that they understood it. A steered AI can hide data in an approved change, for example as an encoded value written to a system that others can read; split a harmful outcome into proposals that each look harmless; or count on fatigue when many proposals arrive together.
3. **Harm within automations and classes.** Changes that people have pre-approved run without a person looking at each one. The bounds in R4.10 and R4.11 limit the damage; they do not prevent it.
4. **Breadth of the requesting person's access.** A steered AI can read anything the requesting person can. Where R1.8 is not used, that can include an administrator's whole estate.
5. **Disclosure to the disclosed third parties.** Everything the AI reads reaches the model provider. Search queries are free text that the AI writes, so anything it read can reach the search provider.
6. **Signalling through permitted choices.** The parameters of requests to managed systems are seen by whoever controls or watches those systems, such as an attacker on a host the AI is investigating. Which permitted search results the AI opens, and when, can also carry information. The bandwidth of these channels can be significant.
7. **Links a person opens.** A person can still open a link the AI wrote after seeing its full address, and the address can carry data.
8. **Software in the person's own session.** Software outside the conformance target that acts in a person's session can use the AI as that person. It can also show the person one proposal while they confirm another, unless the confirmation happens on a separate device that shows the proposal itself (R4.7).
9. **What has already been read.** Revoking access stops further requests, within the documented delay, but cannot remove what the AI has already read.
10. **Load, cost, volume and availability.** Requests within the limits of R1.9 still consume the managed systems' capacity and the model's cost, and a flood of requests can make records harder to review. Because checks fail closed (R0.2), an outage of a service the platform depends on stops the AI, by design.
11. **Compromise outside the AI.** A compromised person together with their authenticator, a compromised platform, or a compromised third party is outside this specification.

## 7. Relation to change management

This section is informative. The rules fit the change practices most IT organisations already follow:

- A **standard change** corresponds to a pre-approved class (R4.11).
- A **normal change** is a proposal and its approval (R4.1 to R4.9). An advisory board's approval can be required as an additional approver (R4.13), and the external change record can be referenced in the decision record (R5.1).
- An **emergency change** follows the same path, faster. There is no bypass: any setting that lets the AI change a system without Rule 4 takes the system out of conformance (R0.3).
- A **change window** is a proposal with a stated start time (R4.3) and an expiry that covers the window (R4.5).
- A **backout plan** can be proposed and approved with the change it reverses.
- **Break-glass access** for people is outside this specification, which governs the AI. When the platform's own checks cannot run, the AI stops (R0.2), and people use their own access.

## 8. Out of scope

- The safety and quality of the model itself: hallucination rates, alignment, content filtering, training data and the model supply chain.
- Securing AI applications generally, such as chatbots and retrieval systems, beyond the AI operations system defined in section 1.
- The security of the platform itself, including its code, hosting, operators and the security of its connections to managed systems, and the platform vendor's own access to its customers' environments. They matter, and [CHECKLIST.md](CHECKLIST.md) asks about them, but they are vendor due diligence, not properties of the AI.
- People's own access to managed systems outside the platform.
- Certification. This specification defines requirements and self-assessment ([CONFORMANCE.md](CONFORMANCE.md)); it does not certify anyone.

## 9. References

- S. Rose, O. Borchert, S. Mitchell, S. Connelly. *Zero Trust Architecture*. NIST Special Publication 800-207, August 2020. <https://doi.org/10.6028/NIST.SP.800-207>. Quoted from section 2.1 (tenets), section 3 (logical components), section 3.3.1 (trust algorithm) and section 5.7 (non-person entities).
- OWASP GenAI Security Project. *OWASP Top 10 for Large Language Model Applications*, version 2026, published on Zenodo on 4 August 2026. <https://doi.org/10.5281/zenodo.22109015>
- OWASP GenAI Security Project, Agentic Security Initiative. *OWASP Top 10 for Agentic Applications 2026*. December 2025. <https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/>
- OWASP GenAI Security Project. *Agent Control Standard*, version 0.1.3, 9 September 2026. <https://github.com/GenAI-Security-Project/agent-control-standard/tree/v0.1.3>
- Model Context Protocol. *Specification*, version 2026-07-28: Overview; Authorization, Security Considerations; Client, Elicitation. <https://modelcontextprotocol.io/specification/2026-07-28>
- Meta. *Agents Rule of Two: A Practical Approach to AI Agent Security*. 31 October 2025. <https://ai.meta.com/blog/practical-ai-agent-security/>
- Simon Willison. *The lethal trifecta for AI agents: private data, untrusted content, and external communication*. 16 June 2025. <https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/>
- Josh Woodruff (MassiveScale.AI). *The Agentic Trust Framework: Zero Trust Governance for AI Agents*. Cloud Security Alliance blog, 2 February 2026. <https://cloudsecurityalliance.org/blog/2026/02/02/the-agentic-trust-framework-zero-trust-governance-for-ai-agents>
- MassiveScale.AI. *Agentic Trust Framework*: specification version 0.9.1, conformance specification version 0.9.0 and maturity model, April 2026. <https://github.com/massivescale-ai/agentic-trust-framework/tree/v0.9.1>
- B. Campbell, J. Bradley, N. Sakimura, T. Lodderstedt. *OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens*. RFC 8705, February 2020.
- D. Fett, B. Campbell, J. Bradley, T. Lodderstedt, M. Jones, D. Waite. *OAuth 2.0 Demonstrating Proof of Possession (DPoP)*. RFC 9449, September 2023.
- S. Bradner. *Key words for use in RFCs to Indicate Requirement Levels*. RFC 2119, March 1997. B. Leiba. *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*. RFC 8174, May 2017.
