- Feature Name: first_class_queries
- Start Date: 2021-03-18
- RFC PR: [amundsen-io/rfcs#0000](https://github.com/amundsen-io/rfcs/pull/27)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/27)

# First Class Queries

## Summary

This proposes to ingest user-run queries directly into Amundsen. The information included would be the raw string query (typically SQL), as well as the objects referenced by it (Columns, Tables, etc). Amundsen would not parse SQL natively, rather it would provide the plumbing for users who have SQL parsing capabilities.

## Motivation

This allows for considerably more dynamism in how Amundsen surfaces usage-based information. Rather than popular tables being determined primarily by querying usage statistics from dashboarding systems, Amundsen could determine it at runtime by querying the metastore. Same goes for frequent table users. It also lays the groundwork for features like: commonly joined tables, popular queries, audit logging, etc. Additionally, if DDL/DML changes are observed in the log, they can be associated with a table.

## Guide-level Explanation (aka Product Details)

In Amundsen, query logs from your DWH, RDBMS or BI tool may be ingested raw. It's up to you to extract the information about which subjects a particular query touches (e.g. which tables it queries, which user ran it, etc), but once ingested, Amundsen can surface interesting insights about query patterns.

## UI/UX-level Explanation

This change does not necessarily entail UX changes. Most simply, we could surface all queries run against a table (similarly to how dashboard queries are currently surfaced). Mostly, though, this underpins future features, as mentioned in Motivation.

## Reference-level Explanation (aka Technical Details)

### Existing functionality

This feature is intended to be implemented entirely separately from existing storage of usage patterns, at least initially. Once we get the implementation in place, we may choose to start using it as the normalized source of truth for more usage information.

This would interact with `DashboardQuery`, in that this would be a superset of that functionality. I suggest implementing this first, and transitioning existing `DashboardQuery` code to use this more generic functionality.

Eventually, this could subsume the denormalized form `TableColumnUsage` for installs that have full data available. It is not likely we would want to deprecate `TableColumnUsage` to allow for different types of integrations. It's possible we would want to pre-generate `TableColumnUsage` from the underlying data, both for compatibility and read-time performance reasons.

### databuilder

We would create a new `class Query(GraphSerializable)` . It would contain these properties:

- Time that the query was run (required)
- At least one of required:
    - Raw string representation of the query (for query types like SQL)
    - Name of job run (for things like Python notebooks run)
- Optional links to other data:
    - User who ran the query
    - Table(s)
        - This relationship optionally includes info like:
            - Relationship type: query from, join?
                - For joins, include the join condition.
    - Column(s) (if given, Tables may possibly be inferred from this)
        - Optionally including info like WHERE clause conditions

This would emit to neo4j in the shape you'd expect: a new node for queries, and relationships to existing stuff.

Given that writing the SQL parsing logic is out-of-scope for this RFC, an `Extractor` is out-of-scope.

## Drawbacks

This is purely additive right now, so there are few downsides.

Considering the future where this is used more heavily to support other features, however: it is possible that query volume would be extremely high, and thus storing in neo4j would be impractical. Pruning old queries would now change the semantics of certain queries: e.g. instead of popular tables being all-time popular, they would only be popular over the expiry period.

## Alternatives

The primary alternative is to continue relying on other systems to aggregate usage information, and ingesting into Amundsen aggregates of such information, rather than the source.

## Prior art

- Issue mentioned by jonhehir in [https://github.com/amundsen-io/amundsen/issues/547](https://github.com/amundsen-io/amundsen/issues/547)

## Unresolved questions

- Is there other prior art or considerations about this? Surely it's been considered before.
- For queries that have layers of subselects, should we emit a single Query, or multiple Query?
- How's the read-time performance of this for the eventual features built on top of it? If we ingest 1MM queries, how fast would a "frequent users" query for a specific table be?

## Future possibilities

There are myriad ways to surface this information in smart ways in the UI, briefly mentioned in Motivation.

For derived tables, it would be reasonable to connect this to the nascent Lineage features. A `INSERT INTO destination_table SELECT FROM source_table` query could be used as a connection represent direct table-to-table lineage. However, because the concept of lineage is broader than just SQL queries, it's not suggested that this be the primary representation of such state.

We could provide SQL parsing a databuilder `Extractor` to make it easy for users to latch onto existing query logs.
