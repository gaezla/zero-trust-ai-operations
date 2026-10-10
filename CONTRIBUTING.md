# Contributing

Thank you for helping make zero-trust AI operations a sharper standard. This repository holds a definition, five rules and a checklist, so contributions are mostly about wording, evidence and scope.

## Start with an issue or a discussion

Open an issue before a pull request, so the change can be discussed before anyone edits the text.

- **A rule is wrong, unclear or missing something:** open a [rule feedback issue](../../issues/new?template=rule-feedback.yml).
- **A checklist question should be added, changed or removed:** open a [checklist issue](../../issues/new?template=checklist-question.yml).
- **An open question or an idea that is not ready yet:** start a [discussion](../../discussions).

Typos and broken links can go straight to a pull request.

## Evidence over opinion

A proposal is strongest when it points at something a reader can check:

- a standard or its exact wording (for example a NIST SP 800-207 tenet),
- a documented incident or failure mode,
- a published design or specification.

"I think" is welcome as a start; a source makes it mergeable.

## Editorial principles

- **Vendor-neutral.** The rules and the checklist name no product, including the maintainer's. Pull requests that promote a product are closed.
- **Plain language.** Short sentences. Say what a person can verify, not what a tool claims.
- **Stay faithful to NIST SP 800-207.** Every rule cites the tenet it applies, quoted verbatim from section 2.1.
- **Small, reviewable changes.** One rule or one question per pull request.
- **British spelling**, matching the existing text.

## Versioning

The specification uses [semantic versioning](https://semver.org/):

- **Major:** a rule is added, removed or changes meaning.
- **Minor:** a checklist question is added or removed, or a rule's explanation is clarified without changing its meaning.
- **Patch:** wording, typos and links.

Every release is recorded in [CHANGELOG.md](CHANGELOG.md) and tagged `vX.Y.Z`, and the version in [CITATION.cff](CITATION.cff) is bumped with it.

## Licence of contributions

By contributing you agree that your contribution is licensed under [CC BY 4.0](LICENSE), the licence of this repository.

## Code of Conduct

Everyone taking part follows the [Code of Conduct](CODE_OF_CONDUCT.md).
