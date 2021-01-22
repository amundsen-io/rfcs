- Feature Name: Add abstract serializable for relational databases in databuilder models
- Start Date: 2021-01-21
- RFC PR: [amundsen-io/rfcs#0000](https://github.com/amundsen-io/rfcs/pull/0000) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Add abstract serializable for relational databases in databuilder models

## Summary

The new abstract serializable in databuilder models will work for the metadata injection to relational databases. This RFC would 
bring two major changes. 

- Add a new abstraction layer working as the serializable to generate next record. 
- Add record iterator and its function for next record instance working with [Amundsenrds](https://github.com/amundsen-io/amundsenrds) in applicable [databuilder models](https://github.com/amundsen-io/amundsendatabuilder/tree/master/databuilder/models). 

## Motivation

Currently, the databuilder models are working for graph db with node/relation iterator. In order to support relational database as backend store
in Amundsen, this RFC aims to make databuilder model as a centralized model which will not only work for graph db but also for relational databases.
Amundsenrds contains the ORM models for relational db use, and the new abstract  relational DB serializable class would be inherited by applicable databuilder models 
to build record iterator and yield record instance with Amundsenrds.

## Guide-level Explanation (aka Product Details)

The new abstraction layer `TableSerializable` will work similarly to the current `GraphSerializable` and will contain abstract method for applicable databuilder models to implement method for record iterator.
Then the record object from `TableSerializable` can be extracted in the future relational DB data loader.

## UI/UX-level Explanation

N/A

## Reference-level Explanation (aka Technical Details)

(1). `TableSerializable`: It mainly contains method `create_next_record()` and `next_record()`

(2). `record_iterator`: For the applicable databuilder models, they will inherit `TableSeriablizable` as well as the existing `GraphSerializable`.
We will treat the current databuilder model as a centralized model with a `record_iterator` in its initialization function. Also, the next method 
for `record_iterator` will yield record instance by calling models in Amundsenrds.
For example(a partial change in `_create_next_record()` of [TableMetadata](https://github.com/amundsen-io/amundsendatabuilder/blob/master/databuilder/models/table_metadata.py)):

```python
def _create_next_record(self):
    """ omitted code snippet yielding Database, Cluster, Schema record instances
    """

    # Table
    yield RDBModels.Table(
        rk=self._get_table_key(),
        name=self.name,
        is_view=self.is_view,
        schema_rk=self._get_schema_key()
    )

    """omitted code snippet yielding the rest
    """
``` 
Note: 
Considering the foreign key constraints among ORM models, we would have to yield record instance in the topological order.
(e.g, in TableMetadata, yield metadata in the order of  Database -> Cluster -> Schema -> Table -> Column)

## Drawbacks

For some property change, like the data type change, in databuilder model, we have to take potential schema update in 
Amundsenrds into consideration. The schema migration will rely on Alembic model upgrade/downgrade.

## Alternatives

Instead of adding a new abstract TableSerializable, update current GraphSerializable as a general serializable containing 
node/relation/record method definition. The concern is that some databuilder models are not applicable for relational DB, like 
[Neo4jESLastUpdated](https://github.com/amundsen-io/amundsendatabuilder/blob/master/databuilder/models/neo4j_es_last_updated.py) and 
for future models that will be only pushed to graph db, adding dummy record iterator in these models would be redundant. 

## Prior art

N/A

## Unresolved questions

N/A

## Future possibilities

With the new abstraction layer, serializable, in databuilder models, we will add new serializer, data loader and publisher 
to support a specific relational database, like MySQL, as the metadata store.
