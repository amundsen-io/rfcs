- Feature Name: Support MySQL as the metadata backend in metadata service
- Start Date: 2021-04-16
- RFC PR: [amundsen-io/rfcs#34](https://github.com/amundsen-io/rfcs/pull/34) 
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Support mysql as the metadata backend in metadata service

## Summary

The support includes the database initialization and a new proxy for MySQL in metadata service.

## Motivation

Metadata can be published into mysql from Amundsen databuilder now and this RFC is going to make metadata retrieval API work
for MySQL in metadata service.  

## Guide-level Explanation (aka Product Details)

- The database initialization will be supported by CLI. The command will init/upgrade table schemas according to 
[amunsen_rds models](https://github.com/amundsen-io/amundsenrds) for MySQL.
- When we use MySQL as the metadata backend, the env 'PROXY_CLIENT' will be applied to make the API call point to the mysql\_proxy.

## UI/UX-level Explanation

N/A

## Reference-level Explanation (aka Technical Details)

- Database initialization: In [amundsen_rds](https://github.com/amundsen-io/amundsenrds), we will add Alembic to manage schema changes,
which will include model change versions for schema migration. 
In metadata\_service, the command for database init will apply the latest schema version of amundsen_rds and finish the schema init/migration work.

- mysql\_proxy: It will work similarly to other proxies, like neo4j\_proxy and aims to handle API calls, retrieve metadata for multiple entities.

## Drawbacks

- The new proxy for MySQL would get some dev work involved and it will increase code size of metadata library. 

## Alternatives

- For the schema change management, we can add Alembic into metadata service to take the advantage of Flask-Migrate, but 
it might be a little inconvenient to autogenerate the schema change version if Alembic and amundsen_rds models
 are not in the same repo.

## Prior art

N/A

## Unresolved questions

N/A

## Future possibilities

In the future, if we switch to call metadata service to publish data into mysql, it might be possible to move rds models and alembic 
into metadata service so that the db models and migration can be managed by Flask SQLAlchemy/Migrate.
