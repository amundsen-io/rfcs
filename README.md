# Amundsen RFCs

We can handle most of the issues we see with regular GitHub issues. However, some changes are "substantial", and we ask that these go through a design process and produce a consensus among the Amundsen community and the [sub-teams](sub-teams).

The "RFC" (request for comments) process is intended to provide a consistent and controlled path for new features to enter the [roadmap](roadmap).

The Amundsen team is still new to this process, and we are actively developing it. We ask for feedback from the community to improve this approach, so don't hesitate to give us your feedback!

## Why?

The RFC process is an excellent opportunity to get more eyeballs on your proposal before it becomes a part of a released version of Amundsen. Quite often, even proposals that seem "obvious" can be significantly improved once a wider group of interested people have a chance to weigh in.

The RFC process can also be helpful to encourage discussions about a proposed feature as it is being designed. It can also help incorporate significant constraints into the design while it's easier to change before it has been fully implemented.

## When to Use RFCs

What constitutes a "substantial" change is evolving based on the community, but may include the following:

- New features that require configuration options to activate/deactivate
- Remove features
- Architectural changes
- Here are some late examples:
  - Adding lineage features
  - Dashboards integration

Some changes do not require an RFC:

- Reorganizing or refactoring code or documentation
- Improvements that tackle objective quality criteria (speedup, better browser support)
- Changes noticeable only by contributors or maintainers
- Recent examples are:
  - Adding programmatic descriptions
  - Adding support for badges at a column level

If you submit a pull request to implement a new feature without going through the RFC process, it may be closed with a polite request to send an RFC first. That said, if most of the work is done, we will accelerate the process.

## Before creating an RFC

First, it is wonderful that you are considering suggesting new features or changes to Amundsen - we appreciate your time and willingness to contribute!

We want contributors to have a great experience helping at Amundsen. However, a hastily-proposed RFC can hurt its chances of acceptance. Low-quality proposals, proposals for previously-rejected features, or those that don't fit into the near-term roadmap, may be quickly rejected, which can demotivate the unprepared contributor.

Laying some groundwork ahead of the RFC can make the process smoother. There is no single way to prepare for submitting an RFC. Still, it is generally good to pursue feedback from other project developers beforehand, to ascertain that the RFC may be desirable.

The most common preparation for writing and submitting an RFC includes talking the idea over on our official Slack channels, discussing the topic on our #amundsen-rfcs channel, and occasionally posting "RFC Drafts" on that channel. You may also open an issue on this repo to start a high-level discussion in the shape of an RFC Draft, to eventually formulate an RFC pull request with the specific implementation design.

As a rule of thumb, receiving encouraging feedback from Amundsen maintainers indicates that the RFC is worth pursuing.

## The Process

In short, to get a major feature added to Amundsen, one must first get the RFC merged into the RFC repository as a markdown file. At that point, the RFC is "active" and may be implemented with the goal of eventual inclusion into Amundsen.

1. Fork the RFC repo https://github.com/amundsen-io/rfcs
1. Choose the appropriate RFC template. For most RFCs, this is `templates/0000-feature-template.md`, for deprecation RFCs it is `templates/000-deprecation-template.md`.
1. Copy the appropriate template file to `rfcs/000-<rfcName>.md`. Don't assign an RFC number yet, as this is going to be the PR number, and we'll rename the file accordingly if the RFC is accepted.
1. Fill in the RFC. Put care into the details: RFCs that do not present convincing motivation, demonstrate a lack of understanding of the design's impact, or are disingenuous about the drawbacks or alternatives tend to be poorly-received.
1. Submit a pull request. As a pull request, the RFC will receive design feedback from the larger community, and the author should be prepared to revise it in response.
1. An RFC Champion will be assigned to the PR, who will be the owner from the maintainers' side, responsible for representing it in the maintainer's team meetings and community meetings.
1. Build consensus and integrate feedback, especially among the [sub-team](sub-teams) responsible. RFCs with broad support are much more likely to make progress than those that don't receive any comments.
1. RFCs rarely go through this process unchanged, especially as alternatives and drawbacks are shown. You can make edits, big and small, to the RFC to clarify or change the design, but make changes as new commits to the pull request, and leave a comment on the pull request explaining your changes. Specifically, do not squash or rebase commits after they are visible on the pull request.
1. RFCs that are candidates for inclusion in Amundsen will enter a **"Final Comment Period (FCP)"**, along with a disposition for the RFC (merge, close, or postpone).
   - This step is taken **when enough of the tradeoffs have been discussed** so that the maintainers are in a position to make a decision. That does not require consensus amongst all participants in the RFC thread. However, the argument supporting the disposition on the RFC needs to have already been clearly articulated. There should not be a strong consensus against that position outside of the maintainer's group.
   - The FCP period lasts **seven days**, and the beginning of this period will be signaled with a comment and tag on the RFC's pull request.
   - For RFCs with lengthy discussion, the motion to FCP is usually preceded by a summary comment trying to lay out the current state of the discussion and major tradeoffs/points of disagreement.
   - Maintainers will ping the `#amundsen` channel about the RFC to attract the community's attention.
   - An RFC can be modified based upon feedback from the team and community. Significant modifications may trigger a new final comment period.
   - RFC must be approved by at least one maintainer, with no -1 on it, in order to start working on it.
   - If consensus is not reached through discussion on an RFC, then we must gain maintainers' approval in the form of a vote, with a voting time of 48 hours. Min required votes for Approval/Rejection of an RFC will be 80% of the total votes.
1. An RFC may be rejected by the team after the public discussion has been settled and comments have been made to summarize the rationale for rejection. The RFC Champion should then close the RFCs associated pull request. Also, an RFC author may withdraw their RFC by closing it themselves.
1. An RFC may be accepted at the close of its final comment period. The RFC Champion will merge the RFCs associated pull request, at which point the RFC will become 'active'.
1. Maintainers will meet every six months and choose three or five active RFCs based on popularity and alignment with project vision and goals. Those selected items become part of the Mid-term goals on our roadmap.

## The RFC life-cycle

An RFC goes through the following stages:

- **Draft**: when a Draft RFC is created for sharing purposes.
- **Pending**: when the RFC is submitted as a PR.
- **Active**: when an RFC PR is merged or undergoing implementation.
- **Landed**: when an RFC's proposed changes are shipped in an actual release.
- **Rejected**: when an RFC PR is closed without being merged.

### Active RFCs

The fact that a given RFC has been accepted and is 'active' implies nothing about what priority is assigned to its implementation, nor whether anybody is currently working on it. Some accepted RFCs represent vital features that need to be implemented right away.

Other accepted RFCs can describe features that can wait until some arbitrary developer feels like doing the work. Active RFCs imply that they will be considered to be included in the official roadmap on the next roadmap creation meeting.

Becoming 'active' is not a rubber stamp. In in particular, it still does not mean the feature will ultimately be merged; it means that the maintainers have agreed to it in principle and are amenable to merging it.

Modifications to active RFCs can be done in followup PRs. We strive to write each RFC in a manner that it will reflect the final design of the feature, but the nature of the process means that we cannot expect every merged RFC to actually reflect what the end result will be at the time of the next major release. Therefore, we try to keep each RFC document somewhat in sync with the project feature as planned, tracking such changes via followup pull requests to the document.

## Implementing RFCs

The author of an RFC is not obligated to implement it. Of course, the RFC author (like any other developer) is welcome to post an implementation for review after the RFC has been accepted.

If you are interested in working on the implementation for an "active" RFC but cannot determine if someone else is already working on it, feel free to ask (e.g., by leaving a comment on the associated issue).

## Reviewing RFCs (\*Meant to go in the Maintaining.md file)

Each maintainer is responsible for regularly reviewing open RFCs, especially if the RFC covers any of his/her particular specialties. While the RFC pull request is up, the maintainers may schedule meetings with the author and/or relevant stakeholders to discuss the issues in greater detail, and in some cases, the topic may be addressed in a team meeting. In either case, a summary from the meeting will be posted back to the RFC pull request.

Maintainers must ensure that each RFC addresses any consequences, changes, or work required for the project. If a maintainer believes an RFC PR is ready to be accepted into active status, they can approve the PR using GitHub's review feature to signal the RFC's approval.

**Amundsen's RFC process owes its inspiration to the [Rust RFC process], [Ember RFC process], [React RFC process], and [Vue RFC process]**.

[react rfc process]: https://github.com/reactjs/rfcs
[rust rfc process]: https://github.com/rust-lang/rfcs
[ember rfc process]: https://github.com/emberjs/rfcs
[vue rfc process]: https://github.com/vuejs/rfcs/blob/master/README.md
[roadmap]: https://github.com/amundsen-io/amundsen/blob/master/docs/roadmap.md
[sub-teams]: https://github.com/orgs/amundsen-io/teams

---
