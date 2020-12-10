- Feature Name: merge_metadata_search_repos
- Start Date: 2020-12-10
- RFC PR: [amundsen-io/rfcs#0000](https://github.com/amundsen-io/rfcs/pull/0000) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Merge Amundsen Metadata and Search Repository

## Summary

Idea is to deprecate the split of metadata and search repositories, and merge them into a single `amundsenbackend` repository.

## Motivation

The actual split is already on the backend we choose for each of the metadata and search service i.e., Neo4j/Atlas/Gremlin and ES/Atlas respectively. 
Both the repositories are simply a thin proxy connecting endpoints to these backends. Hence need more maintenance, multiple PRs to implement a feature, and a need to use a third repo i.e., `amundsencommon`.  

- This will decrease the development efforts, and the tooling cost. One less component to maintain. 
- Shared helper functions, exceptions handling, help remove code duplication
- Configuration is pretty much the same for both repositories, except for the proxy backends. 

## Transition Path
TBD

> This is the bulk of the RFC. Explain the use-cases that deprecated functionality covers, and for each use-case, describe the transition path.
> Describe it in enough detail for someone who uses the deprecated functionality to understand, for someone to write the deprecation guide, and for someone familiar with the implementation to implement.

## How We Communicate This
TBD
> Would the acceptance of this proposal mean the Amundsen documentation must be re-organized or altered?
> Does it change how Amundsen user's are onboarded? Does it mean we need to put effort into highlighting the replacement functionality more?
> How should this deprecation be introduced and explained to existing Amundsen users?

## Drawbacks
TBD
> Why should we _not_ do this?

> Please consider:
> The impact on onboarding and learning about Amundsen
> Integration of this feature with other existing and planned features
> Deprecation cost, both in term of code size and complexity
> If there are tradeoffs to choosing any path. Attempt to identify them here.

## Alternatives
TBD
> What other paths have been considered and what is the rationale for not choosing them?
> What is the impact of not doing this?

## Unresolved questions
TBD
> What parts of this RFC are TBD?

## Future possibilities
TBD
> What kind of possibilities does this deprecation open for the project?
