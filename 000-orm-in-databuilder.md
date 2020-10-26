- Feature Name: Support ORM in databuilder
- Start Date: 2020-10-26
- RFC PR: [amundsen-io/rfcs#0000](https://github.com/amundsen-io/rfcs/pull/0000) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# <RFC title>

## Summary

Currently, Amundsen has a great support on graph DB as backend store. However, in order to use MySQL as the metadata 
store in Amundsen, we have to support ORM for relational DB first in databuilder. This RFC is mainly about the relational DB schema plan, model translator, and the corresponding 
change in data transformer, loader and publisher. 
## Motivation

Relational DBs, like MySQL, are well deployed and widely used in most workplaces. Using the well-established DB infrastructure would save the cost(e.g., new DB deployment, backup, monitoring, proxy) 
for potential Amundsen users. Hence, it would be helpful if Amundsen can support MySQL as metadata store. The expected result is user is able to publish extracted metadata to MySQL via 
data loader, transformer, and elastic search publisher would work with ORM as well.

## Guide-level Explanation (aka Product Details)

There would be new transformers, loader, publisher for ORM/MySQL use. For all current graph db users, they do not have to make any change. 
For new users who want to use MySQL as the backend store, they can still use the existing extractors and then pass new data transformer, loader to ETL task and the new publisher to job executor.


## UI/UX-level Explanation

N/A
## Reference-level Explanation (aka Technical Details)

- Still use the existing models but add model translators to support ORM schema definition and map node/relation data to the record that can be better consumed by relational DBs. 
In order to better reflect the translation, use node_label, relation_type names(probably with the concatenation of some fields) from current graph db models to define column name in new schema definition for relational DBs. 
Also, the model translator would also include the generator function to convert node/relation to record(e.g., `create_record_iterator_from_node_rel()`).

- Regarding the schema to be used in relational db, below is a rough idea of table(attributes) with denormalized model(note: table/attribute names are all from fields in current graph db models)

    - application(application_key, application_url, application_name, application_id, application_description)

    - metric(metric_key, metric_name, metric_description_key, metric_description, metric_type_key, metric_type_name, dashboard_key, dashboard_name)

    - table(table_key, database_key, database_name, cluster_key, cluster_name, schema_key, table_name, table_is_view, table_description_key, table_description, table_programmatic_description, 
    partition_key, high_watermark_key, high_watermark_partition_value, high_watermark_create_time, low_watermark_key, low_watermark_partition_value, low_watermark_create_time, source_key, source, source_type, stat_key, stat_name, stat_value, start_epoch, end_epoch)

    - column(column_key, column_name, column_type, column_sort_order, column_description_key, column_description, column_programmatic_description, stat_key, stat_name, stat_val, start_epoch, end_epoch)

    - tag(tag_key, tag_type, metric_key, dashboard_key, table_key, column_key)
    
    - lineage(upstream_key, downstream_key)
    
    - user(user_key, first_name, last_name, full_name, github_user_name, team_name, employee_type, manager_email, slack_id, is_active, updated_at, role_name, table_key)
    
    - updated_timestamp(updated_timestamp_key, latest_timestamp)

- Due to the reuse of graph db models, add new transformers for entities to convert 'node' and 'relationship' data to 'record' with model translators. 

- Add a new data loader(e.g., file_system_mysql_csv_loader) to consume transformed metadata records and dump them into csv to a new 'records' folder. The new data loader works with the ORM model translator and hence, the csv output could be consistent with the schema we defined for ORM.

- Add a new publisher(e.g., mysql_csv_publisher) to publish the loaded csv files from 'records' folder to MySQL. The new publisher works with the ORM model translator to create schema if necessary and is responsible for ORM CRUD operations.

- Add a new elastic search publisher supporting ORM operations

## Drawbacks

- Support of ORM would get a big change involved in model translator, data loader, transformer and publisher. 
Transformer for model translator might downgrade the performance due to the conversion from node/rel to record.

- We can still use all the existing graph db models, though need to keep the ORM model translator compatible with future change in graph db model. 

- Even if with a schema upgrade feature, relational db is not as flexible as graph db to handle metadata structure change
## Alternatives

- Instead of using model translators, develop complete ORM models containing schema definition and record generator(e.g., `create_record_iterator()`), though it will lead to two model versions for each entity. On the other hand, we may have to duplicate all existing extractors and make them work with the new ORM models.
Moreover, we will have to ensure any future changes compatible not only in two models but also in two extractors.

- Use normalized models to design schema in relational DBs. The disadvantage is it will hurt query performance due to multiple joins.

## Prior art

N/A
## Unresolved questions

- The initial relational DB schema plan was made according to the current graph DB models and it may need some polishment from Amundsen staff who are more familiar with real uses.
- Regarding how to map graph db models to ORM models, it would be great if there is a dynamic way for the model conversion
## Future possibilities

It is possible that one entity need additional metadata in the future, in which case, it's necessary to support schema upgrade for ORM. 