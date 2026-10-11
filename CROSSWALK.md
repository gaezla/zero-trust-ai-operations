# Crosswalk: zero-trust AI operations and related work

How the requirements in [SPECIFICATION.md](SPECIFICATION.md) relate to the standards and guidance security teams already use. Use it to show an auditor where a control comes from, or to check whether a framework you already follow covers the same ground.

This document is informative. Quotations are word for word from the edition named in the [references](SPECIFICATION.md#9-references), with typographic quotation marks and apostrophes normalised. Where a source uses its own numbering, it is kept. Entries marked **(summary)** paraphrase a source rather than quote it. **Partly** means the requirements cover only some of what the source item asks for.

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
| 1. "All data sources and computing services are considered resources." | Every managed system the AI can reach is a resource (section 3, *Managed system*). |
| 2. "All communication is secured regardless of network location. Network location alone does not imply trust." | **Partly.** Rule 3 gives the AI no network path of its own (R3.1, R3.2). Securing the platform's own connections to managed systems is platform security, which is out of scope (section 8). |
| 3. "Access to individual enterprise resources is granted on a per-session basis." … "Access should also be granted with the least privileges needed to complete the task." | **Partly.** R1.3, R1.13 and R2.4 limit access and identity to one bounded session. Restricting the AI below the requesting person's permissions is only a SHOULD (R1.8). |
| 4. "Access to resources is determined by dynamic policy" … "Least privilege principles are applied to restrict both visibility and accessibility." | **Partly.** R1.1 authorises each request against the requesting person's permissions at that time. Least privilege below those permissions is R1.8, a SHOULD. |
| 5. "The enterprise monitors and measures the integrity and security posture of all owned and associated assets. No asset is inherently trusted." | **Partly.** The AI's runtime is never trusted for what it claims (R0.1, R1.2, R1.5). Monitoring the security posture of assets is outside this specification. |
| 6. "All resource authentication and authorization are dynamic and strictly enforced before access is allowed." | R1.1 and R1.2 for every request; R4.1 to R4.7 for changes, where the approval is the enforcement; R0.2: a check that cannot complete stops the request. |
| 7. "The enterprise collects as much information as possible about the current state of assets, network infrastructure and communications and uses it to improve its security posture." | Rule 5: decision records (R5.1), records of every request (R5.2) and of every instruction and output (R5.3). |

### The logical components (section 3)

| NIST component | In an AI operations system |
| --- | --- |
| Subject | The AI, acting for the requesting person. |
| Policy engine (PE), which "makes and logs the decision (as approved, or denied)" | Authorisation of each request (R1.1), recorded under R5.2. For changes, this specification adds a person's approval (R4.6, R4.7), which sits outside NIST's automated policy engine, recorded under R5.1. |
| Policy administrator (PA), which would "generate any session-specific authentication and authentication token or credential" | The platform's credential handling (R2.2, R2.3). This specification goes further: the credential is used on the AI's behalf and never handed to it (R2.1). |
| Policy enforcement point (PEP) | The platform's mediated interface, the AI's only route to a managed system (R3.1). |

### Non-person entities (section 5.7)

Section 5.7 concerns non-person entities used to administer a zero trust architecture. Its warnings carry over to the AI that operates infrastructure: "an attacker will be able to induce or coerce an NPE to perform some task that the attacker is not privileged to perform", and "an attacker could gain access to a software agent's credentials and impersonate the agent when performing tasks". Rule 4 addresses the first: a coerced AI can propose, but cannot change anything except through a person's approval or within bounds people set. Rule 2 addresses the second: there is no credential in the AI's reach to take, and its session identity cannot be used outside its session (R2.4).

## OWASP Top 10 for LLM Applications 2026

The 2026 edition renumbered the list; Excessive Agency, LLM06 in the 2025 edition, is LLM03 in 2026. Its section "What's New in the 2026 Top 10" draws the line that this specification sits on: "The moment that model becomes an actor, with tools it can call, memory it carries between sessions, and consequences it sets in motion downstream, the risk moves to the OWASP Agentic Top 10." Both lists are therefore mapped.

### The ten entries

| Entry | Requirements |
| --- | --- |
| LLM01:2026 Prompt Injection | The threat model (section 4) and all five rules. Strategy by strategy below. |
| LLM02:2026 Sensitive Information Disclosure | **Partly.** R2.1, R2.5 and R2.6 keep credentials out of what the AI reads; R1.10 limits who can see what it reads; Rule 3 limits the ways out. Disclosure through training, pipelines and observability tools is out of scope, and disclosure to the disclosed third parties is residual risk 5. |
| LLM03:2026 Excessive Agency | Rules 1 and 4. Strategy by strategy below. |
| LLM04:2026 Supply Chain | **Partly.** R4.2 pins the dependencies of each change, and R4.12 stops tool servers from changing tool definitions without a person's approval. The model supply chain is out of scope. |
| LLM05:2026 Data and Model Poisoning | **Partly.** Training and model poisoning are out of scope. Poisoning of the AI's persistent memory is limited by R4.12, which keeps memory out of instruction stores, and by R1.10. |
| LLM06:2026 Unbounded Consumption | Not covered for model usage. R1.9 recommends limiting the load the AI places on managed systems. |
| LLM07:2026 Misinformation | Residual risk 1. The entry warns that "The core risk is that the incorrect output is trusted and acted upon"; R4.9 shows the approver source records, not the AI's account, but a source record is not proof. |
| LLM08:2026 Hidden Context Exposure | R2.1 and R2.5. The entry advises that "any contents of the context should not be considered a secret"; under Rule 2 no credential is ever there. |
| LLM09:2026 Vector and Embedding Weaknesses | R1.10 and R1.11 address its first example, "Cross-Tenant Leakage via Shared Similarity Search": what one organisation's sessions read is isolated from others, and within an organisation it reaches only people with access to its sources, unless deliberately shared. |
| LLM10:2026 Improper Output Handling | **Partly.** R3.4, R4.9 and R5.5. Scripts the AI generates are proposals that a person approves (R4.1). Strategy 9 below. |

### LLM01:2026 Prompt Injection

The entry states that "no reliable prevention mechanism exists today" and that "Defense is therefore architectural rather than interceptive." Zero-trust AI operations is an architecture of that kind.

| LLM01:2026 strategy | Requirements |
| --- | --- |
| 1. Constrain the model's role and capabilities in the system prompt **(summary)** | Not relied on (R0.1). The entry itself calls it "a partial control only". |
| 2. Define a strict output schema and validate every response in trusted application code **(summary)** | Complementary; not relied on. |
| 3. Filter at every modality boundary **(summary)** | Not relied on (R0.1). |
| 4. "Hold credentials and state-change capability in application code, not the model, and grant least privilege per operation." | R2.1 and R2.2 for credentials; R4.1 for state change. **Partly** for least privilege: R1.1 authorises every request, capped at the requesting person's permissions, and narrower privilege is R1.8, a SHOULD. |
| 5. Strip invisible characters at every ingest and render boundary **(summary)** | **Partly.** R4.9 makes such characters visible in the approval view, where a hidden difference matters most. Stripping them at every boundary is not required. |
| 6. Pass external content through a structurally separate, provenance-labelled channel so the model can distinguish data from instructions **(summary)** | Not relied on (R0.1), because it is aimed at the model. This specification applies provenance where people decide: R4.9 marks source records and the AI's own text, and R4.12 marks memory with its origin. |
| 7. "Require explicit human confirmation before any privileged, irreversible, or externally visible action, surfacing the exact rendered action rather than a summary to the reviewer." | R4.1, R4.7 and R4.9; R3.3 for externally visible actions. |
| 8. "Budget agent capabilities with the Rule of Two as a floor" | **Partly.** The threat model (section 4) and Rule 3. The strategy adds that "[A,B] or [A,C] configurations need an explicit residual-risk assessment", which [CONFORMANCE.md](CONFORMANCE.md) requires through section 6. Changes within pre-approved classes (R4.11) run without approval of each action, which departs from the strategy's "any [A,B,C] agent needs per-action human approval"; R4.11 bounds them instead. |
| 9. "Treat agent memory writes as privileged operations." | **Partly**, by a different route: R4.12 marks memory with its origin and keeps it out of instruction stores unless a person approves. It does not require approval before memory persists. |
| 10. Pin, sign and verify MCP servers and third-party tool packages, audit tool descriptions and monitor tool composition **(summary)** | **Partly.** R4.12 stops tool servers from changing tool definitions without a person's approval. Signing, auditing and monitoring are not required. |
| 11. "Test against adaptive attackers who have read the deployed defense, and reject static-only attack-success claims." | **Partly.** The verification method (section 2) tests the system as if the attacker fully controls the model, which covers adaptive attacks on the model. Red-teaming the deployed defences is not required. |

### LLM03:2026 Excessive Agency

The entry names three root causes: "excessive functionality", "excessive permissions" and "excessive autonomy".

| LLM03:2026 strategy | Requirements |
| --- | --- |
| 1. Minimize tools | R1.4; R1.8 (SHOULD). |
| 2. Minimize tool functionality | R1.4, R1.5. |
| 3. Avoid open-ended tools | A different approach. Open-ended code execution is allowed, without network (R3.2), and anything that is not a read becomes a proposal (R1.5, R4.1). |
| 4. Minimize tool permissions | **Partly.** R1.1 and R1.3 cap the AI at the requesting person's permissions; less than that is R1.8, a SHOULD. |
| 5. Execute tools in user's context | R1.1, R1.2. **Partly** for "with the minimum privileges necessary" (R1.8). |
| 6. Require user approval | R4.1 to R4.9. |
| 7. Complete mediation | R0.1, R1.1, R1.2, R1.5. The graduated enforcement it describes corresponds to R4.11. |
| 8. Monitor tool use | **Partly.** Rule 5 records every request (R5.2); monitoring the records is left to the organisation. |
| 9. Rate limiting | **Partly.** R1.9, a SHOULD. |

### LLM10:2026 Improper Output Handling

Strategy 9 reads: "Disable auto-rendering of Markdown images, link previews, iframes, and similar elements by default." R3.4 requires this of every interface that shows the AI's output and of every system the platform delivers output to. R4.9 and R5.5 apply the same care to approval views, records and exports.

## OWASP Top 10 for Agentic Applications 2026

The OWASP Top 10 for Agentic Applications (December 2025) covers agents in general. It refers to the 2025 numbering of the LLM list, so its "LLM06:2025 (Excessive Agency)" is LLM03:2026 above.

| Entry | Requirements |
| --- | --- |
| ASI01 Agent Goal Hijack | The threat model (section 4) assumes the goal can be hijacked at any moment, and the rules limit what a hijacked AI can do. Its mitigation of "requiring human approval for high-impact or goal-changing actions" is Rule 4, applied to every change outside the automations and classes people define. |
| ASI02 Tool Misuse and Exploitation | Rules 1 and 3. Its mitigation "Enforce outbound allowlists and deny all non-approved network destinations." corresponds to R3.2, R3.3 and R3.7. |
| ASI03 Identity and Privilege Abuse | R1.1 to R1.3, R1.7, R2.4 and R4.6. Its mitigation "Mandate Per-Action Authorization" is R1.1. Its example "Permissions may be validated at the start of a workflow but change or expire before execution." is answered by authorising every request when it runs (R1.1) and checking the approver's permission again at execution (R4.6). **Partly** for task-scoped permissions, which are R1.8, a SHOULD. |
| ASI04 Agentic Supply Chain Vulnerabilities | **Partly.** Its mitigation "Pin prompts, tools, and configs by content hash and commit ID." targets the agent's own prompts and tools; this specification pins the dependencies of each change (R4.2) and stops tool definitions changing without a person's approval (R4.12). Its supply chain kill switch corresponds in part to R1.12. |
| ASI05 Unexpected Code Execution (RCE) | R3.2, R3.7 and R4.1. Its mitigation "Run code in sandboxed containers with strict limits including network access" is R3.2, which allows no network at all except the platform's interface. |
| ASI06 Memory & Context Poisoning | **Partly.** Its mitigation "Prevent automatic re-ingestion of an agent's own generated outputs into trusted memory" corresponds to R4.12, and its memory segmentation to R1.10 and R1.11. |
| ASI07 Insecure Inter-Agent Communication | Sub-agents are part of the AI (section 3) and are confined by the same rules. Agents outside the platform are external communication (R3.3). Authentication of messages between agents is not specified. |
| ASI08 Cascading Failures | R4.1 stops a fault from reaching a managed system without a person or the bounds of R4.10 and R4.11; R1.9 and R1.12 limit and stop sessions. |
| ASI09 Human-Agent Trust Exploitation | **Partly.** The entry warns of humans "approving actions without independent validation". Its mitigation 4 asks for a "plain-language risk summary (not model-generated rationales)"; R4.9 meets the "not model-generated" part, by showing source records and marking the AI's text, but does not require a risk summary. Its mitigation 7, "Separate preview from effect", is how proposals work (R4.1). This specification does not adopt its mitigation 5, "Adaptive Trust Calibration", for changes. |
| ASI10 Rogue Agents | R1.12 and Rule 2. Its mitigations call for "kill-switches and credential revocation" (R1.12) and state, about signing and attestation keys, that "Keys must never be directly available to agents"; Rule 2 applies the same principle to every credential (R2.1, R2.5). |

## Model Context Protocol specification 2026-07-28

Many AI operations systems connect the AI to tools over the Model Context Protocol (MCP). Its security requirements and principles relate to these rules as follows.

| MCP 2026-07-28 | Requirements |
| --- | --- |
| Authorization, Security Considerations: "The MCP server MUST NOT pass through the token it received from the MCP client." | R2.1, R2.2: credentials for managed systems stay with the platform. |
| Authorization, Security Considerations: "MCP servers MUST validate that tokens presented to them were specifically issued for their use" | **Partly** R2.4, which also binds the AI's session identity to its session and person, and stops a copy from being used elsewhere. |
| Overview: "descriptions of tool behavior such as annotations should be considered untrusted, unless obtained from a trusted server." | R1.5: classifications a tool server supplies are relied on only where the organisation trusts the server. |
| Overview: "Hosts must obtain explicit user consent before invoking any tool" | A departure. Reads run without consent to each request; this specification relies instead on authorising every request (R1.1) and on approval of every change (R4.1). |
| Elicitation: "the protocol itself does not mandate any specific user interaction model" | R4.7. An answer to an elicitation shows that a client responded, not that a person did, so an approval given through one still needs user verification bound to the proposal. |
| Elicitation, URL mode: clients "MUST NOT automatically pre-fetch the URL or any of its metadata.", "MUST show the full URL to the user for examination before consent." and "SHOULD highlight the domain of the URL to mitigate subdomain spoofing." | R3.4 applies the same rules to every address in the AI's output. |
| Elicitation, URL mode, in a section of non-normative examples: "sensitive credentials never pass through the LLM context, MCP client or any intermediate MCP servers" | R2.1 and R2.3. |

## Agents Rule of Two

Meta's Agents Rule of Two (31 October 2025) states that, until robustness research allows prompt injection to be reliably detected and refused, "agents must satisfy no more than two of the following three properties within a session":

- "[A] An agent can process untrustworthy inputs"
- "[B] An agent can have access to sensitive systems or private data"
- "[C] An agent can change state or communicate externally"

An AI operations system always has [A] and [B]. This specification holds the AI itself to those two, in the way Meta describes for an [AB] agent, which "can only send new emails to trusted recipients or once a human has validated the contents of the draft message":

- **Communication** goes only to allow-listed destinations whose readers may see what the AI read, or is approved message by message (R3.3), and nothing in the AI's output is fetched automatically (R3.4).
- **Changes** run only once a person has approved the exact content (Rule 4).
- **Pre-approved automations and classes** (R4.10, R4.11) are the exception: they let changes run without a person reviewing each one. For agents that need all three properties, Meta says the agent "should not be permitted to operate autonomously and at a minimum requires supervision — via human-in-the-loop approval or another reliable means of validation". Automations and classes rely on the second option: deterministic rules that people define, bound and time-limit.

The designs that R3.5 allows for public retrieval follow Meta's own examples: "Limiting web requests to URLs exclusively returned from trusted sources like not visiting URLs constructed by the agent", and work done after "starting a new session (i.e., with a fresh context window)". Meta also cautions that designs following the rule "can still be prone to failure (e.g., a user blindly confirming a warning interstitial)"; section 6 of the specification lists the corresponding residual risks.

## The lethal trifecta

Simon Willison's lethal trifecta (16 June 2025) is "Access to your private data", "Exposure to untrusted content" and "The ability to externally communicate in a way that could be used to steal your data". OWASP LLM01:2026 cites it as restating "the same structural diagnosis as a pre-deployment check", adding that "removing any one leg removes them".

An AI operations system cannot remove the first two legs and still do its job, so Rule 3 limits the third: the AI's code has no network, including name resolution (R3.2); every other channel is removed, allow-listed or approved (R3.3); nothing the AI writes is fetched automatically (R3.4); and what it has read cannot determine a public retrieval beyond a search query and a choice among permitted results (R3.5). Willison notes that a tool "providing a link for a user to click" can also carry data out. Section 6 lists that path, the search query and the other channels that remain as residual risks 5 to 7.

## Agentic Trust Framework

The Agentic Trust Framework (ATF), by Josh Woodruff of MassiveScale.AI, was introduced on the Cloud Security Alliance blog and is maintained as a public review draft: specification version 0.9.1 and conformance specification version 0.9.0. It applies zero trust to AI agents in general, through five elements and a maturity model. The blog post describes the levels as Intern ("observe only"), Junior ("recommend with approval"), Senior ("act with notification") and Principal ("autonomous within domain"). Requirement identifiers below are ATF's own.

| ATF element | Zero-trust AI operations |
| --- | --- |
| Identity | **Partly.** R2.4 binds the AI's session identity to its session and requesting person, compatible with I-2, Credential Binding; in addition, no credential for a managed system is ever in the AI's reach (R2.1). I-1, I-3, I-4 and I-5 are not covered. |
| Behavior | **Partly.** R5.1 and R5.2 cover B-2, Action Attribution, and the substance of B-1, Structured Logging, without requiring a machine-parseable format. B-3 to B-5 are not covered. |
| Data Governance | **Partly.** R2.6 corresponds to D-3, "Automated detection and masking of sensitive data", for credentials only, not personal data. D-2, "Detection of prompt injection and adversarial inputs", is not relied on (R0.1). D-1, D-4 and D-5 are not covered. |
| Segmentation | **Partly.** S-2, Action Boundaries, corresponds to R1.4 and R1.5, and S-1, Resource Allowlist, in part to R1.1 and R1.8. R4.11 corresponds to S-4 and S-5 for classes. S-3, Rate Limiting, corresponds to R1.9, a SHOULD. |
| Incident Response | **Partly.** R1.12 covers R-2, Kill Switch, and R-3, Session Revocation, within a documented time rather than ATF's "Immediate manual termination capability (<1 second)". R-1, R-4 and R-5 are not covered. |

**The main differences in position.**

- **Autonomy.** ATF lets people promote an agent after gates that include its track record and "Explicit approval from authorized stakeholders"; at Senior level an agent may "Execute approved action types autonomously". Here, a record earns the AI nothing. Pre-approved classes (R4.11) resemble Senior's approved action types, but people define, bound and time-limit them, and the AI is never promoted into them.
- **Credentials.** ATF's Senior level allows an agent to "Access credentials within scope". Rule 2 never gives the AI a credential.
- **Detection.** ATF requires injection detection (D-2) at some levels. This specification does not rely on detection.

Conformance to this specification does not imply conformance to ATF at any level, or the reverse.

## OWASP Agent Control Standard

The OWASP Agent Control Standard (ACS), version 0.1.3, describes itself as "a wire specification": "It lets a separate Guardian Agent inspect what an AI agent is about to do and permit, deny, or modify that action before it happens". It standardises how a policy decision is requested and returned; this specification says which outcomes the system must guarantee.

An ACS deployment can serve as the enforcement point for R1.4 and R1.5, and as the membership decision for R4.11, provided that:

- the hook is sent by the platform's mediated interface, not left to the AI's runtime. ACS states that "An agent that implements ACS sends a hook request before it acts", and a steered runtime might not;
- its decisions come from deterministic policy, not from a model's judgement (R0.1);
- it denies, rather than proceeds, when a decision cannot be made (R0.2). The reference implementation starts with "Failure posture: proceed", and the posture can be changed to deny;
- its `ask` disposition, to "route to a human, agent, or service approver", is used for a change only when it routes to a person and the approval meets R4.6 and R4.7;
- its `modify` disposition is never applied to a proposal after approval (R4.3);
- its messages are authenticated. ACS-Core requires a signature on every envelope, while the reference implementation's own documentation states that "The wire is not authenticated".
