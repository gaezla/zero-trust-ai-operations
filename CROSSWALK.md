# Crosswalk: zero-trust AI operations and related work

How the requirements in [SPECIFICATION.md](SPECIFICATION.md) relate to the standards and guidance security teams already use. Use it to show an auditor where a control comes from, or to check whether a framework you already follow covers the same ground.

Every quotation below is taken word for word from the source named, as published on the date given in the [references](SPECIFICATION.md#7-references).

## Contents

- [NIST SP 800-207, Zero Trust Architecture](#nist-sp-800-207-zero-trust-architecture)
- [OWASP Top 10 for LLM Applications 2026](#owasp-top-10-for-llm-applications-2026)
- [Agents Rule of Two](#agents-rule-of-two)
- [The lethal trifecta](#the-lethal-trifecta)
- [CSA Agentic Trust Framework](#csa-agentic-trust-framework)

## NIST SP 800-207, Zero Trust Architecture

Zero-trust AI operations treats the AI as a subject in the sense of NIST SP 800-207: a non-person entity that requests access to resources, and that is not trusted because of where it runs or what it has done before.

### The tenets (section 2.1)

| Tenet | Where it applies |
| --- | --- |
| 1. "All data sources and computing services are considered resources." | Every managed system the AI can reach is a resource (section 3, *Managed system*). |
| 2. "All communication is secured regardless of network location. Network location alone does not imply trust." | Rule 3: the AI gets no network path of its own (R3.1, R3.2). |
| 3. "Access to individual enterprise resources is granted on a per-session basis." … "Access should also be granted with the least privileges needed to complete the task." | Rule 1 (R1.3) and Rule 2 (R2.4): access lasts one session, within the requesting person's permissions. |
| 4. "Access to resources is determined by dynamic policy" | Rule 1 (R1.1): authorisation against the requesting person's permissions at the time of each operation. |
| 5. "No asset is inherently trusted." | The threat model (section 4): the AI is assumed to be wrong or steered at any moment. |
| 6. "All resource authentication and authorization are dynamic and strictly enforced before access is allowed." | Rule 1 (R1.1, R1.2) for reads; Rule 4 (R4.1 to R4.3) for changes, where the approval is the enforcement. |
| 7. "The enterprise collects as much information as possible about the current state of assets, network infrastructure and communications and uses it to improve its security posture." | Rule 5: decision records (R5.1) and read records (R5.2). |

### The logical components (section 3)

| NIST component | In an AI operations system |
| --- | --- |
| Subject | The AI, acting for the requesting person. |
| Policy engine (PE), which "makes and logs the decision (as approved, or denied)" | Authorisation of each operation (R1.1), and for changes, the person's approval (R4.1) recorded in the decision record (R5.1). |
| Policy administrator (PA), which would "generate any session-specific authentication and authentication token or credential" | The platform's credential handling (R2.2, R2.3). Zero-trust AI operations goes further: the credential is used on the AI's behalf and never handed to it (R2.1). |
| Policy enforcement point (PEP) | The platform's mediated interface, the only path from the AI to a managed system (R3.1). |

### Non-person entities (section 5.7)

Section 5.7 warns that "an attacker will be able to induce or coerce an NPE to perform some task that the attacker is not privileged to perform", and that "an attacker could gain access to a software agent’s credentials and impersonate the agent when performing tasks". Rule 4 addresses the first risk: a coerced AI can propose, but cannot change anything without a person. Rule 2 addresses the second: there is no credential in the AI's reach to take.

## OWASP Top 10 for LLM Applications 2026

The 2026 edition renumbered the list. Excessive Agency, LLM06 in the 2025 edition, is LLM03 in 2026.

### LLM01:2026 Prompt Injection

The entry states that "no reliable prevention mechanism exists today" and that "Defense is therefore architectural rather than interceptive." Zero-trust AI operations is an architecture of that kind. Its prevention and mitigation strategies map as follows.

| LLM01:2026 strategy | Requirements |
| --- | --- |
| 4. "Hold credentials and state-change capability in application code, not the model, and grant least privilege per operation." | R2.1, R2.2 (credentials); R4.1 (state change); R1.1 (least privilege per operation). |
| 7. "Require explicit human confirmation before any privileged, irreversible, or externally visible action, surfacing the exact rendered action rather than a summary to the reviewer." | R4.1, R4.4; R3.3 for externally visible actions. |
| 8. "Budget agent capabilities with the Rule of Two as a floor" | The threat model (section 4) and Rule 3. See [Agents Rule of Two](#agents-rule-of-two). |
| 9. "Treat agent memory writes as privileged operations." | R4.9. |

Strategies 1, 2, 3, 5, 6 and 10 (system-prompt constraints, output schemas, modality filtering, invisible-character stripping, provenance labelling and tool supply chain) reduce how often injection succeeds. They complement this specification; none of its requirements relies on them.

### LLM03:2026 Excessive Agency

The entry names three root causes: "excessive functionality", "excessive permissions" and "excessive autonomy".

| LLM03:2026 strategy | Requirements |
| --- | --- |
| 1. Minimize tools | R1.4, R1.5 |
| 2. Minimize tool functionality | R1.4 |
| 3. Avoid open-ended tools | R3.2 (code runs without network), R3.3 |
| 4. Minimize tool permissions | R1.1, R1.3 |
| 5. Execute tools in user's context | R1.1, R1.2 |
| 6. Require user approval | R4.1 to R4.4 |
| 7. Complete mediation | R1.1, R1.2, R4.6; the graduated enforcement it describes corresponds to R4.8 |
| 8. Monitor tool use | Rule 5 |
| 9. Rate limiting | Not covered. Rate limits remain good practice alongside this specification. |

## Agents Rule of Two

Meta's Agents Rule of Two (31 October 2025) states that, until prompt injection can be reliably detected and refused, "agents must satisfy no more than two of the following three properties within a session":

- "[A] An agent can process untrustworthy inputs"
- "[B] An agent can have access to sensitive systems or private data"
- "[C] An agent can change state or communicate externally"

An AI operations system always has [A] and [B]: it reads logs, alerts and tickets that anyone can influence, and it reads production systems. Zero-trust AI operations is therefore an [AB] configuration, and its rules remove [C] from the AI:

- **Changing state** requires a person's approval of the exact change (Rule 4). The Rule of Two allows this: an agent that needs all three properties "at a minimum requires supervision — via human-in-the-loop approval or another reliable means of validation".
- **Communicating externally** is removed, restricted to allow-listed destinations, or approved like a change (R3.3). Retrieval of public information is kept apart from managed systems (R3.4), which corresponds to the "one-way switch" the Rule of Two describes between configurations.

## The lethal trifecta

Simon Willison's lethal trifecta (16 June 2025) is "Access to your private data", "Exposure to untrusted content" and "The ability to externally communicate in a way that could be used to steal your data". OWASP LLM01:2026 restates it: "removing any one leg removes them".

An AI operations system cannot remove the first two legs and still do its job. Rule 3 removes the third: the AI's code has no network (R3.2), every other channel is removed, allow-listed or approved (R3.3), and public retrieval cannot carry data out (R3.4).

## CSA Agentic Trust Framework

The Agentic Trust Framework (ATF) from the Cloud Security Alliance applies zero trust to AI agents in general, through five elements and a maturity model in which an agent earns autonomy from Intern ("observe only") through Junior ("recommend with approval") and Senior ("act with notification") to Principal ("autonomous within domain").

The two are complementary. ATF governs agents of every kind; zero-trust AI operations specifies one kind in depth: the AI that changes production infrastructure.

| ATF element | Zero-trust AI operations |
| --- | --- |
| Identity | R2.4 binds the AI's session identity to its session and requesting person. Compatible with ATF I-2, Credential Binding. In addition, no credential for a managed system is ever in the AI's reach (R2.1). |
| Behavior | Rule 5 covers ATF B-1, Structured Logging, and B-2, Action Attribution, for every change. |
| Data Governance | Not relied on. ATF D-2, Injection Prevention, asks for "Detection of prompt injection and adversarial inputs". This specification assumes detection will sometimes fail (section 4) and limits what a steered AI can do. |
| Segmentation | Rule 1 and Rule 3. |
| Incident Response | Not covered. Kill switches and session revocation remain good practice. R2.4 requires that the AI's identity end with its session. |

**The one difference in position.** For changes to managed systems, zero-trust AI operations holds the AI at ATF's Junior level permanently: it recommends, and a person approves. A good record does not promote it, because one injected log line can steer a model with a perfect history. Unattended change is still possible, but only as pre-approved automations and classes that people define in advance (R4.7, R4.8), never as autonomy the AI has earned.
