# Before an AI gets write access to production

25 questions to ask any AI operations product, or yourself if you are building one, before it touches production. Each comes with what a good answer and a red flag sound like, and most with something to ask the vendor to show you during a trial.

The questions apply [the five rules](SPECIFICATION.md#5-the-five-rules); the tags after each question, such as `R1.1`, point to the requirement it tests. Licensed under [CC BY 4.0](LICENSE).

**How to use it.** Ask every vendor the same questions, in writing. During a trial, ask them to show rather than tell. A plain "no, and here is how we limit the risk instead" is a respectable answer; a vague answer, or one that turns out to be untrue, is the real red flag.

## Rule 1. Act only as the person who asked

**1. Whose permissions does the AI use on each system?** `R1.1` `R1.3`

- **Good answer:** The requesting person's own, checked by the platform on every operation, for that session only.
- **Red flag:** One service account with broad rights, shared by everyone who uses the AI.
- **Ask them to show:** As a user without access to a system, ask the AI to read it. The platform refuses.

**2. If the AI says an action is allowed, does anything check?** `R1.2`

- **Good answer:** Yes. Authorisation is decided by the platform, and nothing the AI says can change it.
- **Red flag:** "The agent knows which actions are safe."

**3. How do you limit a system to read-only, or take an operation away from the AI?** `R1.4`

- **Good answer:** In configuration. Operations that are not offered do not exist for the AI.
- **Red flag:** A line in the system prompt.

**4. Can the AI reach user, password, token or session management?** `R1.5`

- **Good answer:** No. Those endpoints are not offered to it.
- **Red flag:** "We trust the model not to use them."

**5. What can the AI do when nobody is watching, such as on a schedule?** `R1.6` `R4.7`

- **Good answer:** Read, plus automations a person approved in advance, which the AI cannot change.
- **Red flag:** Unattended remediation, with the AI deciding what to fix.

## Rule 2. Never give the AI a credential

**6. Can the model or its tools ever see a credential, including the key for the model provider?** `R2.1` `R2.2` `R2.5`

- **Good answer:** No. The platform attaches credentials to each operation, outside the model.
- **Red flag:** API keys in the agent's environment variables.
- **Ask them to show:** Ask the AI to print its environment and list its files. No secret appears.

**7. How does a job get a secret it needs?** `R2.3`

- **Good answer:** The secret is released only after approval, delivered only to whatever runs the job, and removed from any output the AI can read.
- **Red flag:** Secrets passed in plain text through the conversation.

**8. What identity does an AI session use, and when does it end?** `R2.4`

- **Good answer:** One bound to the session and the person who started it, unusable anywhere else, and revoked when the session ends.
- **Red flag:** One long-lived token shared by every run.

## Rule 3. Give the AI no network of its own

**9. Can the AI reach your private network directly?** `R3.1`

- **Good answer:** No. It reaches your systems only through the platform.
- **Red flag:** "It runs inside your VPN."

**10. Where does code the AI writes run, and can it reach the internet?** `R3.2`

- **Good answer:** In an isolated environment with no network, apart from the platform's own interface.
- **Red flag:** A shell on a server with internet access.
- **Ask them to show:** Ask the AI to run code that requests `https://example.com`. The request fails.

**11. Can the AI browse the web in the same session in which it reads your systems?** `R3.3` `R3.4`

- **Good answer:** No. Public research happens in a separate session with no access to your systems, from sources you allow-list, or with a person approving each request.
- **Red flag:** "Yes, it looks errors up online while it investigates."
- **Ask them to show:** Ask the AI to read a value from one of your systems, then fetch a URL that contains it. It cannot, or the fetch becomes a proposal for a person to approve.

**12. Which channels can the AI use to send anything out: mail, chat, tickets, webhooks, links?** `R3.3`

- **Good answer:** A short, named list, each restricted to destinations you allow-list, or approved like a change.
- **Red flag:** "It can post to any channel it has access to."

**13. Which model provider receives what the AI reads, and do you choose it?** `R3.5`

- **Good answer:** Named and documented, and your choice, under your own account and terms.
- **Red flag:** No clear answer, or the vendor's account with retention terms you cannot see.

## Rule 4. Let only a person approve a change, exactly as it will run

**14. Can the AI change anything without a person approving it first?** `R4.1` `R4.8`

- **Good answer:** No. Every change is a proposal. Anything that runs unattended was approved by a person in advance, and the platform, not the AI, decides what falls under it.
- **Red flag:** "Low-risk changes run automatically", with the AI deciding what counts as low risk.
- **Ask them to show:** Ask the AI to make a change. Nothing changes until a person approves.

**15. Is what runs exactly what was approved?** `R4.2`

- **Good answer:** Yes. Approval binds to the exact content, for example by hash, and is checked again when it runs.
- **Red flag:** The approval covers a description, and the AI fills in the details afterwards.
- **Ask them to show:** Alter a proposal after it has been approved. Execution refuses.

**16. Who can approve?** `R4.3`

- **Good answer:** Only a signed-in person. The approval interface itself refuses the AI, other agents, scheduled jobs and machine tokens.
- **Red flag:** An API key can approve.
- **Ask them to show:** Try to approve with a machine token. It is refused.

**17. What does the person approving actually see?** `R4.4`

- **Good answer:** The exact content that will run, with its reason and evidence.
- **Red flag:** A summary the AI wrote.

**18. Can approvals require a second factor, and does the record say how each one was given?** `R4.5`

- **Good answer:** Yes to both: a second factor bound to the approver, and the channel and assurance recorded for every approval.
- **Red flag:** A button in a chat channel that anyone in it can press.

**19. Who decides whether an operation is a read or a change?** `R4.6`

- **Good answer:** The platform, from what the operation does.
- **Red flag:** The AI, or the HTTP method alone.

**20. Can the AI create or edit schedules, automations, its own memory or its instructions?** `R4.7` `R4.9`

- **Good answer:** Only as proposals a person approves.
- **Red flag:** Yes, directly.

## Rule 5. Record every decision where the AI cannot change it

**21. What does the record of a change contain?** `R5.1`

- **Good answer:** Who asked, the exact content, the evidence, who approved or rejected it, when, over which channel, with what assurance, what ran and the result.
- **Red flag:** A chat transcript.

**22. Can you see what the AI read, while it works and afterwards?** `R5.2`

- **Good answer:** Yes: which system, what it read, and on whose behalf.
- **Red flag:** Only a final summary.

**23. Can the AI alter or delete records, and how are they kept?** `R5.3` `R5.4` `R5.5`

- **Good answer:** The AI cannot touch them. The vendor names how they are protected and how long they are kept, and they can be exported to your own logging or SIEM platform.
- **Red flag:** "Immutable", with no mechanism behind the word.

## The vendor

These questions fall outside the specification, which is about the AI, but every buyer should ask them.

**24. What can the vendor itself reach in your environment?**

- **Good answer:** A specific answer, how that access is controlled and recorded, a public list of sub-processors and a data processing agreement.
- **Red flag:** "Nobody can access anything", or no answer.

**25. Do they publish a conformance statement against this specification?** See [CONFORMANCE.md](CONFORMANCE.md).

- **Good answer:** Yes, dated, with evidence for each requirement, and the requirements they do not yet meet listed plainly.
- **Red flag:** "We're fully compliant", with nothing written down.
