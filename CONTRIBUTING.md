# Contributing

Thank you for helping make zero-trust AI operations a sharper standard. This repository holds a specification, so contributions are mostly about requirements, wording, evidence and scope.

## Start with an issue or a discussion

Open an issue before a pull request, so the change can be discussed before anyone edits the text.

- **A rule or requirement is wrong, unclear or missing something:** open a [rule feedback issue](../../issues/new?template=rule-feedback.yml).
- **A checklist question should be added, changed or removed:** open a [checklist issue](../../issues/new?template=checklist-question.yml).
- **An open question or an idea that is not ready yet:** start a [discussion](../../discussions).

Typos and broken links can go straight to a pull request.

## Evidence over opinion

A proposal is strongest when it points at something a reader can check:

- a standard or its exact wording (for example a NIST SP 800-207 tenet or an OWASP entry),
- a documented incident or failure mode,
- a published design or specification.

"I think" is welcome as a start; a source makes it mergeable.

## Editorial principles

- **Vendor-neutral.** The specification, the crosswalk and the checklist name no product, including the maintainer's. Pull requests that promote a product are closed.
- **Testable.** Every requirement must be checkable from outside, and every rule keeps its verification steps. Say what a person can verify, not what a tool claims.
- **Quote sources exactly.** Quotations from NIST, OWASP, the MCP specification and other sources are verbatim, with the edition named.
- **The platform, not the model.** No requirement may be met by instructing the AI or by a model's judgement (R0.1). Proposals that rely on either are declined.
- **Small, reviewable changes.** One requirement or one question per pull request.
- **British spelling**, matching the existing text.

## Public review

Versions marked as drafts, such as 1.0.0-draft.1, are open for public review. During review, requirements and identifiers may change. Every issue raised against a draft gets a public answer, accepted, changed or declined with a reason, before version 1.0.0 is released.

## Versioning

From version 1.0.0 the specification uses [semantic versioning](https://semver.org/):

- **Major:** a MUST requirement is added, removed or changes meaning, a rule is added or removed, or a verification step is added or changed in a way that a conforming system could fail.
- **Minor:** a SHOULD or MAY requirement is added or removed, informative text changes in substance, or a checklist question is added or removed.
- **Patch:** wording that changes no requirement, typos and links.

Requirement identifiers such as R4.2 never change meaning within a major version. A withdrawn requirement keeps its identifier, marked as withdrawn.

Every release is recorded in [CHANGELOG.md](CHANGELOG.md) and tagged `vX.Y.Z`, and the version in [CITATION.cff](CITATION.cff) is bumped with it.

## Licence of contributions

By contributing you agree that your contribution is licensed under [CC BY 4.0](LICENSE), the licence of this repository.

## Code of Conduct

Everyone taking part follows the [Code of Conduct](CODE_OF_CONDUCT.md).
