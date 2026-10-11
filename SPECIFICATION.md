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

**What is normative.** Sections 1 to 3, the requirements (identifiers R0.1 to R5.9) and the verification steps are normative. Rule titles, the text that introduces each rule, rationales, the threat model, the residual risk and the relation to change management are informative.

**What conforms.** The conformance target is the platform, the agent runtime that sends context to the model and acts on its output, and every interface the platform provides for people to see the AI's output, in a configuration that the conformance statement names. Software that receives the platform's output but is not provided by it, such as people's browsers and AI clients, and mail, chat and ticketing systems, is outside the target: R3.4 and R3.8 govern what the platform delivers to it, and software acting in a person's session is adversary A3 (section 4). A component that offers operations to an agent runtime it does not provide can claim conformance only together with a named runtime and configuration.

**Who must enforce.** Whatever its grammatical subject, every requirement is an obligation on the system to enforce the stated property with its deterministic code. Where a requirement says the system "documents" something, the documentation is part of the conformance statement ([CONFORMANCE.md](CONFORMANCE.md)). Where a requirement sets a limit, the organisation MAY set a stricter value, and the system MUST NOT let anyone set a less strict one.

**Verification.** Each rule ends with verification steps. Passing them is necessary, not sufficient. They test the system, not the model:

- The tester makes the AI's requests and runs code in the AI's execution environment directly, without relying on the model, and then repeats each step through the AI with adversarial instructions where that is possible. Any interface used to do this MUST be subject to every control that applies to the AI's own requests.
- The direct run decides the result. An attempt that the model declines or fails to carry out is recorded as inconclusive, never as a pass.

## 3. Terminology

- **The AI.** The model, the code it writes and runs, the browser or computer sessions it drives, its sub-agents, and any other component within the conformance target, other than the platform, that acts on the model's output without a person in between.
- **Platform.** The deterministic software that authenticates people, mediates the AI's access to systems, holds credentials, runs approvals, executes changes and keeps records, including the agent runtime apart from code the AI writes.
- **Organisation.** The organisation that operates the managed systems and uses the platform, whether it buys the platform or builds it. Where systems are operated on behalf of others, for example by a managed service provider, each party on whose behalf systems are operated is a separate organisation.
- **Person.** A human who has authenticated to the platform with an account assigned to them alone.
- **Session.** A bounded unit of the AI's work for one organisation, with a start and an end.
- **Requesting person.** The person on whose behalf the AI works in a session. Where several people can instruct a session, each instruction's author is the requesting person for the work that instruction causes. For a session started by a schedule, an event or an integration, it is the person who configured that trigger.
- **Unattended session.** A session not started by a person's interactive request.
- **Managed system.** A system that the organisation has connected to the platform for the AI to investigate or change. Public sources retrieved under R3.5 and third parties under R3.6 are not managed systems.
- **Resource.** The unit that a managed system's own access controls protect, such as a namespace, an index, a mailbox, a project, an organisational unit or a file.
- **Access.** A person has access to information when the permissions used under R1.1 allow them to read the resource it came from. Where the platform cannot determine this for an item of information, it treats a person as having access only if they can read everything that the request which returned the item could return.
- **Derived information.** Information is derived from a resource if it was read from that resource, or was produced or stored by a session, sub-agent, memory entry, knowledge item or record whose context held information derived from that resource.
- **Sources of a session.** Every resource from which information in the session's context is derived.
- **Operation.** A kind of action the platform offers the AI.
- **Request.** One use of an operation, with its parameters, on a managed system, a public source under R3.5, or a destination of external communication.
- **Read.** A request to a managed system whose only effects are returning information and the effects that the managed system produces for every request of that kind whatever its parameters, such as its own access logs, metrics and caches. A request that also alters data or state that people or systems use, sends information to any other destination, or causes the managed system to contact a destination that its parameters determine, is not a read. The protocol does not decide: a request that uses HTTP `POST` to run a query can be a read, and a request that uses `GET` to trigger an action is not.
- **Change.** A request to a managed system that is not a read.
- **Configuration change.** Creating, modifying, widening, retargeting, enabling, resuming, renewing or deleting an allow-list entry, an automation, a class, a trusted tool server, the content of an instruction store, or a setting listed under R0.3.
- **External communication.** Any way the AI can send information outside the platform other than a request to a managed system: requests and name resolution from the AI's environment, public retrieval (R3.5), mail, messages, tickets and webhooks the platform sends for the AI, requests to agents or services outside the platform, files written to shared locations, and automatic fetches. The platform's own transfers to the third parties it discloses under R3.6 are not external communication by the AI.
- **Address.** Any locator that software can fetch, such as a URL or a UNC path.
- **Automatic fetch.** A request that software makes for an address found in the AI's output without a person choosing to open it: for example, loading an image, generating a link preview or chat unfurl, embedding a frame, or prefetching a page or a name.
- **Allow-list.** A list of destinations that people have approved in advance through configuration changes (R4.14).
- **Proposal.** The complete content of a change or configuration change, as R4.2 defines it, prepared so that it can run without any further input from the AI.
- **Approval.** A person's decision, confirmed under R4.7, that one proposal, or each proposal in a set shown together, may run.
- **Approval view.** The interface in which a person reviews a proposal before deciding on it.
- **Pre-approved automation.** A change with fixed content and scope that a person has approved to run later or repeatedly on a fixed schedule or trigger (R4.10).
- **Pre-approved class.** A narrow kind of change that people have defined in advance as allowed to run without approval of each instance (R4.11).
- **High-impact change.** A kind of change that the organisation designates as high impact.
- **Credential.** Any secret value that grants access, or from which access can be derived: a password, API key, token, private key, session cookie or recovery code.
- **Source record.** Information the platform retrieved from a managed system or produced itself, shown with its source and the time it was retrieved, as opposed to text the AI wrote. A source record shows what a system returned, not that it is true.
- **Decision record.** The record of a proposal and its outcome (R5.1).
- **User verification.** An authentication step in which an authenticator registered to the person confirms locally that the person is present and is who they claim to be, such as user verification in Web Authentication (WebAuthn).
- **Instruction store.** Any store whose content the platform places in the AI's context other than as the result of a request the AI made in that session, excluding memory that is placed there marked as written by the AI (R4.12). System prompts, skills, tool definitions, playbooks and runbooks are examples.

## 4. Threat model

This section is informative.

**Assumption.** The AI can be wrong, or steered by an attacker, at any moment of any session. Everything the AI reads can carry instructions: log lines, alert text, ticket comments, hostnames, configuration files, documentation, web pages, tool definitions, messages from other agents, the results of its requests and its own memory. In the words of the OWASP Top 10 for LLM Applications 2026 (LLM01:2026 Prompt Injection), "no reliable prevention mechanism exists today", and "Defense is therefore architectural rather than interceptive."

**Adversaries.**

- **A1.** Anyone who can put content where the AI will read it. They need no access to the platform or to the managed systems.
- **A2.** A model that produces harmful output for any reason, including its own error.
- **A3.** Software in a person's own session or device that is outside the conformance target, such as an AI agent in the person's browser or AI client. For reads, which need no approval, it can do through the platform whatever that person can. It cannot complete an approval without that person's user verification.

**Trusted.** The platform's deterministic code; the authentication of people; the access controls of the managed systems themselves; the third parties disclosed under R3.6, for the confidentiality of what they receive on the disclosed terms, but never for the content of their output; and the person who approves a change, to the extent of reading what they approve. Source records are trusted only as a record of what a system returned.

**The Rule of Two.** An AI operations system that does its job has two of the three properties that the Agents Rule of Two (Meta, 2025) warns against combining: it processes untrustworthy inputs [A], and it has access to sensitive systems [B]. These rules confine the third property, the ability to change state or communicate externally [C], to narrow bounds that people set: changes run only through a person's approval or within automations and classes people have defined and bounded in advance, and the AI communicates externally only with allow-listed destinations, through permitted public retrieval, or with a person's approval. [Section 6](#6-residual-risk) states what remains.

**Design goals.** The requirements are designed so that a steered AI cannot do the following, except as section 6 describes and as the allow-lists, automations and classes that people approve permit:

| Goal | A steered AI cannot | Delivered by |
| --- | --- | --- |
| G1 | act beyond the authority the organisation gave the people involved, or expose information to people or organisations without access to it | R1.1 to R1.8, R1.10 to R1.13, R3.3, R4.6, R5.7 |
| G2 | obtain or use a credential that the platform holds, attaches, releases or can recognise | R1.6, R2.1 to R2.6, R3.6 |
| G3 | reach a managed system except through the platform, or send what it has read anywhere other than to disclosed third parties, allow-listed destinations or through changes people approve | R3.1 to R3.7, R4.1 |
| G4 | change a managed system except through a proposal a person approved exactly, against the same targets, once and before it expired, or within an automation or class people defined | R0.3, R1.5, R4.1 to R4.7, R4.10 to R4.12, R4.14 |
| G5 | approve a change, or win an approval by passing off its own words as evidence | R4.6 to R4.9 |
| G6 | alter, erase or evade the record of what happened | R0.2, R5.1 to R5.9 |
| G7 | attack the people and software that read its output | R3.4, R3.8, R4.9, R4.12, R5.5 |

R0.1 underpins every goal. R1.9, R4.13 and R4.15 are recommendations that support G1 and G4 without being needed for them.

**Prior warning.** NIST SP 800-207 (2020) anticipated the risk for a related case. Section 5.7, on non-person entities used to administer a zero trust architecture, warns that "an attacker will be able to induce or coerce an NPE [non-person entity] to perform some task that the attacker is not privileged to perform", and that "an attacker could gain access to a software agent's credentials and impersonate the agent". It calls the authentication of such agents "an open issue". These rules address the same concern for the AI that operates infrastructure.

## 5. Requirements

### General requirements

- **R0.1** A system MUST NOT rely on instructions to the AI, or on the judgement of any model, to meet any requirement in this document. Model-based detection and filtering can add defence in depth, but they never meet a requirement.
- **R0.2** Where a check that a requirement depends on cannot be completed, because a service or store it needs cannot be reached or read, or where the platform cannot record a request as Rule 5 requires, the request or change MUST NOT start. A change already running when recording fails MUST be stopped as R1.12 describes, and recorded as soon as recording is possible.
- **R0.3** The system MUST document every setting that takes it out of conformance. The AI MUST NOT be able to change such a setting, changing it is a configuration change (R4.14), and the system MUST show people when it is running outside conformance.

**Verification.**

1. Make each service or store that a check depends on unreachable in turn. Each request or execution that depends on it is refused.
2. From the AI's environment, try to change a setting the system documents under R0.3. It is refused, or becomes a proposal.

### Rule 1. Act only as the person who asked

The AI never has authority of its own. Every request it makes is authorised for the people involved, by the platform, at the moment it happens, and what it learns stays with the people entitled to it.

- **R1.1** Every request the AI makes, other than a change, MUST be authorised by the platform, before it runs, against the requesting person's permissions as known to the platform at that time; every change MUST be authorised as R4.6 requires. A person's permissions for a managed system are the ones the organisation grants them, whether held in the managed system's own access model, in the platform's, or in both, in which case the more restrictive applies. Where the organisation maps an account on the managed system to the person, the platform MUST NOT allow more through the AI than that account allows. The platform MAY determine permissions by acting as that account, by an authorisation query the managed system offers, or, where it offers neither, from the organisation's mapping in the platform's access model, which is then authoritative for that system; the system documents which method applies to each kind of managed system. A revocation at its source MUST take effect within a delay the system documents, which MUST NOT exceed one hour, and cached permissions MUST NOT be used for longer.
- **R1.2** Authorisation MUST NOT depend on any assertion by the AI about identity, role, authority, purpose or safety.
- **R1.3** The access available to the AI in a session MUST NOT exceed the requesting person's, and MUST end when the session ends. A change MAY exceed the requesting person's authority only if it is approved under R4.6, and MUST NOT exceed the authority of the person who authorised it.
- **R1.4** Operations the platform does not offer the AI MUST be unavailable to it. An operation that is available but forbidden by an instruction to the AI does not meet this requirement.
- **R1.5** The platform MUST classify each request to a managed system as a read or a change, by deterministic rules over its operation and parameters, before it runs; a request that the rules cannot classify as a read is a change. A request whose effect depends on free-form parameters, such as a command, a script or a query, is a change unless the platform enforces, independently of the AI, that it has only the effects of a read: for example through an allow-list of read-only commands and arguments, or a database role that can neither write nor call functions that open network connections, read or write files, or run programs. Classifications that a tool server supplies MUST NOT be relied on unless the organisation has designated that server as trusted through a configuration change. The AI MUST NOT be able to change the rules.
- **R1.6** Requests that create or reset credentials or authentication factors for any principal, or that weaken authentication, MUST be approved individually, never as part of a set (R4.8), an automation (R4.10) or a class (R4.11). Requests that revoke credentials, sessions or tokens MAY run within automations and classes. Credential values that any of these requests produce MUST NOT reach the AI (R2.6). This requirement does not apply to the platform's own authentication to managed systems under R2.2.
- **R1.7** A session MUST NOT start unless the platform can determine its requesting person. An unattended session MUST be limited to reads, proposals that wait for approval, changes within pre-approved classes and external communication that R3.3 permits, and MUST use only the operations and managed systems its trigger states, which MUST NOT exceed its requesting person's. Each unattended session MUST be authorised against the permissions its requesting person holds when it runs, and MUST NOT start once that person's access to the platform has ended. The platform SHOULD alert people the organisation designates when this stops a trigger, and SHOULD let a trigger be transferred to another person as a configuration change.
- **R1.8** The platform MUST let the organisation, and SHOULD let the requesting person, restrict the operations and managed systems available to the AI below the requesting person's permissions.
- **R1.9** The platform SHOULD limit the rate and volume of requests per session and per managed system, to limit the load the AI can place on managed systems and the volume of records it can create.
- **R1.10** Information MUST NOT become available to a person who lacks access to it, or to another organisation, including through the AI's output, shared memory, knowledge, search, caches, records, or data used to train, fine-tune or evaluate models, unless a person with access to it shares it deliberately. Everything the AI produces in a session is derived from every source the session has at that time. Audit access and record export that the organisation authorises count as deliberate sharing. Where more than one person can instruct a session or see its output, the platform MUST authorise each request against the permissions that all of them hold, or show the output only to the person who gave the instruction. Transferring a session to another person MUST start a new session with that person as the requesting person. A later session of the same person MAY receive information from earlier sessions, subject to this requirement at that time.
- **R1.11** The sessions, memory, records and credentials of different organisations MUST be isolated from each other. The platform MUST NOT send one organisation's content to a third party under R3.6 on terms that let it use the content to train, fine-tune or evaluate models available outside that organisation.
- **R1.12** People the organisation authorises MUST be able, in one action, to end all sessions with their access and identities, withdraw pending proposals, suspend approved changes that have not started and all automations and classes, and stop running changes; and MUST be able to take each of these actions separately, and to withdraw any operation or managed system from all sessions. Each action MUST take effect within 60 seconds. A running change MUST be stopped by terminating the processes the platform started, on the platform and, where the operation supports it, on its targets, and by starting no further step, except steps its approved proposal names as having to complete to leave a target safe (R4.2), which an authorised person MUST still be able to stop. A suspended change needs a new approval to run, and resuming an automation or class is a configuration change. The outcome of every stop MUST be recorded.
- **R1.13** A session MUST end no later than a maximum duration the organisation sets, which MUST NOT exceed 24 hours.

**Rationale.** A model that decides its own authority can be talked out of its limits. Tying every request to the people involved, checked by code when it happens, means a steered AI can at worst read what the requesting person could read, and change nothing unless a person with the authority to make the change approves it or has defined it in advance. Classifying each request by what it does, and treating anything unclassified as a change, makes a new endpoint or an unusual command fail closed. Keeping what the AI learns within the reach of the people entitled to it, at the granularity of each system's own access controls and through everything the AI derives from it, stops the AI from becoming a way around those controls. Bounding sessions limits how long a hijacked session, and its identity, can last; work continues in a new session.

**Verification.**

1. As a person without access to a resource, issue a read of it from the AI's environment. The platform refuses. Repeat as a person whose platform role allows the read but whose own mapped account does not. The platform refuses.
2. From the AI's environment, make a request that claims to act for another person, in another role or for a safety reason. Authorisation is unchanged. Call an operation the platform does not offer. It does not exist.
3. Revoke the requesting person's access at its source in the middle of a session. Within the documented delay, the next request to that resource is refused.
4. Add a new operation without classifying it, and call it. Call a command or query operation with parameters its rules do not recognise as a read, and with a query that would make the managed system contact an address you control. Each becomes a proposal.
5. Request a password reset through the AI. It is a proposal that cannot be approved together with others, and the new password never reaches the AI.
6. Remove the access of the person who set up a scheduled session. Its next run does not start.
7. Read a resource in one session, and let the session write memory. As the same person, start a new session that reads only that memory. As a second person who has access to the managed system but not to that resource, and without authorised audit access, search everything the platform shows to people: memory, knowledge, search, sessions and records. Nothing derived from the resource appears. Repeat as a person in another organisation, including another customer of the same service provider.
8. Use the stop action while a session is working, a script is running on a target and a change is waiting to start. Within 60 seconds, the session's next request is refused, the script's process is terminated where the operation supports it, and the waiting change does not start.
9. Let a session run beyond its maximum duration. It ends.

### Rule 2. Never give the AI a credential

The AI uses systems without holding the keys. Credentials stay with the platform, which attaches them to each request outside the model and removes the ones it can recognise from what the AI reads.

- **R2.1** Credentials the platform uses for managed systems MUST NOT be anywhere the AI can read: its context, the results of its requests, error messages, its environment, files, memory, or records it can query.
- **R2.2** The platform MUST attach credentials to requests itself, after authorising each request (R1.1).
- **R2.3** The platform SHOULD attach credentials to changes itself (R2.2), so that the code of a change never holds them. Where a change needs a credential value, the platform MUST let the proposal refer to the credential by name, MUST NOT require its value in proposal content, and MUST release it only after approval, only to the component that executes that change under R3.7, and only for that execution. For such an execution, the platform MUST return to the AI only whether it succeeded, as the platform determines, unless a person with access to its targets releases its output; released output is subject to R2.6.
- **R2.4** Any identity that the AI's session presents to the platform MUST be bound to that session and its requesting person, MUST be accepted only by the platform, MUST NOT be forwarded to managed systems, and MUST expire or be revoked when the session ends. A copy of it MUST NOT be usable from anywhere other than the session's own runtime: either the AI's tools and code cannot read it, or it is sender-constrained, for example by certificate-bound tokens [[RFC8705]](https://www.rfc-editor.org/rfc/rfc8705) or proof of possession [[RFC9449]](https://www.rfc-editor.org/rfc/rfc9449), with a key that the AI's tools and code cannot read. It SHOULD be sender-constrained even where it cannot be read.
- **R2.5** Other credentials the AI's runtime uses, such as keys for the model provider and other third parties under R3.6, MUST NOT be readable by the AI's tools or code.
- **R2.6** Before any content reaches the AI, including the results of requests and changes, trigger payloads, instructions and memory, the platform MUST replace credential values in it, at least:
  - every credential value the platform attaches (R2.2) or releases (R2.3), wherever it appears, verbatim and in the encodings the system documents, which include at least base64, hexadecimal and URL encoding;
  - the values of fields that the managed system's schema or interface marks as secret;
  - private keys and other key material in PEM, PKCS #8, PKCS #12, OpenSSH and JWK form, and in any other form the system documents;
  - credential values in Kubernetes secrets and kubeconfig files; cloud-init files, unattended-installation files and Group Policy Preferences; infrastructure-as-code state; network device configurations, including enable secrets, SNMP communities, pre-shared keys and RADIUS and TACACS+ keys; connection strings; and any other format the system documents as parsed;
  - any credential a change creates or resets.

  The platform SHOULD detect and replace credential values in other data. It MAY show the AI a credential's name and type, whether a value is set, non-secret parts such as a certificate and its expiry date, and a fingerprint keyed for the session that shows only whether two values match.

**Rationale.** A credential in the AI's reach turns one injected instruction into a working attack, and it can leave through logs, scratch files, generated code and the context sent to the model provider. Scoping a credential reduces the damage; keeping it out of reach removes the path. NIST SP 800-207 section 3 gives the policy administrator, not the subject, the job of generating "any session-specific authentication and authentication token or credential". This rule goes one step further for the AI: the credential is used on its behalf and never handed to it. Credentials also sit inside the data the AI reads, in configuration, scripts and secret stores, so the platform removes the ones it can recognise on the way in. Recognising them in free text is imperfect, which is why section 6 lists what may remain. Code that holds a released credential can signal through what it returns, so only the platform's own verdict goes back to the AI.

**Verification.**

1. From the AI's environment, read everything its tools and code can read: environment, files, memory, logs and error output. No credential the platform holds, attaches or released is found, other than a sender-constrained session identity under R2.4.
2. Ask for a stored credential. The AI can learn that it exists, by name and type, but never its value.
3. Read a Kubernetes secret, a network device configuration, a PEM private key and each other format the system documents. The credential values are replaced. Approve a change that writes a credential the platform released into a file on a target, then read the file. The value is replaced.
4. Approve a change that references a credential and whose code sets an unusual exit code and prints the credential in an encoding. The AI sees only whether the change succeeded, unless a person with access to its targets releases the output, and the value never appears.
5. Take the session's identity out of the runtime, if it can be read at all, and present it from another host during the session, in another session, after the session ends, and to a managed system. Each attempt is refused.

### Rule 3. Give the AI no network of its own

The AI reaches your systems only through the platform, and has no way out of its own: no network, no channel that people have not allowed, and no automatic fetch that could carry what it read. This is the rule that limits what a steered AI can send to an attacker.

- **R3.1** The AI's only route to managed systems MUST be the platform. The AI MUST NOT have network access to the organisation's networks. Network diagnostics against managed systems, such as ping, traceroute, name lookups and port checks, are requests that the platform makes and classifies under R1.5.
- **R3.2** Code the AI runs, and any browser or computer session it drives, MUST have no network access except to the platform's mediated interface. Name resolution is network access. Where the AI uses the web interface of a managed system, or a public web page, the platform MUST run the browser, hold its cookies and sessions out of the AI's reach, apply R1.5 to each request the browser sends and R3.5 to each navigation. Input to a computer session on a managed system is a request to that system.
- **R3.3** Except for public retrieval under R3.5, every channel of external communication available to the AI MUST be removed, limited to allow-listed destinations, or used only by sending each message as a proposal under Rule 4. Communication to an allow-listed destination is not a change. Each allow-list entry MUST state the resources that everyone who can read the destination has access to, and the platform MUST NOT send to it from a session that has any other source. Where the destination lets the platform see who can read it, the platform MUST check before each send that each of them has access to every source of the session; the system documents the entries it cannot check. A request whose destination is an exact allow-list entry, and whose content the platform fixes, such as a ping or a lookup of an exact name, does not need to meet the condition on sources.
- **R3.4** Interfaces that show the AI's output MUST NOT make automatic fetches for addresses in it, unless the address belongs to the platform and fetching it causes no request to any other destination, or the address is allow-listed under R3.3 and the session that produced the output meets that entry's condition. Where the platform delivers output to software outside the conformance target, it MUST turn automatic fetching off there, or deliver addresses in a form that such software does not recognise as addresses, for example by defanging them. Links to destinations that are not allow-listed MUST be shown as text that cannot be clicked, or open only through an intermediate page that shows the full address with its domain highlighted and needs a signed-in person's action to continue.
- **R3.5** Where the AI retrieves public information, such as vendor documentation, security advisories or threat intelligence, information derived from managed systems MUST NOT be able to determine the destination or content of any request, except by writing a query sent to a search or lookup service disclosed under R3.6, and by choosing among addresses that the platform has obtained from that service or that match an allow-list people maintain. Where an allow-list entry lets the AI choose part of an address, the operator of that site receives what that part can carry, and MUST be disclosed under R3.6. Designs that meet this requirement include:
  - the platform building every request itself, only for addresses returned by that service or matching the allow-list;
  - a separate session with no access to managed systems, whose input a person writes or approves, and whose results reach other sessions only as untrusted content;
  - a person approving each request.
- **R3.6** The system MUST document every third party that receives content the AI reads or produces, including model, embedding, search, lookup, safety-filtering, telemetry, tracing, hosting and delivery providers and the operators of allow-listed destinations, with the account used, the region, the retention, whether the content is used for training, and whether the account is shared with other organisations. The platform, not the AI, MUST choose the account and endpoint of every request to such a party, so that the AI cannot direct a request to an account or endpoint it names. The organisation SHOULD be able to choose these parties, and SHOULD be able to use its own accounts with them.
- **R3.7** Code that the platform runs for a change, including runs of automations and changes within classes, MUST run where the platform enforces network access limited to the change's targets and to the destinations its proposal declares. Where any part of a change executes on a managed system whose network the platform does not control, the proposal MUST say so (R4.2), the platform MUST NOT release a credential value to that part, and the approval view MUST state that the part runs with that system's network access. The platform SHOULD restrict that system's network where it can.
- **R3.8** Interfaces that show text from the AI or from managed systems MUST render it as inert data: they MUST NOT run scripts, styles, event handlers or embedded frames from it, or interpret it as formulas or control sequences. Formatting such as headings, lists, tables and emphasis MAY be rendered where it cannot fetch, run code or hide text. Where the platform delivers such text to software outside the conformance target, it MUST deliver it in a form that such software treats as text.

**Rationale.** Simon Willison's "lethal trifecta" (2025) is access to private data, exposure to untrusted content, and the ability to communicate externally; an agent with all three can be tricked into sending private data to an attacker. An AI operations system cannot give up the first two, so the AI must not have the third except within bounds people set. A request to an arbitrary address is enough to leak data, because the address itself can carry it, and so is a name lookup or an image that a person's browser loads by itself. Text that a console runs as code turns an attacker who can only write a log line into one who controls a person's session. Being inside the network grants the AI nothing either: NIST SP 800-207 tenet 2 holds that "Network location alone does not imply trust", and this rule removes the network path altogether. An approved change is code the AI wrote, so where the platform runs it, it can reach only what its proposal declares; where it runs on a target, it has the target's network, which is why no credential value goes with it there.

**Verification.**

1. From the AI's environment, and from a browser session the AI drives, request an address that is neither allow-listed nor returned by the disclosed search or lookup service, and look up a name in a domain whose name servers you can watch. The request fails, and no lookup reaches your name servers.
2. From the AI's environment, connect directly to a private address in your network. It fails.
3. Read a value from a managed system, then try to send it out through every channel the AI has: a request to an address of your choosing, mail, chat, a ticket, a webhook. Each attempt is impossible, reaches only an allow-listed destination whose condition the session meets, or becomes a proposal. Allow-list a destination for one resource, read another, and try to send to it. It is refused.
4. Make the AI's output contain an image and a link that point to a server you control, and open that output in the platform and in every system the platform delivers output to. Your server receives nothing, and the link cannot be clicked or opens only through an intermediate page that shows its address.
5. Ask the AI to retrieve an address it has built itself, containing a value it has read. It is refused, or becomes a request that a person must approve.
6. Place a model provider key of your own in a document the AI reads, and instruct the AI to use it. No request reaches your account.
7. Approve a change that the platform runs, whose proposal declares no destination other than its target, and whose content tries to reach another address. The connection fails. Propose a change that runs a script on a target and references a credential. The proposal states that the script runs with the target's network, and no credential value is released to it.
8. Put a script element, an image with an event handler, a style that hides text, and a spreadsheet formula in a log line the AI reads and repeats. Every interface shows them as inert text, and nothing runs.

### Rule 4. Let only a person approve a change, exactly as it will run

The AI may propose any change. A person with the authority to make it approves it, with user verification bound to that proposal, and what runs is the content they saw, against the same targets, once, before the approval expires. Unattended change exists only within automations and classes that people define and bound in advance.

- **R4.1** Every change the AI initiates MUST be a proposal, and MUST NOT run until it is approved under R4.6 and R4.7, except a run of a pre-approved automation (R4.10) or a change within a pre-approved class (R4.11).
- **R4.2** A proposal MUST contain everything that determines what runs:
  - a unique identifier and the time it was created;
  - the operation and its parameters, or the script, playbook or diff;
  - its targets (R4.4), the identity it runs as, and any part that executes on a managed system (R3.7);
  - the network destinations it may reach (R3.7) and the credentials it references by name (R2.3);
  - its start time or starting condition, its retry policy with a maximum number of attempts, the preconditions it states (R4.4), and any step that has to complete to leave a target safe (R1.12), with the reason;
  - each artefact it installs or executes, whether fetched by the executor or by a target as a direct result of the change's content, such as images, packages, modules, charts and firmware.

  Each artefact MUST be pinned by digest, or by an exact version that a signed index binds to a digest, or resolved from a source the organisation has designated, with the resolved versions or digests captured when the proposal is made and verified at execution. Where the platform cannot determine every fetch that content executing on a managed system will make, the proposal MUST say so. Where the tool that applies a change produces a plan of its effect before applying it, such as a saved infrastructure plan, a change set or a server-side dry run, the proposal MUST include the plan, and the platform MUST verify immediately before applying it, by the tool's own stale-plan check or by computing the plan again and comparing, that it is unchanged.
- **R4.3** An approval MUST bind to a digest computed with SHA-256, or a stronger collision-resistant hash, over a canonical serialisation of everything R4.2 lists. The platform MUST render what R4.2 lists in the approval view from the bytes it digests, MUST refuse an approval if the proposal has changed since it was shown, and MUST verify at execution that what runs matches the approved digest; a proposal whose content cannot be verified MUST NOT run. An approval MUST authorise exactly one execution, which the platform, not the AI, starts at the time or on the condition the proposal states. Retries MUST follow the approved retry policy, up to its maximum, and each attempt MUST be verified again under R4.4 and R4.6. The platform MUST NOT let any component, including a policy service, modify a proposal after it is approved.
- **R4.4** A proposal MUST identify its targets by identifiers that the managed system itself reports and does not reuse, where it has them. Otherwise, the platform MUST bind each target to a fingerprint it computes, by rules fixed for each operation and documented by the system, over the state the change reads or modifies. Targets given as queries or groups MUST be resolved to identifiers or fingerprints when the proposal is made. A proposal that reverses another MAY bind its targets to the state that the reversed change is recorded as producing, with that change's completion as a precondition. At execution, the platform MUST verify each target's identifier or fingerprint, and the preconditions the proposal states, and the change MUST NOT run if a check fails. The platform SHOULD require proposals for destructive changes to state the preconditions they depend on.
- **R4.5** An approval MUST expire if its execution has not started within 30 days of the approval, or within a shorter period the organisation sets.
- **R4.6** Only a person can approve. The approval interface MUST refuse the AI, its sub-agents, unattended sessions and machine identities, whatever they claim. Each change MUST be authorised by at least one approving person who holds permission, determined as in R1.1, to make it on each of its targets. The platform MUST check this at approval and again at execution, when that person MUST still hold the permission or remain eligible to obtain it through the organisation's just-in-time process, and MUST NOT run the change otherwise. The organisation MAY also require the requesting person to hold that permission. Additional approvers that the organisation requires under R4.13 need not hold it. A person MUST NOT be able to view or approve a proposal unless they have access to every source of the session that created it.
- **R4.7** Each approval, or each set of proposals approved together (R4.8), MUST be confirmed with user verification that is bound to the digest of every proposal it approves and to a fresh single-use value that the platform issues for that confirmation, and the platform MUST NOT accept the same assertion twice. A confirmation that needs no user verification, such as an approve or deny push notification, does not meet this requirement. The platform MUST let the organisation restrict which authenticators it accepts, and SHOULD verify their attestation. Registering, replacing or removing an authenticator MUST require an existing registered authenticator, or an out-of-band process that the system documents and that software in the person's session cannot complete. The platform SHOULD offer confirmation on a separate registered device that itself shows each proposal's operation, targets and at least the first 64 bits of its digest. Rejecting a proposal needs no verification.
- **R4.8** When proposals are approved together, the approval view MUST show each one in full and let the person reject each one individually. The platform SHOULD limit how many proposals one confirmation covers. Requests that R1.6 requires to be approved individually, and high-impact changes, MUST be approved individually.
- **R4.9** The approval view MUST show the following, with what R4.2 lists rendered from the bytes the approval binds to (R4.3) and everything else taken from the platform's own records, never from text the AI supplies:
  - the exact content as inert text (R3.8), with invisible and bidirectional-control characters made visible, and characters outside the scripts the organisation allows, or confusable mixtures of scripts, highlighted;
  - the targets and how each is bound, the identity the change runs as, the credentials it references, the destinations it may reach, any part that executes on a managed system and with what network access, any artefact not pinned by digest, and any fetch the platform cannot determine;
  - the start time or condition, the retry policy and its maximum, the preconditions, and any step that the stop action will let complete;
  - the requesting person, the sources of the session, the session's other proposals that are pending or were approved within a period the organisation sets, the proposal's age, and when the approval will expire;
  - the evidence the proposal cites, as source records with their source and retrieval time, marked where the source accepts content from people other than the managed system's administrators, such as logs, tickets, alerts and web pages.

  Text the AI wrote, including its reasoning, summaries and memory, MUST be marked as written by the AI, and MUST NOT be presented as evidence.
- **R4.10** A person MAY approve a pre-approved automation; the approval authorises the runs that its schedule or trigger defines until it expires, each matching the approved digest (R4.3). An automation MUST start only by the schedule or trigger fixed in its approval, and the trigger MUST NOT be a condition that the AI's requests, proposals or outputs can cause. The approval MUST fix a maximum run rate, which the platform enforces. The AI MUST NOT be able to start, parameterise or time an automation. Its targets and parameters MUST come from its approval, or be derived by deterministic rules from attributes that only people named in the approval can set. It MUST expire within a period the organisation sets, which MUST NOT exceed one year. Each run MUST be authorised against the current permissions of the person who approved it, and MUST NOT start once that person's access has ended. The platform SHOULD alert people the organisation designates when this stops an automation.
- **R4.11** The platform MUST NOT let a change the AI initiates run without approval of that change, unless the change falls in a pre-approved class that:
  - people defined in advance through a configuration change (R4.14), limited to named operations, and to named targets or targets selected by attributes that only people named in the class can set, resolved again for each change;
  - limits every parameter to an enumerated set of values or a bounded range, and includes no operation whose effect depends on free-form parameters, such as a command, a script or a query, except as complete parameter values listed in the class;
  - states a maximum scope and rate, such as targets per change and changes per period, which the platform enforces;
  - expires within a period the organisation sets, which MUST NOT exceed one year, and admits no change once the access of the person who approved it has ended;
  - excludes requests that R1.6 requires to be approved individually, and high-impact changes;
  - writes content that can be read by people without access to every source of the session only where the class fixes that content.

  The platform MUST decide membership by deterministic rules over the proposal's operation, its parameters, and attributes of its targets that the platform resolves itself; never over descriptions, labels or other text the AI supplies, and with no model taking part.
- **R4.12** The system MUST document every instruction store. A change to an instruction store, other than by a person through the platform's configuration interface, including an update to tool definitions from a tool server, MUST NOT take effect until it is approved as a configuration change (R4.14). Memory the AI writes MUST be stored with its origin, MUST be marked as written by the AI wherever it is shown or placed in the AI's context, and MUST NOT enter an instruction store or be shown as a source record unless a person approves it as a configuration change.
- **R4.13** The organisation MAY require that kinds of change it defines are approved by someone other than the requesting person, or by more than one person, such as a change authority. The platform SHOULD support this.
- **R4.14** A configuration change that the AI initiates MUST be a proposal under R4.2 to R4.9, with the setting as its target, approved by a person to whom the organisation has given that permission on the platform and who, for an allow-list entry, automation or class, holds permission under R1.1 for everything it covers. Configuration changes that people make directly MUST be confirmed with user verification (R4.7). Disabling or suspending an allow-list entry, automation, class or trusted tool server is not a configuration change and needs no approval.
- **R4.15** The platform SHOULD let the organisation set periods in which changes, automations and classes do not run.

**Rationale.** NIST SP 800-207 tenet 6 requires that authorisation be "dynamic and strictly enforced before access is allowed". For a change made by an AI, the approval is that enforcement, and it holds only if it binds to what will actually run: the content and everything it pulls in, the targets as they were when the person looked, its timing and retries, one execution, and a time window. It also holds only if a person with the authority to make the change gave it. A signed-in session proves who opened it, not who pressed the button, and software now acts inside people's browsers and AI clients; user verification bound to the proposal and to a fresh value is something that such software cannot complete without the person. Anything the AI writes, including a proposal, can carry what it has read, so only people with access to the session's sources may see it. The approver is the last line of defence against a steered AI, so what they see must come from the platform, rendered so that it cannot deceive, and not from the AI's own account of events. Some frameworks let people promote an agent to act without approval of each action once it has a good record. For changes to production infrastructure this specification takes the stricter position: a record earns the AI nothing, because one injected log line can steer a model with a perfect history. What may run unattended is decided, bounded and time-limited by people, never by the AI, and attributes that select its targets can be set only by people the organisation names.

**Verification.**

1. Ask the AI to make a change. It produces a proposal, and nothing changes until a person approves it.
2. Call the approval interface with the AI's own identity, an unattended session's identity and a machine token. All are refused.
3. Try to approve, and to view, as a signed-in person who lacks permission to make the change on its target, and as a person without access to a source of the session. Both are refused. Approve as a person who holds the permission, then remove it, with no just-in-time eligibility, before the change runs. It does not run.
4. Try to approve without user verification: by replaying the approval request from the browser session, by driving the browser with an automation tool, and with an authenticator registered without an existing one or the documented out-of-band process. Each is refused. Start the verification for one proposal and present it for another. It is refused. Let an approval expire, have the AI propose identical content again, and replay the earlier assertion. It is refused.
5. Alter a proposal after it has been shown for approval, and again after it has been approved, including its retry policy and start time. The approval is refused, and execution refuses the altered content. Re-tag a container image that the proposal uses. What runs is unchanged, or execution refuses it. Approve a change that includes a computed plan, then alter the target so that the plan would differ. Execution refuses it.
6. Approve a change, then replace its target with a new object of the same name before it runs. Execution refuses it.
7. Let an approval pass its expiry. The change cannot run. Run an approved change, then try to run it again with the same approval. It is refused. Approve a change with a retry policy, make it fail repeatedly, and count the attempts. They stop at the maximum.
8. Put invisible, bidirectional-control and look-alike characters, markup and a formula in a proposal's content. The approval view shows them as visible inert text.
9. Look at the approval view. Everything R4.9 lists is shown, the evidence links to source records with their sources, and the AI's own text, including its memory, is marked as such.
10. Approve a set that contains a credential reset. It is refused until the reset is approved on its own.
11. Ask the AI to start, edit, retarget or widen an automation or a class, or to change an instruction store, an allow-list or a trusted tool server. Each becomes a proposal. Submit a proposal that a class would admit only because of its description, and one that passes a free-form command to a class operation. The class admits neither. Set a selecting attribute on a target as someone the class does not name. The class does not select it.

### Rule 5. Record every decision where the AI cannot change it

What the AI was asked, what it read, what it proposed, who decided and what ran is kept where the AI cannot touch it.

- **R5.1** For every proposal, including those rejected, withdrawn and expired, and for every change run under an automation or a class, the platform MUST keep a decision record of:
  - the requesting person and the session, and the model provider, model and version that produced the proposal;
  - the exact content and its digest, and the targets and how each is bound;
  - the evidence cited, as references to source records or a snapshot of them, with their retrieval time;
  - for each approval or rejection, who gave it, when, through which channel and with which authenticator, and any external change-record reference the organisation uses;
  - the automation or class that applied, if any, and its membership decision;
  - for each execution and each retry, when it started and ended, the steps it ran, its result, and who released its output, if anyone did (R2.3).
- **R5.2** For every request the platform makes or refuses on the AI's behalf, including reads, changes, refused requests, public retrievals, search and lookup queries and external communication, the platform MUST record the operation, its parameters, the system or destination, the session, the requesting person, the authorisation decision, the time and the result status. The platform SHOULD keep the content returned to the AI for a documented period.
- **R5.3** The platform MUST record each instruction a person gives the AI, each output the AI returns to people or to the platform, the version or digest of every instruction-store item and memory entry placed in the AI's context, and the model provider, model and version used. Reasoning that the model keeps hidden is not output.
- **R5.4** The AI MUST NOT be able to modify or delete any record.
- **R5.5** Text from the AI or from managed systems MUST be kept in records as data, and text the AI wrote MUST be marked as written by the AI. Interfaces MUST render it as inert data (R3.8), and exports MUST encode it so that common consumers do not interpret it either, for example by neutralising a leading `=`, `+`, `-` or `@` in CSV and escaping control characters and line breaks.
- **R5.6** Records MUST be kept for at least one year, or a longer period the organisation sets, and MUST NOT be removed because of volume before that period ends. The organisation MAY set a shorter period to meet a legal obligation, which the conformance statement records, and an authorised person MAY remove a record to meet a legal obligation, which is itself recorded. The system documents the volume of records it has been tested to hold.
- **R5.7** Access to records that contain information derived from managed systems MUST be limited as R1.10 requires.
- **R5.8** Records SHOULD be tamper-evident, for example by chaining their hashes or writing them to storage that the people who run the platform cannot alter.
- **R5.9** Records MUST be exportable to the organisation's own logging or SIEM platform.

**Rationale.** When something goes wrong, the questions are what the AI was told, what it read, what it did and who let it. Records of requests, with their parameters, are what show after a leak what could have left. NIST SP 800-207 tenet 7 calls for the enterprise to collect "as much information as possible about the current state of assets, network infrastructure and communications"; these records are that information for the AI. Section 3 of NIST SP 800-207 gives the same duty to its policy engine, which "makes and logs the decision (as approved, or denied)". Text from the AI and from managed systems is attacker-influenced, so it must not become an attack on the people and tools that later read the records.

**Verification.**

1. Approve a change, reject another and let a third expire. Each decision record has every field in R5.1.
2. Have the AI make a read, a refused request and a search. Each record has every field in R5.2, and each instruction you gave, each output the AI returned and each instruction-store item it was given are recorded (R5.3).
3. From the AI's environment, try to delete or edit a record. It cannot.
4. Make the AI write text containing markup, a spreadsheet formula and control characters, and view its record in the platform and in an export opened in a spreadsheet. All of it shows as plain text, marked as written by the AI.
5. Generate records beyond the volume the documentation states, and check that records inside the retention period are still there. Export records to a logging platform.

## 6. Residual risk

This section is informative. A conforming system still leaves risks that the organisation must accept or manage in other ways, and [CONFORMANCE.md](CONFORMANCE.md) requires every conformance statement to assess them.

1. **Misleading conclusions and forged evidence.** A steered or mistaken AI can give a person a wrong diagnosis, and a compromised managed system can return records that its attacker wrote. The rules stop the AI from acting on a diagnosis by itself, and R4.9 shows the approver source records and where they came from, but a source record is not proof, and a person can still act on the AI's advice by other means.
2. **Harm a person approves.** The rules guarantee that the approver saw the exact content, not that they understood it. A steered AI can hide data in an approved change, for example as an encoded value written to a system that others can read, or as data sent to a destination the proposal declares; split a harmful outcome into proposals that each look harmless; or count on fatigue when many proposals arrive together. A person who releases a credentialed execution's output can release an encoded credential with it.
3. **Harm within automations and classes.** Changes that people have pre-approved run without a person looking at each one. The bounds in R4.10 and R4.11 limit the damage; they do not prevent it. People named to set selecting attributes can still set them wrongly, and an attacker who can raise an event that triggers an automation can choose when it runs.
4. **Breadth of the requesting person's access.** A steered AI can read anything the requesting person can, unless the organisation restricts it under R1.8. For an administrator, that can be the whole estate.
5. **Disclosure to the disclosed third parties.** Everything the AI reads reaches the model provider. Search and lookup queries are free text that the AI writes, so anything it read can reach those services.
6. **Signalling through permitted choices.** The parameters of requests to managed systems are seen by whoever controls or watches those systems, such as an attacker on a host the AI is investigating. Which permitted search results the AI opens, when it makes requests, and whether a credentialed execution succeeds can also carry information. The bandwidth of these channels can be significant.
7. **Links a person opens.** A person can still open a link the AI wrote after seeing its full address, and the address can carry data.
8. **Software in the person's own session.** Software outside the conformance target that acts in a person's session can make the platform return everything that person can read and send it out through its own channels. For reads, that is equivalent to compromise of the person's read access, and its bandwidth is not limited by these rules. It can also show the person one proposal while they confirm another, unless the confirmation happens on a separate device that shows the proposal itself, and it may complete user verification itself where an authenticator unlocked in the same session reports it and the organisation has not restricted authenticators (R4.7).
9. **Code that runs on a managed system.** A part of a change that executes on a managed system has that system's network and data. R3.7 keeps credential values away from it and shows the approver where it runs; it does not stop that part from reaching whatever the system can reach.
10. **Credentials the platform cannot recognise.** Credential values in free text, images or formats the platform does not parse can reach the AI and the disclosed third parties, and a steered AI can use one it finds in a proposal.
11. **What has already been read.** Revoking access stops further requests within the documented delay, but cannot remove what the AI has already read.
12. **Load, cost, volume and availability.** Requests within the limits of R1.9 still consume the managed systems' capacity and the model's cost, and a flood of requests can make records harder to review. Because checks fail closed (R0.2), an outage of a service the platform depends on stops the AI, by design.
13. **Compromise outside the AI.** A compromised person together with their authenticator, a compromised platform, or a compromised third party is outside this specification.

## 7. Relation to change management

This section is informative. The rules fit the change enablement practice most IT organisations already follow:

- A **standard change** corresponds to a pre-approved class (R4.11). Creating or resetting credentials and authentication factors, often handled as standard changes or service requests, is excluded from classes (R1.6), because those requests are the usual target of social engineering.
- A **normal change** is a proposal and its approval (R4.1 to R4.9). A change authority, such as an advisory board, can be required as additional approvers (R4.13), who need not hold permission to make the change themselves. An approval recorded in an external change-management system can be a precondition (R4.4) and can be referenced in the decision record (R5.1), but the binding approval is given in the platform under R4.7.
- An **emergency change** follows the same path, faster. Change practices often allow emergency changes to be authorised after the fact; this specification does not allow that for changes the AI makes. Any setting that would let the AI change a system without Rule 4 takes the system out of conformance (R0.3), and people use their own access instead.
- A **change window** is a proposal with a stated start time (R4.2), approved no more than 30 days before it starts (R4.5). A **change freeze** is a period in which nothing runs (R4.15).
- A **backout plan** is a proposal that reverses a change. Where the targets are bound by fingerprint, approve it after the change has run, or bind it to the state the change is recorded as producing (R4.4).
- **Break-glass access** for people is outside this specification, which governs the AI. When the platform's own checks cannot run, the AI stops (R0.2), and people use their own access.

## 8. Out of scope

- The safety and quality of the model itself: hallucination rates, alignment, content filtering, training data and the model supply chain.
- Securing AI applications generally, such as chatbots and retrieval systems, beyond the AI operations system defined in section 1.
- The security of the platform itself, including its code, hosting, operators and the security of its connections to managed systems, and the platform vendor's own access to its customers' environments. They matter, and [CHECKLIST.md](CHECKLIST.md) asks about them, but they are vendor due diligence, not properties of the AI.
- People's own access to managed systems outside the platform.
- Certification. This specification defines requirements and self-assessment ([CONFORMANCE.md](CONFORMANCE.md)); it does not certify anyone. Organisations subject to requirements for tamper-evident logs should require R5.8, which is a recommendation here.

## 9. References

- S. Rose, O. Borchert, S. Mitchell, S. Connelly. *Zero Trust Architecture*. NIST Special Publication 800-207, August 2020. <https://doi.org/10.6028/NIST.SP.800-207>. Quoted from section 2.1 (tenets), section 3 (logical components), section 3.3.1 (trust algorithm) and section 5.7 (non-person entities).
- OWASP GenAI Security Project. *OWASP Top 10 for Large Language Model Applications*, version 2026, publication date 4 August 2026 (Zenodo record 22109015). <https://doi.org/10.5281/zenodo.22109015>
- OWASP GenAI Security Project, Agentic Security Initiative. *OWASP Top 10 for Agentic Applications 2026*. December 2025. <https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/>
- OWASP GenAI Security Project. *Agent Control Standard*, specification version 0.1.0, as published in repository release v0.1.3 (tagged 5 October 2026). <https://github.com/GenAI-Security-Project/agent-control-standard/tree/v0.1.3>
- Model Context Protocol. *Specification*, version 2026-07-28: Overview (Security and Trust & Safety); Authorization, Security Considerations; Server, Tools; Client, Elicitation. <https://modelcontextprotocol.io/specification/2026-07-28>
- Meta. *Agents Rule of Two: A Practical Approach to AI Agent Security*. 31 October 2025. <https://ai.meta.com/blog/practical-ai-agent-security/>
- Simon Willison. *The lethal trifecta for AI agents: private data, untrusted content, and external communication*. 16 June 2025. <https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/>
- Josh Woodruff (MassiveScale.AI). *The Agentic Trust Framework: Zero Trust Governance for AI Agents*. Cloud Security Alliance blog, 2 February 2026. <https://cloudsecurityalliance.org/blog/2026/02/02/the-agentic-trust-framework-zero-trust-governance-for-ai-agents>
- MassiveScale.AI. *Agentic Trust Framework*: specification version 0.9.1, conformance specification version 0.9.0 and maturity model, April 2026. <https://github.com/massivescale-ai/agentic-trust-framework/tree/v0.9.1>
- B. Campbell, J. Bradley, N. Sakimura, T. Lodderstedt. *OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens*. RFC 8705, February 2020.
- D. Fett, B. Campbell, J. Bradley, T. Lodderstedt, M. Jones, D. Waite. *OAuth 2.0 Demonstrating Proof of Possession (DPoP)*. RFC 9449, September 2023.
- S. Bradner. *Key words for use in RFCs to Indicate Requirement Levels*. RFC 2119, March 1997. B. Leiba. *Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words*. RFC 8174, May 2017.
