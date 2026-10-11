# Before an AI gets access to production

25 questions to ask any AI operations product, or yourself if you are building one, before it touches production. Each comes with what a good answer and a red flag sound like, and ten come with something to ask the vendor to show you during a trial.

The questions apply [the specification](SPECIFICATION.md#5-requirements); the tags after each question, such as `R1.1`, point to the requirements it tests. This checklist is informative; the specification decides. Licensed under [CC BY 4.0](LICENSE).

**How to use it.** Ask every vendor the same questions, in writing. During a trial, ask them to show rather than tell, and test the product, not the model: when the AI declines to do something, that tells you about the model, not about what the product prevents. A plain "no, and here is how we limit the risk instead" is a respectable answer; a vague answer, or one that turns out to be untrue, is the real red flag.

## Rule 1. Act only as the person who asked

**1. Whose permissions does the AI use on each system, and what enforces them?** `R1.1` `R1.2` `R1.3`

- **Good answer:** The requesting person's, checked by the product on every request, for that session only, down to the namespace, mailbox or index, and never more than that person's own account on the system allows. A revoked permission takes effect within an hour at most.
- **Red flag:** A broad service account with no per-person check, or "the agent knows what it is allowed to do".
- **Ask them to show:** As a user without access to a system, have the AI read it. The product refuses.

**2. How do you take an operation away from the AI, and what happens to a command nobody has classified?** `R1.4` `R1.5` `R0.2`

- **Good answer:** Operations not offered do not exist for the AI. Every request is classified as a read or a change by what it does; commands and queries count as changes unless the product enforces that they only read; and if a check cannot complete, nothing runs.
- **Red flag:** A line in the system prompt, "GET requests are reads", or "the tool says it is read-only".

**3. What can the AI do when nobody is watching, such as on a schedule?** `R1.7` `R4.10` `R4.11`

- **Good answer:** Read and propose, within a scope the trigger states. Anything that changes a system unattended is an automation or a narrow class of change that people defined in advance, with limits and an expiry of at most a year. Each run uses the current permissions of the person responsible, and stops if that person leaves.
- **Red flag:** Unattended remediation, with the AI deciding what to fix.

**4. Who else can see what the AI read on my behalf?** `R1.10` `R1.11`

- **Good answer:** Only people with access to what it came from, down to the namespace, mailbox or index, including anything the AI later derives from it through memory or summaries, unless someone with that access shares it deliberately. Other customers never can, including other customers of the same service provider. In a shared channel, the AI works only with what everyone there may see.
- **Red flag:** Memory, knowledge or search shared across users or customers.

**5. Can you stop everything at once, and narrow what the AI can reach?** `R1.12` `R1.13` `R1.8` `R1.9`

- **Good answer:** One action ends every session, withdraws pending proposals, stops running changes and suspends approved changes, automations and classes, within 60 seconds. Sessions end within 24 hours, requests are rate-limited, and you can restrict the AI to less than the requesting person's own access.
- **Red flag:** "We would redeploy", or sessions that never end.

## Rule 2. Never give the AI a credential

**6. Can the model or its code ever see a credential the product uses, including the key for the model provider?** `R2.1` `R2.2` `R2.5`

- **Good answer:** No. The product attaches credentials to each request, outside the model.
- **Red flag:** API keys in the agent's environment variables or configuration files.
- **Ask them to show:** From the AI's execution environment, list the environment and files. No credential appears, apart from a session identity that is useless elsewhere.

**7. What does the AI see when it reads a system that holds credentials, or resets one?** `R2.6` `R1.6`

- **Good answer:** Credential values are replaced before they reach the AI: every credential the product itself holds, in common encodings; fields the system marks as secret; key material; and formats such as Kubernetes secrets, kubeconfig files, device configurations and infrastructure-as-code state. A reset is approved on its own, and the new value never reaches the AI. The vendor says plainly that credentials in free text are caught only on a best-effort basis.
- **Red flag:** "The model knows not to repeat them."
- **Ask them to show:** Read a Kubernetes secret and a network device configuration through the AI. The values are replaced.

**8. How does an approved job get a credential it needs?** `R2.3`

- **Good answer:** Preferably the product attaches it itself, so the job never holds it. Otherwise the proposal names the credential; the value is released only after approval, only to a job the product runs and fences, never to a script on a target, and the AI sees only whether it succeeded unless a person releases the output.
- **Red flag:** Credentials passed in plain text through the conversation.

**9. What identity does an AI session use, and what happens if it is copied?** `R2.4`

- **Good answer:** One bound to the session and the person who started it, accepted only by the product, never passed on to your systems, which ends with the session, and which the AI cannot read or which is useless from anywhere else.
- **Red flag:** A long-lived bearer token shared by every run.
- **Ask them to show:** Copy the session's token, if it is readable at all, and use it from another machine. It is refused.

## Rule 3. Give the AI no network of its own

**10. Can the AI reach your private network directly?** `R3.1`

- **Good answer:** No. It reaches your systems only through the product, which also runs and classifies any diagnostics such as ping or name lookups.
- **Red flag:** "It runs inside your VPN."

**11. Where does code the AI writes run, and can it, or a browser it drives, reach the internet or look up a name?** `R3.2`

- **Good answer:** In an isolated environment with no network, and no DNS, apart from the product's own interface. Any browser it uses is run by the product, which keeps the cookies.
- **Red flag:** A shell on a server with internet access.
- **Ask them to show:** From the AI's environment, request `https://example.com` and look up a name in a domain you control. The request fails, and no lookup reaches your name servers.

**12. Can the AI browse the web in the same session in which it reads your systems?** `R3.5`

- **Good answer:** Only in a way that what it read cannot steer: it queries a disclosed search or lookup service and opens only the results or sources you allow-list, or research happens in a separate session with no access to your systems, or a person approves each request. The vendor says the query itself still reaches that service.
- **Red flag:** "Yes, it looks errors up online while it investigates", with any address it likes.
- **Ask them to show:** Have the AI read a value from one of your systems, then fetch an address that contains it. It cannot, or a person must approve it.

**13. Which channels can carry anything out, and can anything in the AI's answers load or run by itself?** `R3.3` `R3.4` `R3.8`

- **Good answer:** A short, named list of channels. Each is limited to destinations you allow-list, with a stated audience that may see what the AI read, or each message is approved. No interface, chat or mail integration loads images or link previews from the AI's output, nothing in an answer runs as code, and links to other destinations cannot be clicked directly.
- **Red flag:** "It can post to any channel it has access to", or AI answers that render remote images.
- **Ask them to show:** Make the AI's answer include an image from a server you control, and open it everywhere the product shows answers. Your server receives nothing.

**14. Which third parties receive what the AI reads, and who chooses the account?** `R3.6`

- **Good answer:** A complete list: model, embedding, search, filtering, telemetry, tracing, hosting and delivery providers, with account, region, retention and training use. The product, not the AI, chooses the account for every request, and you can use your own.
- **Red flag:** Only the model provider is named, or the AI could use a key it found in a document.

## Rule 4. Let only a person approve a change, exactly as it will run

**15. Can the AI change anything without a person approving it first?** `R4.1` `R4.11` `R0.1` `R0.3`

- **Good answer:** Only within automations and narrow classes of change that people defined in advance, limited to named operations and targets, with a maximum scope, rate and expiry. Fixed rules decide what falls in a class, never the AI or another model, and no setting the AI can reach turns approval off.
- **Red flag:** "Low-risk changes run automatically", with the AI deciding what counts as low risk.
- **Ask them to show:** Ask the AI to make a change. Nothing changes until a person approves.

**16. Is what runs exactly what was approved, including everything it downloads, and only once?** `R4.2` `R4.3` `R3.7`

- **Good answer:** Yes. Approval binds to a digest of the exact content, including its timing and a bounded retry policy; packages and images are pinned, or captured by digest when the proposal is made; plans are checked again before applying; the approval covers one run; and code the product runs can reach only the destinations it declared. Parts that run on your hosts are flagged as having the host's network.
- **Red flag:** The approval covers a description, and the details are filled in afterwards, or a script pulls `latest` at run time.
- **Ask them to show:** Alter a proposal after it has been approved. Execution refuses.

**17. What happens if the target changes, or the approval sits unused for a week?** `R4.4` `R4.5`

- **Good answer:** Targets are bound by identifiers that are not reused, or by a fingerprint of their state that the product computes, and are checked again at execution. Approvals expire within 30 days.
- **Red flag:** Targets by name only, and approvals that never expire.

**18. Who can approve, and how do you know it was them and not software?** `R4.6` `R4.7`

- **Good answer:** Only a person with permission to make the change, and with access to everything the session read, checked again when it runs. Every approval is confirmed with user verification, such as a passkey, bound to that specific proposal and used once; you can restrict which authenticators count; and adding an authenticator needs an existing one.
- **Red flag:** "They are signed in", an approve or deny push, an API key that can approve, or "anyone on the team can approve anything".
- **Ask them to show:** Try to approve with a machine token, as a person without permission, and by driving a signed-in browser with an automation tool. All are refused.

**19. What does the person approving actually see?** `R4.9` `R4.8`

- **Good answer:** The exact content as plain text, with hidden characters made visible; the targets, the identity it runs as, the credentials and destinations it uses, and any part that runs on a host; timing, retries and any step that cannot be stopped; what else the session proposed and what it read; and evidence drawn from the systems themselves, with untrusted sources marked. Anything the AI wrote is marked as such. Several proposals approved together are each shown in full and can be rejected one by one.
- **Red flag:** A summary the AI wrote, presented as the reason to approve.

**20. Can the AI change its own schedules, classes, allow-lists, instructions or memory, and can a tool server change its tools?** `R4.10` `R4.11` `R4.12` `R4.14`

- **Good answer:** Only through proposals a person with that permission approves. What the AI writes to memory is marked as its own and never becomes instructions without approval, and updated tool definitions need approval too. Classes allow only listed operations and parameter values, never a free-form command.
- **Red flag:** Yes, directly.

**21. Can you require someone other than the requester, or two people, to approve some changes, and can you freeze changes?** `R4.13` `R4.15`

- **Good answer:** Yes, for kinds of change you choose, including a change authority that does not itself hold admin rights, and freeze periods in which nothing runs.
- **Red flag:** No way to separate the person who asked from the person who approves.

## Rule 5. Record every decision where the AI cannot change it

**22. What is recorded?** `R5.1` `R5.2` `R5.3`

- **Good answer:** Every instruction to the AI and every answer, with the model and version used and the instructions it was given. Every request it made or was refused, with its parameters. For every proposal, including rejected and expired ones: the exact content, the targets, the evidence, who decided, how they verified, what ran and the result.
- **Red flag:** A chat transcript, or nothing for reads.

**23. Can the AI alter records, and how are they kept?** `R5.4` `R5.5` `R5.6` `R5.7` `R5.8` `R5.9`

- **Good answer:** The AI cannot touch them. Text in them is shown as plain text everywhere, including exports. The vendor names how records are protected, they are kept for at least a year and are not dropped because of volume, and they reach your own logging or SIEM platform.
- **Red flag:** "Immutable", with no mechanism behind the word.

## The vendor

These questions fall outside the specification, which is about the AI, but every buyer should ask them.

**24. What can the vendor itself reach in your environment?**

- **Good answer:** A specific answer, how that access is controlled and recorded, a public list of sub-processors and a data processing agreement.
- **Red flag:** "Nobody can access anything", or no answer.

**25. Will they put their answers in writing, including what the product does not prevent?** See [CONFORMANCE.md](CONFORMANCE.md).

- **Good answer:** Yes: a dated statement with evidence for each requirement, such as a conformance statement against this specification, the settings that would weaken it, and the risks it leaves.
- **Red flag:** "We're fully compliant", with nothing written down.
