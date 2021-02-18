- Feature Name: Add MySQL as a data sink in the databuilder
- Start Date: 2021-02-17
- RFC PR: [amundsen-io/rfcs#0000](https://github.com/amundsen-io/rfcs/pull/0000) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Add MySQL as a data sink in the databuilder

## Summary

This RFC aims to add multiple modules, like serializer, data loader and publisher to load metadata into MySQL.

## Motivation

Currently, ETL flow works for graph databases. In order to support MySQL as a ackend metadata store, we need add new data loader, and publisher to push extracted metadata into MySQL.

## Guide-level Explanation (aka Product Details)

With the new serializer, data loader, publisher, users could invoke them in ETL job of databuilder if they are using MySQL as the backend store.

## UI/UX-level Explanation

N/A

## Reference-level Explanation (aka Technical Details)

1.`mysql_serializer`: it serializes the record instances which are in the format of MySQL ORM 
[models](https://github.com/amundsen-io/amundsenrds) to dictionary. It mainly converts all metadata attributes of record to
key value paris in dictionary and excludes their SQLAlchemy attribute, '\_sa\_instance\_state'.

2.`file_system_mysql_csv_loader`: it is used to generate csv files from extracted and serialized metadata record instances.
This data loader invokes `mysql_serializer `first to get record in dictionary format and would get table name from 
the record instance by calling its `__tablename__` attribute.

We forced the record\_iterator to yield record instance in the order of topography order(e.g., database -> cluster -> schema -> table -> column),
and the data loader will output csv files in the same order. In `mysql_publisher`, however, the python function `listdir()` can not list files
in that order. My plan is to add index with a separator in the file name(e.g., 1-table\_name) to make all files can be sorted first based on the index when listed in `mysql_publisher`. 
The index will be set by the length of `file_mapping` dictionary, which is used for csv writer in the new `file_system_mysql_csv_loader`.

The format of the generated csv files will be same as the mysql table records, namely, field names on header row and then field values on the subsequent rows.

3.`mysql_csv_publisher`: it reads csv files with the given directory and get ORM model class from the table name in the file name. 
The publisher will also set extra attributes(published\_tag, publisher\_last\_updated\_epoch\_ms) for the record instances before calling SQLAlchemy ORM methods to upsert csv records to mysql with the given transaction size.

4.`mysql_search_data_extractor`: it is going to extract ORM objects with their SQLAlchemy relationships from mysql and convert them to current ElasticSearch document objects. Then it can work with the existing `FSElasticsearchJSONLoader` and `ElasticsearchPublisher`
to execute the following es_publish job. It will support `table`, `dashboard`, `user` data search and the extraction from mysql would be in the method level instead of query language level. 
I.e., there will be methods like `search_tables()`, `search_dashboards()`, `search_users()` dispatched based on the config. The functionality of each method will refer to 
the CQL in the current [neo4j\_search\_data\_extractor](https://github.com/amundsen-io/amundsendatabuilder/blob/master/databuilder/extractor/neo4j_search_data_extractor.py#L23-L115). 

Note: the new data extractor may not support custom SQL for search currently.


## Drawbacks

`mysql_search_data_extractor` would get joins among multiple tables involved when fetching mysql data, which can not perform as fast as that in graph databases. 
On the other hand, it may not support custom SQL from config to generate es document objects currently and will call default table/dashboard/user search method instead.

## Alternatives

1.Instead of adding an index in csv file name, we can list files based on their create\_time in `mysql_csv_publisher`. However, the method getting file create_time, like `os.path.getctime()` or `stat().st_ctime` only returns the change time of the file metadata, or only works for a specific OS. An another possible solution is to maintain a constant reflecting the topological order of table names which can be used to read files, but it will need extra 
maintenance.

2.Instead of reading table name from csv filename, we can put the table name into serialized dictionary and then in csv files, but it will need a little extra work, 
converting table name to model class for each record and exclude table name field when publisher dumps csv content into MySQL ORM models. 


## Prior art

N/A

## Unresolved questions

N/A

## Future possibilities

According to the offline talk with Tao Feng, we will consider to call metadata API for loading extracted metadata into mysql sink in the future. 
