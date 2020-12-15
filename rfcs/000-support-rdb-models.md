- Feature Name: Support Relational Database Models
- Start Date: 2020-12-14
- RFC PR: [amundsen-io/rfcs#0000](https://github.com/amundsen-io/rfcs/pull/0000) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Support Relational Database Models

## Summary

In order to use relational DBs(RDB), like MySQL, as the metadata store in Amundsen, we have to support Object-relational Mapping(ORM) first. 
This RFC is mainly about the models used for relational DBs.

## Motivation

Relational DBs, like MySQL, are well deployed and widely used in most workplaces. Using the well-established DB infrastructure would save much cost(e.g., new DB deployment, backup, monitoring, proxy) 
for potential Amundsen users. Hence, it would be very helpful if Amundsen can support relational DBs as the metadata store. 
The expected result is users are able to call the relational DB models to ingest/retrieve metadata in MySQL.

## Guide-level Explanation (aka Product Details)

Instead of duplicating extractors to work with RDB models directly, RDB models are designed to work with current graph db models 
and build each model instance as record with the extracted metadata. These models will be used in databuilder(e.g., ETL and publishing job), and metadataservice.


## UI/UX-level Explanation

N/A

## Reference-level Explanation (aka Technical Details)

Node and some relation labels in graph DB are reflected as the entities and associated entities in RDB models. 
Attributes in graph node and relation is kept as the fields in RDB models. Node `key` is converted to `rk`(record key) to avoid the keyword use in some relational DBs. 
Following are the Model(**Primary Key**, field\_1, field\_2, <ins>Foreign Key</ins>) designed according to the existing graph db nodes and relations.

> Database, Cluster, Schema

- Database(**rk**, name, published\_tag, publisher\_last\_updated\_epoch_ms)
- Cluster(**rk**, name, <ins>database\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- Schema(**rk**, name, <ins>cluster\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- SchemaDescription(**rk**, description\_source, description, <ins>schema\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)

> Table

- Table(**rk**, name, is\_view, <ins>schema\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- TableDescription(**rk**, description\_source, description, <ins>table\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- TableProgrammaticDescription(**rk**, description\_source, description, <ins>table\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- TableTimestamp(**rk**, last\_updated\_timestamp, timestamp, name, <ins>table\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- TableTag(**<ins>table\_rk</ins>**, **<ins>tag\_rk</ins>**, published\_tag, publisher\_last\_updated\_epoch_ms)
- TableBadge(**<ins>table\_rk</ins>**, **<ins>badge\_rk</ins>**, published\_tag, publisher\_last\_updated\_epoch_ms)
- TableOwner(**<ins>table\_rk</ins>**, **<ins>user\_rk</ins>**, published\_tag, publisher\_last\_updated\_epoch_ms)
- TableFollower(**<ins>table\_rk</ins>**, **<ins>user\_rk</ins>**, published\_tag, publisher\_last\_updated\_epoch_ms)
- TableColumnUsage(**<ins>table\_rk</ins>**, **<ins>user\_rk</ins>**, read\_count, published\_tag, publisher\_last\_updated\_epoch_ms)
- Watermark(**rk**, partition\_key, partition\_value, create\_time, <ins>table\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- Source(**rk**, source, source\_type, <ins>table\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)

> Column

- Column(**rk**, name, type, sort\_order, <ins>table\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- ColumnDescription(**rk**, description\_source, description, <ins>column\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- ColumnBadge(**<ins>column\_rk</ins>**, **<ins>badge\_rk</ins>**, published\_tag, publisher\_last\_updated\_epoch_ms)
- Stat(**rk**, stat\_name, stat\_val, start\_epoch, end\_epoch, <ins>column\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)

> Application

- Application(**rk**, application\_url, name, id, description, published\_tag, publisher\_last\_updated\_epoch_ms)
- ApplicationTable(**<ins>application\_rk</ins>**, **<ins>table\_rk</ins>**, published\_tag, publisher\_last\_updated\_epoch_ms)

> Dashboard

- DashboardGroup(**rk**, name, dashboard\_group\_url, <ins>cluster\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- DashboardGroupDescription(**rk**, description, <ins>dashboard\_group\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- Dashboard(**rk**, name, created\_timestamp, dashboard\_url, <ins>dashboard\_group\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- DashboardDescription(**rk**, description, <ins>dashboard\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- DashboardTimestamp(**rk**, timestamp, name, <ins>dashboard\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- DashboardTag(**<ins>dashboard\_rk</ins>**, **<ins>tag\_rk</ins>**, published\_tag, publisher\_last\_updated\_epoch_ms)
- DashboardBadge(**<ins>dashboard\_rk</ins>**, **<ins>badge\_rk</ins>**, published\_tag, publisher\_last\_updated\_epoch_ms)
- DashboardOwner(**<ins>dashboard\_rk</ins>**, **<ins>user\_rk</ins>**, published\_tag, publisher\_last\_updated\_epoch_ms)
- DashboardFollower(**<ins>dashboard\_rk</ins>**, **<ins>user\_rk</ins>**, published\_tag, publisher\_last\_updated\_epoch_ms)
- DashboardUsage(**<ins>dashboard\_rk</ins>**, **<ins>user\_rk</ins>**, read\_count, published\_tag, publisher\_last\_updated\_epoch_ms)
- DashboardTable(**<ins>dashboard_rk</ins>**, **<ins>table_rk</ins>**)
- Chart(**rk**, id, name, type, url, <ins>query\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- Query(**rk**, id, name, url, query\_text, <ins>dashboard\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)
- Execution(**rk**, timestamp, state, <ins>dashboard\_rk</ins>, published\_tag, publisher\_last\_updated\_epoch_ms)

> Tag, Badge

- Tag(**rk**, tag\_type, published\_tag, publisher\_last\_updated\_epoch_ms)
- Badge(**rk**, badge\_category, published\_tag, publisher\_last\_updated\_epoch_ms)

> User

- User(**rk**, email, is\_active, first\_name, last\_name, full\_name, github\_username, team\_name, employee\_type, slack\_id, role\_name, updated\_at, manager\_rk, published\_tag, publisher\_last\_updated\_epoch_ms)

> Amundsen updated timestamp

- UpdatedTimestamp(**rk**, last\_timestamp, published\_tag, publisher\_last\_updated\_epoch_ms)

## Drawbacks

1. Updating model schema in relational DBs is not as flexible as in schemaless NoSQL DBs. 
We need upgrade model version when there is a definition change on RDB models.

2. For any change in graph db models, we need remember to update RDB models accordingly to make them compatible with graph db models.

## Alternatives

1. Develop standalone RDB models which are not derived from existing graph db models and work with extractors directly. 
This alternative would make the implementation complex and needs duplicate all the current extractors, and in which case,
we need not only make model changes but also extractor changes compatible for graph DBs and relational DBs.

2. For some graph db model labels, like tag, description, each of them can be a single RDB model with a generic reference field(e.g., 'item_rk') 
instead of being divided into multiple {entity}Tag/Description, but the single RDB model can not have foreign key constraint for that field. 

## Prior art

N/A

## Unresolved questions
None

## Future possibilities
To make MySQL as the metadata backend store, we will have to support other modules(e.g., data loader, publisher) in databuilder.