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

Databuilder already has the table lineage model, which creates an upstream/downstream relation to adding to the Neo4j graph. 
Column lineage model however still needs to be developed as a part of this RFC.

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

> Why should we _not_ do this?

> Please consider:
> Implementation cost, both in term of code size and complexity
> Integration of this feature with other existing and planned features
> The impact on onboarding and learning about Amundsen
> Cost of migrating existing Amundsen installations (is it a breaking change?)
> If there are tradeoffs to choosing any path. Attempt to identify them here.

## Alternatives

> Why is this design the best in the space of possible designs?
> What other designs have been considered and what is the rationale for not choosing them?
> What is the impact of not doing this?

## Prior art

> Discuss prior art, both the good and the bad, in relation to this proposal. A few examples of what this can include are:

> Does this feature exist in other data search applications and what experience have their community had?
> For community proposals: Is this done by some other community and what were their experiences with it?
> Papers: Are there any published papers or great posts that discuss this? If you have some relevant papers to refer to, this can serve as a more detailed theoretical background.

> This section is intended to encourage you as an author to think about the lessons from other projects, provide readers of your RFC with a fuller picture. If there is no prior art, that is fine - your ideas are interesting to us whether they are brand new or if it is an adaptation from other projects.

## Unresolved questions

> What parts of the design do you expect to resolve through the RFC process before this gets merged?
> What parts of the design do you expect to resolve through the implementation of this feature before stabilization?
> What related issues do you consider out of scope for this RFC that could be addressed in the future independently of the solution that comes out of this RFC?

## Future possibilities

> Think about what the natural extension and evolution of your proposal would be and how it would affect the project as a whole in a holistic way. Also, consider how this all fits into the roadmap for the project and of the relevant sub-team.
> This is also a good place to "dump ideas" if they are out of scope for the RFC you are writing but otherwise related.
> If you have tried and cannot think of any future possibilities, you may simply state that you cannot think of anything.
