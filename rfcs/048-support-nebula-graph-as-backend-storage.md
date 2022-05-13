- Feature Name: Support Nebula Graph
- Start Date: 2022-04-16
- RFC PR: [amundsen-io/rfcs#48](https://github.com/amundsen-io/rfcs/pull/48)
- Amundsen Issue: [amundsen-io/amundsen#1816](https://github.com/amundsen-io/amundsen/issues/1816)

# Nebula Graph Support

## Summary

The support includes the Nebula Graph data builder and a new proxy for Nebula Graph in metadata service.

## Motivation

Metadata can be published into Nebula Graph from Amundsen data builder now and this RFC is going to make metadata retrieval API work for Nebula Graph in metadata service.  

## Guide-level Explanation (aka Product Details)

The goal of this RFC is to add additional loaders, publishers, and serializers to the library suite so that Nebula Graph is supported.

## UI/UX-level Explanation

N/A

## Reference-level Explanation (aka Technical Details)

A new proxy for Nebula Graph is added to support Nebula Graph in the metadata service.

To support Nebula Graph in the data builder, several new components are added:

- Nebula Extractor
- Nebula Search Data Extractor
- Nebula CSV Loader
- Nebula CSV Publisher
- Nebula Serializer
- Nebula Sample Data Loader

### Nebula Graph Schema handling

They worked quite similarly to those components for Neo4j(it even speaks a dialect of OpenCypher) but differentiated in one thing: Nebula is not schemaless like Neo4j, that is, a label(named tag in Nebula Graph) or an edge type should be created before it's being referred in a query.

Instead of maintaining a versioned schema by the user(with extra interfaces introduced), the proposed design is to parse the schema from data being published(Nebula CSV Publisher), which will do a schema check and DDL change when needed automatically, where the schema information will be in a single source of truth: the data model of the data builder.

This requires the user to run the Nebula data builder sample script to initialize the Nebula Graph Schema, which could be discussed/revisited.

### Nebula Graph Index

Another difference in Nebula towards Neo4j is, when it comes to "starting point seeking" of an OpenCypher `MATCH` Query, if non of the `key`(it's called Vertex ID in Nebula Graph) is provided, an index on the LABEL(tag)  should be created(or a `LIMIT` clause is added), that is either no conditions at all: `MATCH (n: Table) return n` or conditions for starting point is only under a property: `MATCH (n: Table{category: "foo"}) return n` in the query. [Here](https://siwei.io/en/nebula-index-explained/) is a post where I explained why it's designed so.

All those data models with queries in meta service API required indexes were handled by Nebula Publisher now, included in `NEBULA_INDEX_TAG_FIELDS`.

## Drawbacks

The RFC adds support for another datastore which brings in additional components and increases the code size of the repo.

## Alternatives

For the Nebula Schema auto adaptation in Nebula Publisher, one alternative is adding an extra interface/utility to manage the Nebula Graph schema.

## Prior art

N/A

## Unresolved questions

N/A

## Future possibilities

None so far.
