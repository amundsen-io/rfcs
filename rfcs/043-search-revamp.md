- Feature Name: Search Revamp
- Start Date: 2021-09-20
- RFC PR: todo

# Search Revamp

## Summary

Revamp search with a cleaner backend implementation, more robust queries, better customizability, and
making use of modern v7 ES features.  

## Motivation

At Lyft, search enhancement are the #1 most frequent Amundsen feedback. This is no surprise, as the current 
technical state of search has not changed much from MVP.

We see search as serving two distinct (but related) use cases:
  - known-data searches, which have a starting point (e.g. tables with a `user_id` column in the `production` schema) 
    and are more interested in listing the available data matching a known condition
  - discovery searches, which are looking to find previously-unknown data that would be relevant for solving
    one's problem

Although we believe Amundsen has been particularly lacking in the discovery search category, our proposal is seeking
to improve the experience of both search types. 


## UI/UX Details

Although the bulk of proposed changes will be to search result quality and relevance, we're also proposing
a handful of UI/UX changes to filters. We're currently working on designs and will share those as soon as they're
available.

#### Multiple Values Per Filter.
Allow multiple values (likely comma-separated) for filters, which has been very highly requested. 

There is an obvious question of whether multiple values would be AND'ed or OR'ed together. Our goal is to provide
additional expressiveness without going down the path of supporting full logical expressions. To better understand the problem,
keep in mind that there's two kind of fields: single-valued fields (table name, schema name) and multi-valued fields (columns, tags, badges).
Our proposed tradeoff between expressiveness and simplicity is to combine filter values for single-valued 
fields with OR (e.g. "staging,production" filter applied to the "schema" field)
whereas for multi-valued fields we'd give users an options to choose AND or OR 
(a user would choose AND/OR to apply the filters "product_id,user_id" to the "column" field)

#### Negation filtering
Another highly requested feature is specifying negative filters - e.g. "schema is not temp, tmp, or temporary".
As above, we want to avoid going down the rabbit hole of full-blown logical expressions. Likely, we'd implement
this on the frontend as a separate set of filter boxes below the current set. Stay tuned for designs. 

#### Type-ahead suggestions for filters
Because values entered as filters are not interpreted or analyzed (this is the current behavior and is not expected to change),
it can be hard for users to remember/guess the exact values to filter for. We'd like to provide typeahead
suggestions that will suggest the most likely filter value given a prefix.


## Backend Changes
As part of this work, most of the elasticsearch [proxy](https://github.com/amundsen-io/amundsen/blob/main/search/search_service/proxy/elasticsearch.py)
would be re-written, while keeping the API of the search service the same (in part to allow Atlas to stay unchanged).

#### Upgrade Elasticsearch to v7
Elasticsearch queries/mappings are generally written to a specific version, since a number of features are typically
added and removed in each new release. We suggest moving to v7 as part of this revamp. How to gracefully handle 
the backwards incompatibility for folks not ready to adopt a new ES version is an open question. 
One option is to keep the existing ES proxy for v6 uses and add all our improvements in new code.

#### Filtered vs unfiltered search unification
Filtered search was added on top of regular search and was implemented separately via separate endpoints. 
It also uses a different (and incorrect) elasticsearch [query_string](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html) command. 
We plan to unify the implementations, so both use the same top-level elasticsearch command either with or without an additional filter specifier. 

#### Replace JSON in queries/mappings with Elasticsearch library objects
The requests made to Elasticsearch are constructed with either hardcoded (in the case of mappings) or templated JSON. 
This makes it more difficult to compose and understand requests, 
but more importantly it makes it very difficult to extend OSS code. 
The Elasticsearch-dsl library includes classes for constructing requests by putting together native python objects.

## Ranking/relevance changes
The bulk of improvements we want to make are around improving how we use Elasticsearch to obtain higher quality results. 

#### Combination of queries
The main insight is that no unary query can sufficiently express all the ways a query text matches a document.

These include (at least):
- exact matches
- exact matches with spelling mistakes (e.g. "ordered_iems" instead of "ordered_items")
- analyzed matches where a user inputs keywords (e.g. "returns") and looks to find the most relevant entities

Elasticsearch uses [compound queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/compound-queries.html) to combine multiple subqueries together. The most common is  [Boolean](https://www.elastic.co/guide/en/elasticsearch/reference/master/query-dsl-bool-query.html).

#### Query Optimizations
To be effective, the union of queries discussed above still needs to be composed of high quality subqueries. 
Here, we’ll list a handful of optimization directions and parameters we’ll experiment with:

##### Fuzziness
Allows for misspellings by specifying a maximum edit distance. See [docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-fuzzy-query.html).

##### Synonyms 
This will require tuning and validation, but some users may get significant mileage from a hand-generated synonyms dictionary file. 
We could provide a facility for specifying such a file. [Docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-graph-tokenfilter.html).

##### Match Phrase Query
In case someone searches for a phrase out of an entity description, this subquery will take word order into consideration. [Docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html).

##### Index Fields in More Ways
Experimenting with multiple analyzers (i.e. independently analyzing an input field via multiple analyzers) has potential benefits like normalizing stemmings and filtering stop words. 
See [multi-field](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html#multi-fields) about how to index a field in multiple ways, [english](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lang-analyzer.html#english-analyzer) analyzer for stemmings and stop word removal.
Amundsen already does this to index some fields both as an unanalyzed `keyword` and as analyzed `text`.


## Drawbacks and Alternatives
We believe our proposal will be a significant leap in search quality that will broadly benefit all Amundsen users. Along
with improving out-of-the-box ranking and relevance, we plan to make more parameters tunable by end users.

The main drawback is around the proposed migration to ES 7, which is discussed below.

## Unresolved questions
A full ES7 migration may be difficult for some users. One option is to keep the existing ES proxy for legacy use, but
this is complicated by needing to upgrade the Elasticsearch library in the search service to v7+. 
Since it is recommended to use the same version of the ES lib as the server version, making v6 compatible requests
with the v7 library may run into unexpected corner cases. 

At the same time, v7 has too many new features to pass up and revamping search against the older v6 version is likely
suboptimal.

 