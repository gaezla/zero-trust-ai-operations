# Before an AI gets access to production

25 questions to ask any AI operations product, or yourself if you are building one, before it touches production. Each comes with what a good answer and a red flag sound like, and most with something to ask the vendor to show you during a trial.

The questions apply [the specification](SPECIFICATION.md#5-requirements); the tags after each question, such as `R1.1`, point to the requirements it tests. Licensed under [CC BY 4.0](LICENSE).

**How to use it.** Ask every vendor the same questions, in writing. During a trial, ask them to show rather than tell, and test the product, not the model: when the AI declines to do something, that tells you about the model, not about what the product prevents. A plain "no, and here is how we limit the risk instead" is a respectable answer; a vague answer, or one that turns out to be untrue, is the real red flag.

## Rule 1. Act only as the person who asked

**1. Whose permissions does the AI use on each system, and what enforces them?** `R1.1` `R1.2` `R1.3`

- **Good answer:** The requesting person's own, checked by the product on every operation, for that session only. Even where the product connects with its own service account, it checks the person's permissions first.
- **Red flag:** A broad service account with no per-person check, or "the agent knows what it is allowed to do".
- **Ask them to show:** As a user without access to a system, have the AI read it. The product refuses.

**2. How do you take an operation away from the AI, and what happens to an operation nobody has classified?** `R1.4` `R1.5` `R1.6` `R0.2`

- **Good answer:** In configuration: operations not offered do not exist for the AI, and password, MFA and token management is not offered at all. Every operation is classified as a read or a change by what it does, anything unclassified is treated as a change, and if a check cannot complete, nothing runs.
- **Red flag:** A line in the system prompt, or "GET requests are reads".

**3. What can the AI do when nobody is watching, such as on a schedule?** `R1.7` `R4.8`

- **Good answer:** Read and propose. Anything else is an automation a person approved in advance, which the AI cannot change. Each run uses the current permissions of the person who set it up, and stops if that person leaves.
- **Red flag:** Unattended remediation, with the AI deciding what to fix.

**4. Who else can see what the AI read on my behalf?** `R1.9`

- **Good answer:** Only people with access to the same systems, unless someone with that access shares it deliberately. Other customers never can.
- **Red flag:** Memory, knowledge or search shared across users or customers.

**5. Can you stop every AI session at once, and limit how hard it hits a system?** `R1.10` `R1.8`

- **Good answer:** Yes: one action ends all sessions, their access and their identities, and operations are rate-limited per session and per system.
- **Red flag:** "We would redeploy", or no limits on reads.

## Rule 2. Never give the AI a credential

**6. Can the model or its code ever see a credential, including the key for the model provider?** `R2.1` `R2.2` `R2.5`

- **Good answer:** No. The product attaches credentials to each operation, outside the model.
- **Red flag:** API keys in the agent's environment variables or configuration files.
- **Ask them to show:** From the AI's execution environment, list the environment and files. No secret appears.

**7. What does the AI see when it reads a system that holds secrets, such as a Kubernetes secret?** `R2.6`

- **Good answer:** Names and types only. Values are replaced before they reach the AI.
- **Red flag:** "The model knows not to repeat them."
- **Ask them to show:** Read a Kubernetes secret through the AI. The values are replaced.

**8. How does an approved job get a secret it needs?** `R2.3`

- **Good answer:** The proposal names the secret; the value is released only after approval, delivered only to whatever runs the job, and removed from any output the AI can read.
- **Red flag:** Secrets passed in plain text through the conversation.

**9. What identity does an AI session use, and what happens if it is copied?** `R2.4`

- **Good answer:** One bound to the session and the person who started it, which ends with the session, and which the AI cannot read or which is useless from anywhere else.
- **Red flag:** A long-lived bearer token shared by every run.
- **Ask them to show:** Copy the session's token, if it is readable at all, and use it from another machine. It is refused.

## Rule 3. Give the AI no network of its own

**10. Can the AI reach your private network directly?** `R3.1`

- **Good answer:** No. It reaches your systems only through the product.
- **Red flag:** "It runs inside your VPN."

**11. Where does code the AI writes run, and can it reach the internet or look up a name?** `R3.2`

- **Good answer:** In an isolated environment with no network, and no DNS, apart from the product's own interface.
- **Red flag:** A shell on a server with internet access.
- **Ask them to show:** From the AI's environment, request `https://example.com` and look up a name in a domain you control. The request fails, and no lookup reaches your name servers.

**12. Can the AI browse the web in the same session in which it reads your systems?** `R3.5`

- **Good answer:** Only in a way the AI cannot steer: the product fetches only search results or sources you allow-list, or research happens in a separate session with no access to your systems, or a person approves each request.
- **Red flag:** "Yes, it looks errors up online while it investigates", with any address it likes.
- **Ask them to show:** Have the AI read a value from one of your systems, then fetch an address that contains it. It cannot, or a person must approve it.

**13. Which channels can carry anything out, and does any screen load addresses from the AI's answers by itself?** `R3.3` `R3.4`

- **Good answer:** A short, named list of channels, each limited to destinations you allow-list or approved like a change. No interface, chat or mail integration loads images or link previews from the AI's output.
- **Red flag:** "It can post to any channel it has access to", or AI answers that render remote images.
- **Ask them to show:** Make the AI's answer include an image from a server you control, and open it everywhere the product shows answers. Your server receives nothing.

**14. Which model and search providers receive what the AI reads, and who chooses the account?** `R3.6`

- **Good answer:** Named and documented, chosen by you, with the account set by the product so the AI cannot send requests anywhere else.
- **Red flag:** No clear answer, or an AI that could use a key it found in a document.

## Rule 4. Let only a person approve a change, exactly as it will run

**15. Can the AI change anything without a person approving it first?** `R4.1` `R4.9` `R0.1`

- **Good answer:** No. Every change is a proposal. Anything that runs unattended was approved by a person in advance, and the product, not the AI or another model, decides what falls under it.
- **Red flag:** "Low-risk changes run automatically", with the AI deciding what counts as low risk.
- **Ask them to show:** Ask the AI to make a change. Nothing changes until a person approves.

**16. Is what runs exactly what was approved, including everything it downloads?** `R4.2` `R4.3`

- **Good answer:** Yes. Approval binds to a digest of the exact content, every image or package it pulls is pinned by digest, and the digest is checked again when it runs.
- **Red flag:** The approval covers a description, and the details are filled in afterwards, or a script pulls `latest` at run time.
- **Ask them to show:** Alter a proposal after it has been approved. Execution refuses.

**17. What happens if the target changes, or the approval sits unused for a week?** `R4.4` `R4.5`

- **Good answer:** Targets are identified by IDs that are never reused and checked again at execution, and approvals expire.
- **Red flag:** Targets by name only, and approvals that never expire.

**18. How do you know a person, and not software, gave the approval?** `R4.6`

- **Good answer:** The approval screen refuses agents, schedules and machine tokens, and every approval is confirmed with a passkey or a confirmation on a separate device, which software cannot supply.
- **Red flag:** "They are signed in", an approve button in a chat channel, or an API key that can approve.
- **Ask them to show:** Try to approve with a machine token, and by driving a signed-in browser with an automation tool. Both are refused.

**19. What does the person approving actually see?** `R4.7`

- **Good answer:** The exact content that will run, with hidden characters made visible, the targets, the expiry, and evidence drawn from the systems themselves. Anything the AI wrote is marked as such.
- **Red flag:** A summary the AI wrote, presented as the reason to approve.

**20. Can the AI change its own schedules, rules, prompts, skills or memory?** `R4.8` `R4.9` `R4.10`

- **Good answer:** Only as proposals a person approves. What it writes to memory is labelled and treated as untrusted when read back.
- **Red flag:** Yes, directly.

**21. Can you require a second person to approve some changes?** `R4.11`

- **Good answer:** Yes, for kinds of change you choose.
- **Red flag:** No way to separate the person who asked from the person who approves.

## Rule 5. Record every decision where the AI cannot change it

**22. What is recorded, for changes and for reads?** `R5.1` `R5.2`

- **Good answer:** For every change: who asked, the exact content, the targets, the evidence, who approved or rejected it, how it was verified, what ran and the result. For every read: what, from where, when and for whom.
- **Red flag:** A chat transcript, or nothing for reads.

**23. Can the AI alter records, and how are they kept?** `R5.3` `R5.4` `R5.5` `R5.6` `R5.7`

- **Good answer:** The AI cannot touch them, its text is shown as plain text everywhere, and the vendor names how records are protected, how long they are kept, and how they reach your own logging or SIEM platform.
- **Red flag:** "Immutable", with no mechanism behind the word.

## The vendor

These questions fall outside the specification, which is about the AI, but every buyer should ask them.

**24. What can the vendor itself reach in your environment?**

- **Good answer:** A specific answer, how that access is controlled and recorded, a public list of sub-processors and a data processing agreement.
- **Red flag:** "Nobody can access anything", or no answer.

**25. Will they put their answers in writing, including what the product does not prevent?** See [CONFORMANCE.md](CONFORMANCE.md).

- **Good answer:** Yes: a dated statement with evidence for each requirement, such as a conformance statement against this specification, the settings that would weaken it, and the risks it leaves.
- **Red flag:** "We're fully compliant", with nothing written down.
