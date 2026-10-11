# Crosswalk: zero-trust AI operations and related work

How the requirements in [SPECIFICATION.md](SPECIFICATION.md) relate to the standards and guidance security teams already use. Use it to show an auditor where a control comes from, or to check whether a framework you already follow covers the same ground.

Every quotation below is taken word for word from the edition named in the [references](SPECIFICATION.md#8-references). Where a source uses another numbering scheme, its own numbering is kept.

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

Zero-trust AI operations treats the AI as a subject in the sense of NIST SP 800-207: a non-person entity that requests access to resources, and that is not trusted because of where it runs or what it has done before.

### The tenets (section 2.1)

| Tenet | Where it applies |
| --- | --- |
| 1. "All data sources and computing services are considered resources." | Every managed system the AI can reach is a resource (section 3, *Managed system*). |
| 2. "All communication is secured regardless of network location. Network location alone does not imply trust." | Rule 3: the AI gets no network path of its own (R3.1, R3.2). |
| 3. "Access to individual enterprise resources is granted on a per-session basis." … "Access should also be granted with the least privileges needed to complete the task." | R1.3 and R2.4: access and identity last one session, within the requesting person's permissions. R4.5: an approval is valid for a limited time. |
| 4. "Access to resources is determined by dynamic policy" | R1.1: authorisation against the permissions the requesting person holds at the time of each operation. |
| 5. "No asset is inherently trusted." | The AI's runtime is never trusted for what it claims (R1.2, R1.5, R0.1), and section 4 assumes it can be steered at any moment. |
| 6. "All resource authentication and authorization are dynamic and strictly enforced before access is allowed." | R1.1 and R1.2 for reads; R4.1 to R4.6 for changes, where the approval is the enforcement; R0.2: a check that cannot complete stops the operation. |
| 7. "The enterprise collects as much information as possible about the current state of assets, network infrastructure and communications and uses it to improve its security posture." | Rule 5: decision records (R5.1) and records of every operation (R5.2). |

### The logical components (section 3)

| NIST component | In an AI operations system |
| --- | --- |
| Subject | The AI, acting for the requesting person. |
| Policy engine (PE), which "makes and logs the decision (as approved, or denied)" | Authorisation of each operation (R1.1), and for changes, the person's approval (R4.1, R4.6), recorded in the decision record (R5.1). |
| Policy administrator (PA), which would "generate any session-specific authentication and authentication token or credential" | The platform's credential handling (R2.2, R2.3). Zero-trust AI operations goes further: the credential is used on the AI's behalf and never handed to it (R2.1). |
| Policy enforcement point (PEP) | The platform's mediated interface, the only path from the AI to a managed system (R3.1). |

### Non-person entities (section 5.7)

Section 5.7 warns that "an attacker will be able to induce or coerce an NPE to perform some task that the attacker is not privileged to perform", and that "an attacker could gain access to a software agent’s credentials and impersonate the agent when performing tasks". Rule 4 addresses the first risk: a coerced AI can propose, but cannot change anything without a person. Rule 2 addresses the second: there is no credential in the AI's reach to take, and its session identity cannot be used outside its session (R2.4).

## OWASP Top 10 for LLM Applications 2026

The 2026 edition renumbered the list; Excessive Agency, LLM06 in the 2025 edition, is LLM03 in 2026. Its preface draws the line this specification sits on: "The moment that model becomes an actor, with tools it can call, memory it carries between sessions, and consequences it sets in motion downstream, the risk moves to the OWASP Agentic Top 10." Both lists are therefore mapped.

### The ten entries

| Entry | Requirements |
| --- | --- |
| LLM01:2026 Prompt Injection | The threat model (section 4) and all five rules. Strategy by strategy below. |
| LLM02:2026 Sensitive Information Disclosure | R2.1, R2.5 and R2.6 keep credentials out of what the AI reads; R1.9 keeps what it reads with the people entitled to it; Rule 3 removes the ways out. |
| LLM03:2026 Excessive Agency | Rules 1 and 4. Strategy by strategy below. |
| LLM04:2026 Supply Chain | Partly. R4.2 pins every dependency a change fetches; R4.10 makes tool definitions changes. Model supply chain is out of scope (section 7). |
| LLM05:2026 Data and Model Poisoning | Out of scope: model training and data (section 7). |
| LLM06:2026 Unbounded Consumption | Not covered for model usage. R1.8 limits load on the managed systems, not on the model. |
| LLM07:2026 Misinformation | Section 6, residual risk 1. R4.7 shows the approver platform records instead of the AI's account; the entry's own warning is that "The core risk is that the incorrect output is trusted and acted upon." |
| LLM08:2026 Hidden Context Exposure | R2.1 and R2.5. The entry advises that "any contents of the context should not be considered a secret"; under Rule 2 no credential is ever there. |
| LLM09:2026 Vector and Embedding Weaknesses | R1.9 covers its first example, "Cross-Tenant Leakage via Shared Similarity Search": what one session or organisation reads never becomes available to another. |
| LLM10:2026 Improper Output Handling | R3.4 and R5.4. Strategy 9 below. |

### LLM01:2026 Prompt Injection

The entry states that "no reliable prevention mechanism exists today" and that "Defense is therefore architectural rather than interceptive." Zero-trust AI operations is an architecture of that kind.

| LLM01:2026 strategy | Requirements |
| --- | --- |
| 1. Constrain the model's role and capabilities in the system prompt | Not relied on (R0.1). The entry itself calls it "a partial control only". |
| 2. Define a strict output schema and validate every response | Complementary. Not relied on. |
| 3. Filter at every modality boundary | Complementary. Not relied on (R0.1). |
| 4. "Hold credentials and state-change capability in application code, not the model, and grant least privilege per operation." | R2.1, R2.2 (credentials); R4.1 (state change); R1.1 (least privilege per operation). |
| 5. Strip invisible characters at every ingest and render boundary | R4.7 makes invisible and look-alike characters visible to the approver, where a hidden difference would matter most. |
| 6. Pass external content through a provenance-labelled channel | R4.7 (evidence shown as platform records with their sources) and R4.10 (memory labelled with its origin). |
| 7. "Require explicit human confirmation before any privileged, irreversible, or externally visible action, surfacing the exact rendered action rather than a summary to the reviewer." | R4.1, R4.6, R4.7; R3.3 for externally visible actions. |
| 8. "Budget agent capabilities with the Rule of Two as a floor" | The threat model (section 4) and Rule 3. The strategy adds that "[A,B] or [A,C] configurations need an explicit residual-risk assessment"; section 6 requires one in every conformance statement. |
| 9. "Treat agent memory writes as privileged operations." | R4.10. |
| 10. Pin, sign and verify every MCP server and third-party tool package | Partly: R4.10 makes tool definitions changes, and R4.2 pins what a change fetches. |
| 11. "Test against adaptive attackers who have read the deployed defense, and reject static-only attack-success claims." | Section 2: verification tests the platform directly and repeats each test through the AI with adversarial instructions, and a model's refusal never counts as a pass. |

### LLM03:2026 Excessive Agency

The entry names three root causes: "excessive functionality", "excessive permissions" and "excessive autonomy".

| LLM03:2026 strategy | Requirements |
| --- | --- |
| 1. Minimize tools | R1.4, R1.6 |
| 2. Minimize tool functionality | R1.4 |
| 3. Avoid open-ended tools | R3.2 (code runs without network), R3.3, R3.5 |
| 4. Minimize tool permissions | R1.1, R1.3 |
| 5. Execute tools in user's context | R1.1, R1.2 |
| 6. Require user approval | R4.1 to R4.7 |
| 7. Complete mediation | R1.1, R1.2, R1.5, R0.1. The graduated enforcement it describes corresponds to R4.9. |
| 8. Monitor tool use | Rule 5, in particular R5.2 |
| 9. Rate limiting | R1.8 |

### LLM10:2026 Improper Output Handling

Strategy 9 reads: "Disable auto-rendering of Markdown images, link previews, iframes, and similar elements by default." R3.4 requires it of every interface that shows the AI's output, and of every system the platform delivers output to. R5.4 applies the same care to records and exports.

## OWASP Top 10 for Agentic Applications 2026

The OWASP Top 10 for Agentic Applications (December 2025) covers agents in general. It refers to the 2025 numbering of the LLM list, so its "LLM06:2025 (Excessive Agency)" is LLM03:2026 above.

| Entry | Requirements |
| --- | --- |
| ASI01 Agent Goal Hijack | The threat model (section 4) assumes the goal can be hijacked at any moment, and the rules limit what a hijacked AI can do. Its mitigation of "requiring human approval for high-impact or goal-changing actions" is Rule 4, applied here to every change. |
| ASI02 Tool Misuse and Exploitation | Rule 1 and Rule 3. Its mitigation "Enforce outbound allowlists and deny all non-approved network destinations." is R3.2 and R3.3. |
| ASI03 Identity and Privilege Abuse | R1.1 to R1.3, R1.7 and R2.4. Its mitigation "Mandate Per-Action Authorization" is R1.1. |
| ASI04 Agentic Supply Chain Vulnerabilities | R4.2 and R4.10. Its mitigation "Pin prompts, tools, and configs by content hash and commit ID." matches R4.2 for what a change fetches; its supply chain kill switch corresponds to R1.10 for sessions. |
| ASI05 Unexpected Code Execution (RCE) | R3.2 and R4.1. Its mitigation "Run code in sandboxed containers with strict limits including network access" is R3.2, which allows no network at all except the platform's interface. |
| ASI06 Memory & Context Poisoning | R4.10 and R1.9. Its mitigation "Prevent automatic re-ingestion of an agent’s own generated outputs into trusted memory" is R4.10, and its memory segmentation is R1.9. |
| ASI07 Insecure Inter-Agent Communication | Sub-agents are part of the AI (section 3) and are confined by the same rules. Agents outside the platform are external communication (R3.3). Message authentication between agents is not specified. |
| ASI08 Cascading Failures | R4.1 stops a fault from reaching a managed system without a person; R1.8 and R1.10 limit and stop sessions. |
| ASI09 Human-Agent Trust Exploitation | R4.6 and R4.7. The entry warns of humans "approving actions without independent validation", and advises a "plain-language risk summary (not model-generated rationales)"; R4.7 shows the approver platform records and marks the AI's own text. |
| ASI10 Rogue Agents | R1.10 and Rule 2. Its mitigations call for "kill-switches and credential revocation" (R1.10) and state that "Keys must never be directly available to agents" (R2.1, R2.5). |

## Model Context Protocol specification 2026-07-28

Many AI operations systems connect the AI to tools over the Model Context Protocol (MCP). The specification's security requirements line up with these rules.

| MCP 2026-07-28 | Requirements |
| --- | --- |
| Authorization, security considerations: "The MCP server MUST NOT pass through the token it received from the MCP client." | R2.1, R2.2: credentials for managed systems stay with the platform. |
| Authorization, security considerations: "MCP servers MUST validate that tokens presented to them were specifically issued for their use" | R2.4: the AI's session identity is bound to its session and person. |
| Elicitation: "the protocol itself does not mandate any specific user interaction model" | R4.6. An answer to an elicitation shows that a client responded, not that a person did, so an approval given through one still needs user verification that software cannot supply. |
| Elicitation, URL mode: clients "MUST NOT automatically pre-fetch the URL or any of its metadata." | R3.4 applies the same principle to every address in the AI's output. |
| Elicitation, URL mode, for sensitive data: "sensitive credentials never pass through the LLM context, MCP client or any intermediate MCP servers" | R2.1 and R2.3. |

## Agents Rule of Two

Meta's Agents Rule of Two (31 October 2025) states that, until prompt injection can be reliably detected and refused, "agents must satisfy no more than two of the following three properties within a session":

- "[A] An agent can process untrustworthy inputs"
- "[B] An agent can have access to sensitive systems or private data"
- "[C] An agent can change state or communicate externally"

An AI operations system always has [A] and [B]: it reads logs, alerts and tickets that anyone can influence, and it reads production systems. Zero-trust AI operations is therefore an [AB] configuration, and its rules remove [C] from the AI:

- **Changing state** requires a person's approval of the exact change (Rule 4). The Rule of Two allows this: an agent that needs all three properties "at a minimum requires supervision — via human-in-the-loop approval or another reliable means of validation".
- **Communicating externally** is removed, restricted to allow-listed destinations, or approved like a change (R3.3), and automatic fetches of addresses in the AI's output are blocked (R3.4). Retrieval of public information is kept apart from managed systems (R3.5), which corresponds to the "one-way switch" the Rule of Two describes between configurations.

## The lethal trifecta

Simon Willison's lethal trifecta (16 June 2025) is "Access to your private data", "Exposure to untrusted content" and "The ability to externally communicate in a way that could be used to steal your data". OWASP LLM01:2026 restates it: "removing any one leg removes them".

An AI operations system cannot remove the first two legs and still do its job. Rule 3 removes the third: the AI's code has no network, including name resolution (R3.2); every other channel is removed, allow-listed or approved (R3.3); nothing the AI writes is fetched automatically (R3.4); and public retrieval cannot carry data out (R3.5).

## Agentic Trust Framework

The Agentic Trust Framework (ATF), by Josh Woodruff of MassiveScale.AI and published on the Cloud Security Alliance blog, applies zero trust to AI agents in general, through five elements and a maturity model in which an agent earns autonomy from Intern ("observe only") through Junior ("recommend with approval") and Senior ("act with notification") to Principal ("autonomous within domain").

The two are complementary. ATF governs agents of every kind; zero-trust AI operations specifies one kind in depth: the AI that operates production infrastructure. Requirement identifiers below are ATF's own.

| ATF element | Zero-trust AI operations |
| --- | --- |
| Identity | R2.4 binds the AI's session identity to its session and requesting person, compatible with ATF I-2, Credential Binding. In addition, no credential for a managed system is ever in the AI's reach (R2.1). |
| Behavior | Rule 5 covers ATF B-1, Structured Logging, and B-2, Action Attribution, for every operation (R5.2) and every change (R5.1). |
| Data Governance | Not relied on. ATF D-2, Injection Prevention, asks for "Detection of prompt injection and adversarial inputs". This specification assumes detection will sometimes fail (section 4, R0.1) and limits what a steered AI can do. |
| Segmentation | Rule 1, including R1.9, and Rule 3. |
| Incident Response | R1.10: people can end any session, or all of them, and with them the sessions' access and identity. |

**The one difference in position.** For changes to managed systems, zero-trust AI operations holds the AI at ATF's Junior level permanently: it recommends, and a person approves. A good record does not promote it, because one injected log line can steer a model with a perfect history. Unattended change is still possible, but only as pre-approved automations and classes that people define in advance (R4.8, R4.9), never as autonomy the AI has earned.

## OWASP Agent Control Standard

The OWASP Agent Control Standard (ACS) describes itself as "a wire specification": "It lets a separate Guardian Agent inspect what an AI agent is about to do and permit, deny, or modify that action before it happens". It standardises where and how a policy decision is made; this specification says which outcomes the platform must guarantee.

The two fit together. An ACS deployment can serve as the enforcement point for R1.4 and the membership decision for R4.9, provided that:

- its decisions come from deterministic policy, not from a model's judgement (R0.1);
- it denies, rather than proceeds, when a decision cannot be made (R0.2). The reference implementation starts with "Failure posture: proceed", and the posture can be changed to deny.
