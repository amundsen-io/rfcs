- Feature Name: configurable_elasticsearch_queries
- Start Date: 2020-11-04
- RFC PR: [amundsen-io/rfcs#0011](https://github.com/amundsen-io/rfcs/pull/11) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Configurable Elasticsearch Queries

## Summary

This RFC brings an idea to make elasticsearch queries something that is more friendly to parametrize.

## Motivation

Currently, if Amundsen users want to use custom search queries they need to overwrite ElasticsearchProxy and reimplement search methods.

It might be useful to enable easy configuration option so users can try out different search patterns, either refining what upstream proposes or 
simply more relevant to specific Amundsen deployment environment / search scenarios.

## Guide-level Explanation (aka Product Details)

* There is a configuration option to define custom Elasticsearch query for each resource type
* User can define custom query as path to json file
* User can define custom query as Python dict
* User can define custom query as Elasticsearch Search Template
* Users can more easily inject different search queries to experiment with and refine relevancy of search in their installations

## UI/UX-level Explanation

N/A

## Reference-level Explanation (aka Technical Details)

* There are `ELASTICSEARCH_SEARCH_QUERIES_${RESOURCE_TYPE}` configuration options:
```python
class Config:
    ELASTICSEARCH_SEARCH_QUERIES_TABLE = './elasticsearch/table_search_query.json'
    ELASTICSEARCH_SEARCH_QUERIES_USER = './elasticsearch/user_search_query.json'
    ELASTICSEARCH_SEARCH_QUERIES_DASHBOARD = './elasticsearch/dashboard_search_query.json'
```
* Default queries are stored in json files
* Overriding search query is as simple as doing:
```python
class MyCustomConfig(Config): 
    ELASTICSEARCH_SEARCH_QUERIES_TABLE = '/custom_queries/my_query.json'
```
(Custom queries can be for )
* There is a support to define query as dict instead of json file:
```python
class MyCustomConfig(Config): 
    ELASTICSEARCH_SEARCH_QUERIES_USER = {'query': '*'}
```
* If custom query is not meeting certain requirements, Amundsen fallbacks to default one

## Drawbacks

* This is an advanced feature, might not be widely used
* Having this feature doesn't introduce complexity in onboarding new users - if improving search is not ones desire, they simply do not use it

## Alternatives

* Only alternative to achieve this for now is to subclass Elasticsearch Proxy and reimplement whole search methods.

## Prior art

> Discuss prior art, both the good and the bad, in relation to this proposal. A few examples of what this can include are:

> Does this feature exist in other data search applications and what experience have their community had?
> For community proposals: Is this done by some other community and what were their experiences with it?
> Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

> This section is intended to encourage you as an author to think about the lessons from other projects, provide readers of your RFC with a fuller picture. If there is no prior art, that is fine - your ideas are interesting to us whether they are brand new or if it is an adaptation from other projects.

## Unresolved questions

* Do we want support for Elasticsearch Search Templates ?

## Future possibilities

> Think about what the natural extension and evolution of your proposal would be and how it would affect the project as a whole in a holistic way. Also consider how the this all fits into the roadmap for the project and of the relevant sub-team.
> This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related.
> If you have tried and cannot think of any future possibilities, you may simply state that you cannot think of anything.
