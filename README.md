# MOXY OSS RFCs

This repository is intended to be used for reviewing submitted RFCs (request for comments) related to additions or changes to MOXY OSS projects and processes.

The MOXY OSS RFC process aims to provide a consistent way for MOXY to facilitate the transparency of decisions made for features and changes in OSS related projects and processes.

In short, an MOXY RFC is a proposal for a significant change or a new feature.

The RFC process will:

- Facilitate the transparency of decision making by MOXY to end-users, including MOXY team members themselves and the community.
- Enable individual contributors to help make decisions.
- Allow domain experts to have input in decisions.
- Manage the risk of decisions made.
- Have a snapshot of context for the future.
- Triage future work according to its relevance.

**[See open RFCs](https://github.com/moxystudio/rfcs/pulls)**.

## Process outline

- Copy `0000-template.md` to `rfcs/0000-my-subject.md`, where 'my-subject' is descriptive. Don't assign an RFC number yet.
- Fill in the RFC. Put care into the details: RFCs that do not present convincing motivation, demonstrate understanding of the impact of the design, or are disingenuous about the drawbacks or alternatives tend to be poorly-received.
- Submit a pull request. As a pull request the RFC will receive design feedback from the larger community, and the author should be prepared to revise it in response.
- Build consensus and integrate feedback. RFCs that have broad support are much more likely to make progress than those that don't receive any comments.
- When the author decides that the RFC is finalized, a new final comment period is given (usually 1 week) and the pull request is tagged with `final comments`.
- An RFC may be rejected after public discussion has settled and comments have been made summarizing the rationale for rejection. Once rejected, the RFC's associated pull request shall be closed by a MOXY team member.
- An RFC may be accepted at the close of its final comment period. The RFC's associated pull request will be merged (with squash), at which point the RFC will become part of this repository.

## Reviewing RFCs

Members of the MOXY team will attempt to review some set of open RFC pull requests on a regular basis. If a MOXY team member believes an RFC PR is ready to be accepted, they can approve the PR using GitHub's review feature to signal their approval of the RFC.

When applicable, reviewers should use GitHub's suggestions feature to make it easier to incorporate changes to the document.

## Acceptance or dismissal of RFCs

Ideally, the RFC should gather consensus and culminate in a collaborative decision. However, when a collaborative decision is not possible, people with different stakes have different decision power. As an example, authors and maintainers of the project the RFC relates to have more weight than others. MOXY's CTO and Engineer Leads may also express their sentiment to help make a final decision.

## Details on accepted RFCs

Once an RFC is accepted, the authors may implement it and submit the feature as a pull request to the associated repository. 'Accepted' is not a rubber stamp, and in particular still does not mean the feature will ultimately be merged; it does mean that it has been endorsed in principle and are amenable to merging it.

Furthermore, the fact that a given RFC has been accepted implies nothing about what priority is assigned to its implementation, nor whether anybody is currently working on it.

Modifications to accepted RFC's can be done in followup PR's.

## Credits

MOXY's OSS RFC process owes its inspiration to the [React RFC process](https://github.com/reactjs/rfcs), [Vue RFC process](https://github.com/reactjs/rfcs), [Rust RFC process](https://github.com/rust-lang/rfcs) and [Ember RFC process](https://github.com/emberjs/rfcs).
