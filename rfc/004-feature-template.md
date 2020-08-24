- Feature Name: badges_node
- Start Date: 2020-08-24
- RFC PR: [amundsen-io/rfcs#4](https://github.com/amundsen-io/rfcs/pull/4)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000)

# Separate badges from tags

## Summary

Create a new node type for neo4j which would represent Badges that is separate from the Tag node.

## Motivation

In order to best use badges we want to add sentiment and category fields to the Badge nodes. This makes the Tag and Badge components different, as Badge requires more information now. Separating the two now will likely lead to less tech debt in the future when other features are implemented that impact the way we ingest and use badges. Additionally, this change will help clarify the difference in use and purpose between tags and badges throughtout Amundsen. This feature will also guarantee badges are not ingested without having sentiment and category.


## Guide-level Explanation (aka Product Details)

> Explain the proposal as if it was already included in Amundsen and you were teaching it to an Amundsen user. That generally means:

> Introducing new named concepts.
> Explaining the feature largely in terms of examples.
> Explaining how Amundsen users should think about the feature, and how it should impact the way they use Amundsen. It should explain the impact as concretely as possible.
> If applicable, provide deprecation warnings, or migration guidance.
> For implementation-oriented RFCs, this section should focus on how maintainers should think about the change, and give examples of its concrete impact. For policy RFCs, this section should provide an example-driven introduction to the policy, and explain its impact in concrete terms.
- In the future this feature will hopefully replace all use of old Tag badges.

## Reference-level Explanation (aka Technical Details)

The implementation will require:
- Add methods to the metadata service API to add, remove and, get badges as currently the service uses the methods for tags.
- Update badge ingestion docs to reflect new API call and parameters.
- Make updates to search, databuilder and frontend service to add functionality to use Badge instead of Tag.
- Handle old badges:
    - Keep the existing functionality and add new functionality for surfacing new badge nodes, can eventually remove old functionality
    - Migration code to move all Tags with type “badge” to Badge nodes with default ‘sentiment’ and ‘category’ values of ‘Neutral’ and ‘Table status’ respectively. (This might only be possible down the line when we deprecate old functionality)


## Drawbacks

- Takes more effort and time to implement than the alternative of keeping them together.
- Duplicate code for some functionality between BadgeAPI and TagAPI.

## Alternatives

- Keep the current functionality and simply add two optional parameters fo sentiment and category to Tag.

## Unresolved questions

- Do we want to constraint the input space for sentiment and category to be an enum to reduce room for error rather than being a string?

## Future possibilities

- Deprecate any functionality relying on the prior implementation of badges.