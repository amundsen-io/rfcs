- Feature Name: badges_node
- Start Date: 2020-08-24
- RFC PR: [amundsen-io/rfcs#4](https://github.com/amundsen-io/rfcs/pull/4)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000)

# Separate badges from tags

## Summary

There is a need to create a new entity for Badges that is separate from Tag in the metadata service. Badges in Amundsen are components which in the past have been used to indicate different bits of metadata like explaining if a resource was a table view or if a user was no longer active in amundsen (alumni). Tags were also simple bits of information consisting of a name that were used to filter resources when searching, so it made sense to combine them at the time. 

Moving forward we want to use badges to indicate the status of a table resource (alpha, beta GA, deprecated), whether we recommend using a certain resource, as well as using them to indicate if a column in a table is a primary key or partition column.

## Motivation

In order to best use badges we want to add sentiment and category fields to the entity to help decide things like the presentation of the badge (color, placement) and how to prioritize resources based on what badges they have. This makes the Tag and Badge entities different, as Badge requires more information now and will be used in a very different way from Tag. Separating the two now will likely lead to less tech debt in the future when other features are implemented that impact the way we ingest and use badges. Additionally, this change will help clarify the difference in use and purpose between tags and badges throughtout Amundsen. This feature will also guarantee badges are not ingested without having sentiment and category.


## Guide-level Explanation (aka Product Details)

A badge is a predetermined indicator which can be added by a table owner in order to offer information about the status of a table resource (alpha, beta GA, deprecated), whether they recommend using a certain resource, or to indicate if a column in a table is a primary key or partition column. 
Using badges to boost results in searches and to filter resources when searching will help Amundsen users find the resources they need easier, as well as help them discern which resources are most trustworthy.

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
- Do we plan to add more attributes for badge later (besides sentiment and category)? 

## Future possibilities

- Deprecate any functionality relying on the prior implementation of badges.
