# Before an AI gets write access to production

25 questions to ask any AI operations tool before it touches production, and what a good answer sounds like. Ask every vendor, and write down the answers.

Part of [Zero-trust AI operations](README.md): the questions apply the [five rules](README.md#the-five-rules) to a real purchase. Licensed under [CC BY 4.0](LICENSE).

## Reach: what can it see?

1. **Which systems can the AI reach, and through what?**
    - *Good answer:* Named systems, through each vendor's own API, scoped by connection.
    - *Red flag:* "It can reach anything on your network."
2. **Can you limit it to read-only on a given system?**
    - *Good answer:* A per-connection switch that refuses changes, ideally before a proposal is even created.
    - *Red flag:* Read-only is a prompt instruction.
3. **When a system is not configured, is it hidden from the AI entirely?**
    - *Good answer:* Yes.
    - *Red flag:* The AI sees every possible tool and is told not to use some.
4. **Are dangerous endpoints fenced?** Passwords, user management, sessions, tokens.
    - *Good answer:* Fenced in code, so they cannot be searched or called.
    - *Red flag:* "We trust the model not to."

## Credentials: what does it hold?

5. **Does the model ever see or hold a credential?**
    - *Good answer:* No. Credentials stay server-side and are attached outside the model.
    - *Red flag:* An API key in the agent's environment.
6. **How are secrets for a job delivered?**
    - *Good answer:* Released only after approval, sealed to the one host that runs the job, and scrubbed from the output.
    - *Red flag:* Passed in plain text through the conversation.
7. **Can you revoke the tool's access in your own systems at any time?**
    - *Good answer:* Yes, because it uses dedicated, least-privilege service accounts you create.
    - *Red flag:* A shared admin account.

## Where the AI's code runs

8. **If the AI writes code, where does it run?**
    - *Good answer:* In a sandbox with networking switched off and a time limit.
    - *Red flag:* On a jump host with network access.
9. **Can the AI's own tools reach your private network directly?**
    - *Good answer:* No. Your systems are reachable only through the governed API.
    - *Red flag:* "It's inside the VPN."
10. **What identity does each AI run use, and when does it expire?**
    - *Good answer:* One bound to the run, the case and the person who started it, revoked when the run ends.
    - *Red flag:* One long-lived service token.

## Changes: who decides?

11. **Can the AI change anything without a person approving it first?**
    - *Good answer:* Not on your systems. Every change is a proposal.
    - *Red flag:* "Only low-risk changes run automatically", with no definition of low-risk.
12. **Is what runs exactly what was approved?**
    - *Good answer:* The approved content is pinned by hash and checked again when it runs.
    - *Red flag:* The approval covers a description, and the AI fills in the details later.
13. **Can the AI, or another automated process, approve?**
    - *Good answer:* No. Approve and execute accept only a signed-in person; agents, routines and machine tokens are refused.
    - *Red flag:* An API key can approve.
14. **Does a proposal carry its reason and evidence?**
    - *Good answer:* Reason, risk, target, exact content and evidence from the same investigation, plus optional preconditions, stop conditions and rollback.
    - *Red flag:* "Trust me."
15. **Can you require a second factor on approvals?**
    - *Good answer:* Yes: a push to the approver's phone, on every channel.
    - *Red flag:* Approval is a chat button anyone in the channel can press.
16. **What runs without a fresh approval each time, and what can it do?** Schedules, routines, automations.
    - *Good answer:* A clear list, set up by a person with the right role, with unattended AI limited to reads.
    - *Red flag:* "Everything is approved", with no exceptions listed.

## Hosts and endpoints

17. **How does work reach a server?**
    - *Good answer:* An enrolled agent with its own identity that runs only signed playbooks and verifies the signature itself.
    - *Red flag:* SSH with a shared key.
18. **Can one approval fan out to many hosts, and is each one still tracked?**
    - *Good answer:* Yes, to a bounded number of hosts, each with its own signed work item.
    - *Red flag:* Unbounded fan-out.
19. **For temporary admin access, who gets the second-factor prompt?**
    - *Good answer:* The approver, never the requester, and the access expires by itself.
    - *Red flag:* Permanent group membership "for now".

## Records: can you prove what happened?

20. **What does a decision record contain?**
    - *Good answer:* Who approved, over which channel, whether step-up was required and passed, the timing, and what ran.
    - *Red flag:* A chat log.
21. **Is that record tamper-evident, and where else does it go?**
    - *Good answer:* An honest answer, and an export to your own SIEM or log platform.
    - *Red flag:* "Immutable", with no mechanism behind it.
22. **Can you watch what the AI is reading while it works?**
    - *Good answer:* A live trail, with the evidence kept in the case.
    - *Red flag:* Only a final summary.

## The model and your data

23. **Which model, under whose account?**
    - *Good answer:* Yours: your seats or your endpoint, under your provider's terms.
    - *Red flag:* The vendor's account, with no say over retention.
24. **What stops a prompt injection from becoming an action?**
    - *Good answer:* "It can propose something wrong, but a person approves the exact content." Recalled memory is treated as data, and outbound mail goes only to allow-listed recipients.
    - *Red flag:* "Our model can't be prompt-injected."

## The vendor

25. **What can the vendor itself reach?**
    - *Good answer:* A specific, honest answer, a public sub-processor list and a DPA.
    - *Red flag:* "Nobody can access anything", or no answer.
