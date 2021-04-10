- Feature Name: lineage-backend
- Start Date: 2021-03-31
- RFC PR: [amundsen-io/rfcs#32](https://github.com/amundsen-io/rfcs/pull/32)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Amundsen Lineage - Backend

## Summary

Currently, we link to the 3rd Party lineage tool in order to show table and column lineage for tables in Amundsen.
The goal of this project is to have a lineage graph that is native to Amundsen so that users can find table and column lineage in one platform.

## Motivation

At the moment we are relying completely on any 3rd party lineage tool to show the table and column lineage in Amundsen.
Not everyone has access to such a 3rd party tool and want to have lineage information available natively in Amundsen.

ref: https://github.com/amundsen-io/rfcs/blob/master/rfcs/025-lineage-stage-0.md

## Guide-level Explanation (aka Product Details)

No new concepts/definitions are being introduced as a part of this RFC.

This implementation will only cover Neo4j. Please see Future Work for what was descoped.

Databuilder already has the table lineage model, which creates an upstream/downstream relation to adding to the Neo4j graph.
Column lineage model however still needs to be developed as a part of this RFC.

More specifically, most of the existing `TableLineage` model will be extracted into `BaseLineage`, from which `TableLineage` and the new `ColumnLineage` will inherit from. These will use `uri`s instead of properties as identifiers and constructors, but otherwise will largely be the same as the current `TableLineage`.

This RFC does not specify databuilder extractors at the moment. Please see Future Work.

Important to note: We will not deprecate the ability to show lineage graph/information from 3rd party lineage tools
(which is implemented in Stage 0), hence this implementation will be compatible with the existing functionality.


## UI/UX-level Explanation

No UI changes required as a part of this RFC.
The implementation should work and support the existing UI, and will not break any of the existing UI.

## Reference-level Explanation (aka Technical Details)

Metadata service will have the required endpoints (identical to what we have today for the 3rd party lineage tools).
For Table:
- https://amundsenmetadata.com/table/current_table_key
- https://amundsenmetadata.com/table/current_table_key/lineage?direction=upstream

and the response will look something like below:

```json
{“table_key”: “current_table_key”,“upstream_tables”: [{“table_key”: “table_key1”,“job_status”: “failed”,"last_updated_timestamp": 1604664969,“children_count”: 1,“parents_count”: 3,},],“downstream_tables”: [{“table_key”: “table_key2”,“job_status”: “running”,"last_updated_timestamp": 1602364761,“children_count”: 1,“parents_count”: 3,},],}
```

For Column:
- https://amundsenmetadata.com/table/current_table_key/column/column_name/lineage

and response:

```json
{“column_key”: “current_column_key”,“column_description”: “this is a column”,“column_query”: “SELECT * FROM current_table_name”"stats": [{"end_epoch": 1607126400,"stat_type": "column_usage","start_epoch": 1604534400,"stat_val": "1284"}],“upstream_columns”: [{“column_key”: “column_key1”,"last_updated_timestamp": 1604664969,“children_count”: 5,“parents_count”: 1,},],“downstream_columns”: [{“column_key”: “column_key2”,"last_updated_timestamp": 1602364761,“children_count”: 7,“parents_count”: 1,},],}
```


## Drawbacks

n/a -- These sections have generally been answered by the prior large Lineage RFC, but please comment for more details.

## Alternatives

n/a

## Prior art

n/a

## Unresolved questions

n/a

## Future possibilities

We expect that adding Neptune support for lineage would be quite easy based on this implementation.

Adding RDBMS support would be trickier but possible, but haven't considered the details.

Atlas is currently out of scope as well, but should be possible to support. At present, the Atlas lineage UI is superior to Amundsen's, so the benefit in supporting it is relatively small. However, once we have a full graphical UI, it would be sensible to add metadata proxy support.

We intend to add easy-to-use extractors for dbt and BigQuery, since those platforms provide easy-to-access lineage information.
