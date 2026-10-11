# Crosswalk: zero-trust AI operations and related work

How the requirements in [SPECIFICATION.md](SPECIFICATION.md) relate to the standards and guidance security teams already use. Use it to show an auditor where a control comes from, or to check whether a framework you already follow covers the same ground.

This document is informative. Quotations follow the conventions in section 2 of the specification: they are word for word from the edition named in the [references](SPECIFICATION.md#9-references), with typographic quotation marks and apostrophes normalised, and they remain under their own licences. Where a source uses its own numbering or names, they are kept. The labels mean:

- **(summary)**: the source item is paraphrased, not quoted.
- **Partly**: the requirements cover some of what the source item asks for; the text says what is covered and what is not.
- **Different approach**: the specification reaches the same aim by another means.
- **Out of scope**: the item concerns something section 8 of the specification excludes.
- **Not relied on**: the item is compatible, but no requirement depends on it (R0.1).
- **Departure**: the specification deliberately takes another position.
- **Residual risk**: the specification does not prevent what the item describes, and section 6 says so.
- **Not covered**: the item is in scope, but no requirement addresses it.

An item without a label is covered by the requirements named, or is broken down in the table that follows it.

## Contents

- [NIST SP 800-207, Zero Trust Architecture](#nist-sp-800-207-zero-trust-architecture)
- [OWASP Top 10 for LLM Applications 2026](#owasp-top-10-for-llm-applications-2026)
- [OWASP Top 10 for Agentic Applications 2026](#owasp-top-10-for-agentic-applications-2026)
- [Model Context Protocol specification 2026-07-28](#model-context-protocol-specification-2026-07-28)
- [Agents Rule of Two](#agents-rule-of-two)
- [The lethal trifecta](#the-lethal-trifecta)
- [Agentic Trust Framework](#agentic-trust-framework)
- [OWASP Agent Control Standard](#owasp-agent-control-standard)

## NIST SP 800-207, Zero Trust Architecture

Zero-trust AI operations treats the AI as a subject in the sense of NIST SP 800-207: a non-person entity that requests access to resources and is never trusted because of where it runs. NIST lets a subject's history inform dynamic policy; section 3.3.1 notes that "A contextual TA takes the subject or network agent's recent history into consideration when evaluating access requests." This specification never lets the AI's history widen what it may change.

### The tenets (section 2.1)

| Tenet | Where it applies |
| --- | --- |
| 1. "All data sources and computing services are considered resources." | Every managed system the AI can reach, and every resource in it, is a resource (section 3, *Managed system*, *Resource*). |
| 2. "All communication is secured regardless of network location. Network location alone does not imply trust." | **Partly.** Covered: Rule 3 gives the AI no network path of its own (R3.1, R3.2). Not covered: securing the platform's own connections to managed systems, which is platform security and out of scope (section 8). |
| 3. "Access to individual enterprise resources is granted on a per-session basis." … "Access should also be granted with the least privileges needed to complete the task." | **Partly.** Covered: R1.3, R1.13 and R2.4 limit permissions and identity to one bounded session, and R1.8 requires that the organisation can restrict the AI below the requesting person's permissions. Not covered: whether least privilege is applied is the organisation's choice. |
| 4. "Access to resources is determined by dynamic policy" … "Least privilege principles are applied to restrict both visibility and accessibility." | **Partly.** Covered: R1.1 authorises each request against the requesting person's current permissions, at the granularity of each system's resources. Not covered: least privilege below those permissions depends on the organisation using R1.8. |
| 5. "The enterprise monitors and measures the integrity and security posture of all owned and associated assets. No asset is inherently trusted." | **Partly.** Covered: the AI's runtime is never trusted for what it claims (R0.1, R1.2, R1.5). Not covered: monitoring the security posture of assets. |
| 6. "All resource authentication and authorization are dynamic and strictly enforced before access is allowed." | **Partly.** Covered: R1.1 and R1.2 for every request; R4.1 to R4.7 for changes, where the approval is the enforcement; R0.2, a check that cannot complete stops the request. Not covered: the authentication of people, which the specification trusts, and continuous re-evaluation; R1.1 allows permissions to be cached for up to an hour. |
| 7. "The enterprise collects as much information as possible about the current state of assets, network infrastructure and communications and uses it to improve its security posture." | **Partly.** Covered: Rule 5 records everything the AI is told, requests and decides (R5.1 to R5.3). Not covered: collecting information about assets and networks generally. |

### The logical components (section 3)

| NIST component | In an AI operations system |
| --- | --- |
| Subject | The AI, acting for the requesting person. |
| Policy engine (PE), which "makes and logs the decision (as approved, or denied)" | Authorisation of each request (R1.1), recorded under R5.2. For changes, this specification adds a person's approval (R4.6, R4.7), which sits outside NIST's automated policy engine, recorded under R5.1. |
| Policy administrator (PA), which would "generate any session-specific authentication and authentication token or credential" | The platform's credential handling (R2.2, R2.3). This specification goes further: the credential is used on the AI's behalf and never handed to it (R2.1). |
| Policy enforcement point (PEP) | The platform's mediated interface, the AI's only route to a managed system (R3.1). |

### Non-person entities (section 5.7)

Section 5.7 concerns non-person entities used to administer a zero trust architecture; it opens: "Artificial intelligence and other software-based agents are being deployed to manage security issues on enterprise networks." Its warnings carry over to the AI that operates infrastructure: "an attacker will be able to induce or coerce an NPE to perform some task that the attacker is not privileged to perform", and "an attacker could gain access to a software agent's credentials and impersonate the agent when performing tasks". Rules 1, 3 and 4 address the first: a coerced AI can read only what the requesting person can, cannot send it out except through channels people control, and cannot change anything except through a person's approval or within bounds people set. Rule 2 addresses the second: no credential the platform uses is placed in the AI's reach, and its session identity cannot be used outside its session (R2.4).

## OWASP Top 10 for LLM Applications 2026

The 2026 edition renumbered the list; Excessive Agency, LLM06 in the 2025 edition, is LLM03 in 2026. Its section "What's New in the 2026 Top 10" draws the line that this specification sits on: "The moment that model becomes an actor, with tools it can call, memory it carries between sessions, and consequences it sets in motion downstream, the risk moves to the OWASP Agentic Top 10." Both lists are therefore mapped.

### The ten entries

| Entry | Requirements |
| --- | --- |
| LLM01:2026 Prompt Injection | The threat model (section 4) and all five rules. Strategy by strategy below. |
| LLM02:2026 Sensitive Information Disclosure | **Partly.** Covered: R2.1, R2.5 and R2.6 keep credentials out of what the AI reads; R1.10 limits who can see information, including through training on it; R1.11 and R3.6 cover what third parties may do with it; Rule 3 limits the ways out. Not covered: memorisation by the model, which is out of scope; aggregation of individually permitted information; and disclosure to the disclosed third parties, which is residual risk 5. |
| LLM03:2026 Excessive Agency | Rules 1 and 4. Strategy by strategy below. |
| LLM04:2026 Supply Chain | **Partly.** Covered: R4.2 pins each artefact a change installs or executes by digest, and R4.12 stops tool servers from changing tool definitions without a person's approval. Not covered: the model supply chain, which is out of scope. |
| LLM05:2026 Data and Model Poisoning | **Partly.** Covered: poisoning of the AI's persistent memory is limited by R4.12, which keeps memory out of instruction stores and never shows it as evidence, and by R1.10. Not covered: training and model poisoning, which are out of scope. |
| LLM06:2026 Unbounded Consumption | **Not covered** for model usage. R1.9 recommends limiting the load the AI places on managed systems. |
| LLM07:2026 Misinformation | **Residual risk** 1. The entry warns that "The core risk is that the incorrect output is trusted and acted upon"; R4.9 shows the approver source records, not the AI's account, but a source record is not proof. |
| LLM08:2026 Hidden Context Exposure | **Partly.** Covered: R0.1, R2.1 and R2.5. The entry advises that "any contents of the context should not be considered a secret"; no credential the platform uses is placed there, and credentials in data the AI reads are replaced as R2.6 requires. Not covered: credentials it cannot recognise (residual risk 10). |
| LLM09:2026 Vector and Embedding Weaknesses | **Partly.** Covered: R1.10 and R1.11 state the outcome its first example, "Cross-Tenant Leakage via Shared Similarity Search", threatens: information is available only to people with access to it, through any store. Not covered: the inference from result counts, scores and timing that the example describes. |
| LLM10:2026 Improper Output Handling | **Partly.** Covered: R3.4 and R3.8 for interfaces and delivery, R4.9 and R5.5 for approval views, records and exports; scripts that would change a managed system are proposals (R4.1), and code the AI runs in its own environment has no network (R3.2). Not covered: the entry's other strategies, such as validating generated code before it is proposed. Strategy 9 below. |

### LLM01:2026 Prompt Injection

The entry states that "no reliable prevention mechanism exists today" and that "Defense is therefore architectural rather than interceptive." Zero-trust AI operations is an architecture of that kind.

| LLM01:2026 strategy | Requirements |
| --- | --- |
| 1. Constrain the model's role and capabilities in the system prompt **(summary)** | **Not relied on** (R0.1). The entry itself calls it "a partial control only". |
| 2. Define a strict output schema and validate every response in trusted application code **(summary)** | **Not relied on.** Compatible. |
| 3. Filter at every modality boundary **(summary)** | **Not relied on** (R0.1). |
| 4. "Hold credentials and state-change capability in application code, not the model, and grant least privilege per operation." | **Partly.** Covered: R2.1 and R2.2 for credentials; R4.1 for state change; R1.1 authorises every request, capped at the requesting person's permissions. Not covered: least privilege below those permissions depends on the organisation using R1.8. |
| 5. Strip invisible characters at every ingest and render boundary **(summary)** | **Partly.** Covered: R4.9 makes such characters visible in the approval view, where a hidden difference matters most. Not covered: stripping them at every other boundary. |
| 6. Pass external content through a structurally separate, provenance-labelled channel so the model can distinguish data from instructions **(summary)** | **Not relied on** (R0.1), because it is aimed at the model. This specification applies provenance where people decide: R4.9 marks source records and the AI's own text, and R4.12 marks memory with its origin. |
| 7. "Require explicit human confirmation before any privileged, irreversible, or externally visible action, surfacing the exact rendered action rather than a summary to the reviewer." | **Partly.** Covered: R4.1, R4.7 and R4.9 for changes; R3.3 for external communication. Not covered: reads, including privileged reads, communication to allow-listed destinations, and changes within automations and classes (R4.10, R4.11) run without confirmation of each action. |
| 8. "Budget agent capabilities with the Rule of Two as a floor" | **Partly.** Covered: the threat model (section 4) and Rule 3; the strategy's demand that "[A,B] or [A,C] configurations need an explicit residual-risk assessment" is met by section 6 and [CONFORMANCE.md](CONFORMANCE.md). Not covered: changes within automations and classes run without approval of each action, which departs from the strategy's "any [A,B,C] agent needs per-action human approval"; R4.10 and R4.11 bound them instead. |
| 9. "Treat agent memory writes as privileged operations." | **Different approach.** R4.12 marks memory with its origin, never shows it as evidence and keeps it out of instruction stores unless a person approves. It does not require approval before memory persists. |
| 10. Pin, sign and verify MCP servers and third-party tool packages, audit tool descriptions and monitor tool composition **(summary)** | **Partly.** Covered: R4.12 stops tool servers from changing tool definitions without a person's approval, R5.3 records the version of every tool definition the AI is given, and R1.5 relies on a tool server's classifications only if the organisation has designated it as trusted. Not covered: signing, auditing and monitoring. |
| 11. "Test against adaptive attackers who have read the deployed defense, and reject static-only attack-success claims." | **Partly.** Covered: the verification method (section 2) tests the system as if the attacker fully controls the model, which covers adaptive attacks on the model. Not covered: red-teaming the deployed defences. |

### LLM03:2026 Excessive Agency

The entry names three root causes: "excessive functionality", "excessive permissions" and "excessive autonomy".

| LLM03:2026 strategy | Requirements |
| --- | --- |
| 1. Minimize tools | **Partly.** Covered: R1.4 makes unoffered operations unavailable, and R1.8 lets the organisation restrict the rest. Not covered: minimising them is the organisation's choice. |
| 2. Minimize tool functionality | **Partly.** Covered: R1.4 and R1.5. Not covered: minimising functionality is the organisation's choice. |
| 3. Avoid open-ended tools | **Different approach.** Open-ended code execution is allowed, without network (R3.2), and anything that is not a read becomes a proposal (R1.5, R4.1). |
| 4. Minimize tool permissions | **Partly.** Covered: R1.1 and R1.3 cap the AI at the requesting person's permissions, and R1.8 lets the organisation restrict further. Not covered: applying the restriction is the organisation's choice. |
| 5. Execute tools in user's context | **Partly.** Covered: R1.1, R1.2. Not covered: "with the minimum privileges necessary" depends on R1.8. |
| 6. Require user approval | R4.1 to R4.9. |
| 7. Complete mediation | R0.1, R1.1, R1.2, R1.5. The graduated enforcement it describes corresponds to R4.11. |
| 8. Monitor tool use | **Partly.** Covered: Rule 5 records every request (R5.2). Not covered: monitoring the records, which is left to the organisation. |
| 9. Rate limiting | **Partly.** Covered: the rate limits that R4.10 and R4.11 require for automations and classes. Not covered: limits on sessions, which R1.9 only recommends. |

### LLM10:2026 Improper Output Handling

Strategy 9 reads: "In client renderers (chat UIs, IDEs, email clients, mobile apps), prevent model output from triggering automatic outbound requests to attacker-controlled endpoints. Disable auto-rendering of Markdown images, link previews, iframes, and similar elements by default. Where rendering is required, restrict fetches to an explicit allowlist of origins or proxy them through a server-side fetcher that strips data-bearing query parameters." R3.4 requires this of every interface that shows the AI's output and of every system the platform delivers output to, with exceptions only for the platform's own addresses and allow-listed destinations, and R3.8 requires that text from outside sources is never run as code. R4.9 and R5.5 apply the same care to approval views, records and exports.

## OWASP Top 10 for Agentic Applications 2026

The OWASP Top 10 for Agentic Applications (December 2025) covers agents in general. It refers to the 2025 numbering of the LLM list, so its "LLM06:2025 (Excessive Agency)" is LLM03:2026 above.

| Entry | Requirements |
| --- | --- |
| ASI01 Agent Goal Hijack | **Partly.** Covered: the threat model (section 4) assumes the goal can be hijacked at any moment, and the rules limit what a hijacked AI can do. Its mitigation of "requiring human approval for high-impact or goal-changing actions" is Rule 4, and its requirement that prompt changes "must go through configuration management and human approval" is R4.12. Not covered: intent validation, input sanitisation and behavioural baselines. |
| ASI02 Tool Misuse and Exploitation | **Partly.** Covered: its mitigation "Enforce outbound allowlists and deny all non-approved network destinations." corresponds to R3.2, R3.3 and R3.7 for code the platform runs; its dry-run diff before approval to R4.2 and R4.9; and "Bind keys to specific user sessions to prevent lateral abuse." to R2.4. Not covered: tool budgets for sessions, which R1.9 only recommends, and drift detection. |
| ASI03 Identity and Privilege Abuse | **Partly.** Covered: R1.1 to R1.3, R1.7, R2.4 and R4.6. Its mitigation "Mandate Per-Action Authorization" is R1.1, and its example "Permissions may be validated at the start of a workflow but change or expire before execution." is answered by authorising every request when it runs (R1.1) and checking the approver's permission again at execution (R4.6). Not covered: task-scoped permissions depend on the organisation using R1.8. **Departure:** the entry warns of an agent without "a distinct, governed identity of its own"; under Rule 1 the AI deliberately has no authority of its own, and acts only within the permissions of the people involved. |
| ASI04 Agentic Supply Chain Vulnerabilities | **Partly.** Covered: its mitigation "Pin prompts, tools, and configs by content hash and commit ID." targets the agent's own prompts and tools; this specification records the version of every instruction-store item given to the AI (R5.3), stops instruction stores and tool definitions changing without a person's approval (R4.12), and pins the artefacts of each change by digest (R4.2). Not covered: signed attestation of components; its supply chain kill switch corresponds only in part to R1.12. |
| ASI05 Unexpected Code Execution (RCE) | **Partly.** Covered: its mitigation "Run code in sandboxed containers with strict limits including network access" is R3.2 for code the AI runs, which allows no network at all except the platform's interface, and R3.7 and R4.1 for code the platform runs for a change. Not covered: parts of approved changes that run on a managed system have that system's network (residual risk 9), and its other mitigations, such as banning eval in production agents. |
| ASI06 Memory & Context Poisoning | **Partly.** Covered: its mitigation "Prevent automatic re-ingestion of an agent's own generated outputs into trusted memory" corresponds to R4.12, and its memory segmentation to R1.10 and R1.11. Not covered: scanning memory writes, trust scores and expiry of unverified memory. |
| ASI07 Insecure Inter-Agent Communication | **Different approach.** Sub-agents are part of the AI (section 3) and are confined by the same rules, so messages between them need no trust; agents outside the platform are external communication (R3.3). Authentication of messages between agents is not specified. |
| ASI08 Cascading Failures | **Partly.** Covered: R4.1 stops a fault from reaching a managed system without a person or the bounds of R4.10 and R4.11; R1.12 stops sessions, changes, automations and classes. Not covered: drift detection and replay testing. |
| ASI09 Human-Agent Trust Exploitation | **Partly.** The entry warns of humans "approving actions without independent validation". Covered: its mitigation 4 asks for a "plain-language risk summary (not model-generated rationales)", and R4.9 meets the "not model-generated" part by showing source records and marking the AI's text; its mitigation 7, "Separate preview from effect", is how proposals work (R4.1). Not covered: a risk summary, its mitigation 7's "risk badge with source provenance and expected side effects", and a way for users to flag suspicious behaviour. **Departure:** its mitigation 5, "Adaptive Trust Calibration", is not adopted for changes. |
| ASI10 Rogue Agents | **Partly.** Covered: its mitigations call for "kill-switches and credential revocation" (R1.12) and state, about signing and attestation keys, that "Keys must never be directly available to agents"; Rule 2 applies the same principle to the credentials the platform uses (R2.1, R2.5). Not covered: behavioural attestation and watchdog agents. |

## Model Context Protocol specification 2026-07-28

Many AI operations systems connect the AI to tools over the Model Context Protocol (MCP). Its security requirements and principles relate to these rules as follows.

| MCP 2026-07-28 | Requirements |
| --- | --- |
| Authorization, Security Considerations: "The MCP server MUST NOT pass through the token it received from the MCP client." | R2.1, R2.2 and R2.4: credentials for managed systems stay with the platform, and the AI's session identity is never forwarded to managed systems. |
| Authorization, Security Considerations: "MCP servers MUST validate that tokens presented to them were specifically issued for their use" | R2.4 requires the session identity to be accepted only by the platform, and goes further: it is bound to its session and person, and a copy cannot be used elsewhere. |
| Server, Tools: "clients MUST consider tool annotations to be untrusted unless they come from trusted servers." | R1.5: classifications a tool server supplies are relied on only where the organisation has designated the server as trusted. |
| Server, Tools, Security Considerations: servers "MUST" "Rate limit tool invocations". | **Partly.** Covered: rate limits for automations and classes (R4.10, R4.11). Not covered: R1.9 only recommends rate limits for sessions. |
| Overview, Security and Trust & Safety, Key Principles: "Hosts must obtain explicit user consent before invoking any tool"; and Server, Tools: "there SHOULD always be a human in the loop with the ability to deny tool invocations." | **Departure** from the principle, compatible with the SHOULD. Reads run without consent to each request; this specification relies instead on authorising every request (R1.1), on approval of every change (R4.1), and on people's ability to stop any session (R1.12). |
| Overview, Security and Trust & Safety, Key Principles: "Hosts must not transmit resource data elsewhere without user consent" | R3.3: external communication goes only to allow-listed destinations that people approved in advance, or is approved message by message. |
| Client, Elicitation: "the protocol itself does not mandate any specific user interaction model" | R4.7. An answer to an elicitation shows that a client responded, not that a person did, so an approval given through one still needs user verification bound to the proposal. |
| Client, Elicitation, Safe URL Handling: clients "MUST NOT automatically pre-fetch the URL or any of its metadata.", "MUST show the full URL to the user for examination before consent." and "SHOULD highlight the domain of the URL to mitigate subdomain spoofing." | **Partly.** Covered: R3.4 applies these rules to every address in the AI's output that is not allow-listed. Not covered: its recommended warnings for ambiguous or suspicious addresses, such as Punycode. |
| Client, Elicitation, in a section of non-normative examples: "sensitive credentials never pass through the LLM context, MCP client or any intermediate MCP servers" | R2.1 and R2.3. |

## Agents Rule of Two

Meta's Agents Rule of Two (31 October 2025) states that, until robustness research allows prompt injection to be reliably detected and refused, "agents must satisfy no more than two of the following three properties within a session":

- "[A] An agent can process untrustworthy inputs"
- "[B] An agent can have access to sensitive systems or private data"
- "[C] An agent can change state or communicate externally"

An AI operations system always has [A] and [B]. Outside automations and classes, and apart from queries to disclosed search and lookup services (residual risk 5), this specification holds the AI to those two in the way Meta describes for an [AB] agent, which "can only send new emails to trusted recipients or once a human has validated the contents of the draft message":

- **Communication** goes only to allow-listed destinations whose readers have access to what the session read, or is approved message by message (R3.3); public retrieval cannot be steered by what the AI read beyond its query and a choice among permitted results (R3.5); and nothing in the AI's output is fetched automatically, except the platform's own addresses and allow-listed destinations (R3.4).
- **Changes** run only once a person has approved the exact content (Rule 4).
- **Pre-approved automations and classes** (R4.10, R4.11) are the exception: they let changes run without a person reviewing each one. For agents that need all three properties, Meta says the agent "should not be permitted to operate autonomously and at a minimum requires supervision — via human-in-the-loop approval or another reliable means of validation". Automations and classes rely on the second option: deterministic rules that people define, bound and time-limit, consistent with Meta's view that "traditional software guardrails and human approvals continue to be the preferred method of satisfying the Agents Rule of Two in present use cases".

For public retrieval, R3.5's first design follows Meta's example of "Limiting web requests to URLs exclusively returned from trusted sources like not visiting URLs constructed by the agent". Its second design, a separate session with no access to managed systems, is an [AC] configuration in Meta's terms, like Meta's web-browsing research assistant, which it protects by "Running the browser in a restrictive sandbox without preloaded session data". Meta also cautions that designs following the rule "can still be prone to failure (e.g., a user blindly confirming a warning interstitial)"; section 6 of the specification lists the corresponding residual risks.

## The lethal trifecta

Simon Willison's lethal trifecta (16 June 2025) is "Access to your private data", "Exposure to untrusted content" and "The ability to externally communicate in a way that could be used to steal your data". OWASP LLM01:2026 cites it as restating "the same structural diagnosis as a pre-deployment check", adding that "removing any one leg removes them".

An AI operations system cannot remove the first two legs and still do its job, so Rule 3 limits the third: the AI's code has no network, including name resolution (R3.2); every other channel is removed, allow-listed or approved (R3.3); nothing in the AI's output is fetched automatically, except the platform's own addresses and allow-listed destinations, or run as code (R3.4, R3.8); and what it has read cannot determine a public retrieval beyond a query to a disclosed service and a choice among permitted results (R3.5). Willison notes that a tool "providing a link for a user to click" can also carry data out. Section 6 lists that path, the queries and the other channels that remain, as residual risks 2, 3, 5 to 9 and 11.

## Agentic Trust Framework

The Agentic Trust Framework (ATF), by Josh Woodruff of MassiveScale.AI, was introduced on the Cloud Security Alliance blog and is maintained as a public review draft: specification version 0.9.1 and conformance specification version 0.9.0. It applies zero trust to AI agents in general, through five core elements and a maturity model. The blog post describes the levels as Intern ("observe only"), Junior ("recommend with approval"), Senior ("act with notification") and Principal ("autonomous within domain"). Element names, requirement identifiers and requirement names below are those of ATF specification 0.9.1 and conformance specification 0.9.0.

| ATF element | Zero-trust AI operations |
| --- | --- |
| Identity Management | **Partly.** Covered: R2.4 binds the AI's session identity to its session and requesting person, and its sender-constrained option matches I-2, Credential Binding; no credential the platform uses is placed in the AI's reach (R2.1). I-3, Ownership Chain, is covered in part by the requesting person and R1.7, and I-4, Purpose Declaration, in part by the stated scope of unattended triggers (R1.7). Not covered: I-1 and I-5. |
| Behavioral Monitoring | **Partly.** Covered: R5.1 and R5.2 cover B-2, Action Attribution, and the substance of B-1, Structured Logging; R5.3 covers part of B-5, Explainability. Not covered: B-1's machine-parseable format for every record, and B-3 and B-4. |
| Data Governance | **Partly.** Covered: the tracking of derived information (section 3, R1.10), with R4.9 and R4.12, covers part of D-5, Data Lineage; R2.6 applies the idea of D-3, PII/PHI Protection, to credentials. Not covered: personal data under D-3, and D-1 and D-4. D-2, Injection Prevention, "Detection of prompt injection and adversarial inputs", is **not relied on** (R0.1). |
| Segmentation | **Partly.** Covered: S-2, Action Boundaries, corresponds to R1.4 and R1.5, and S-1, Resource Allowlist, in part to R1.1 and R1.8; R4.10 and R4.11 correspond to S-3, Rate Limiting, S-4, Transaction Limits, and S-5, Blast Radius Containment, for automations and classes. Not covered: rate limits for sessions, which R1.9 only recommends. |
| Incident Response | **Partly.** Covered: R1.12 corresponds to R-2, Kill Switch, and R-3, Session Revocation; R4.4's reversal proposals to part of R-4, State Rollback; and R0.2 with R1.12's suspension of automations and classes to part of R-5, Graceful Degradation. Not covered: R-2's bound of "Immediate manual termination capability (<1 second)", since R1.12 allows 60 seconds, and R-1. |

**The main differences in position.**

- **Autonomy.** ATF lets people promote an agent after gates that include its track record and "Explicit approval from authorized stakeholders"; at Senior level an agent may "Execute approved action types autonomously". Here, a record earns the AI nothing. Pre-approved classes (R4.11) resemble Senior's approved action types, but people define, bound and time-limit them, and the AI is never promoted into them.
- **Credentials.** ATF's Senior level allows an agent to "Access credentials within scope". Rule 2 never places a credential the platform uses within the AI's reach.
- **Detection.** ATF requires injection detection (D-2) at some levels. This specification does not rely on detection.

Conformance to this specification does not imply conformance to ATF at any level, or the reverse.

## OWASP Agent Control Standard

The OWASP Agent Control Standard (ACS), specification version 0.1.0 as published in repository release v0.1.3, describes itself as "a wire specification": "It lets a separate Guardian Agent inspect what an AI agent is about to do and permit, deny, or modify that action before it happens, over an authenticated channel, with an audit trail that reconstructs after the fact." It standardises how a policy decision is requested and returned; this specification says which outcomes the system must guarantee.

An ACS deployment can serve as the enforcement point for R1.4 and R1.5, and as the membership decision for R4.11, provided that:

- the hook is sent by the platform's mediated interface, not left to the AI's runtime. ACS states that "An agent that implements ACS sends a hook request before it acts", and a steered runtime might not;
- its decisions come from deterministic policy only (R0.1). ACS allows that "A chain config MAY delegate to a second, agent layer, an LLM", which no decision this specification depends on may use;
- it denies, rather than proceeds, when a decision cannot be made (R0.2). The reference implementation starts with "Failure posture: proceed", and the posture can be changed to deny;
- its `defer` disposition, which can "postpone the verdict when the Guardian cannot yet reach one", never lets an action proceed without a verdict (R0.2);
- its `ask` disposition, to "route to a human, agent, or service approver", is used for a change only when it routes to a person and the approval meets R4.6 and R4.7;
- its `modify` disposition is never applied to a proposal after approval (R4.3);
- its messages are authenticated. ACS-Core requires "a baseline HMAC-SHA256 signature on every envelope", and lists "cryptographic signing" among optional conformance profiles, while the repository's README, in its account of where the reference implementation stands, states that "The wire is not authenticated".
