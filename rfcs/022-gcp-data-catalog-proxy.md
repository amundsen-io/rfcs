- Feature Name: GCP Data Catalog Proxy Support
- Start Date: 2021-02-04
- RFC PR: [amundsen-io/rfcs#22](https://github.com/amundsen-io/rfcs/pull/22)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# GCP Data Catalog Proxy Support

## Summary

This RFC proposes introducing support for GCP Data Catalog as new Proxy.

## Motivation

As of now there is no support for fully-managed proxy in Amundsen. It always entails:
* running backend (elasticsearch+neo4j, atlas+solr/elastic+hbase/cassandra, or just elastic when using neptune)
* metadata ingestion processes

This RFC proposes proxy, whose backend services:
* are fully managed by cloud operator (GCP) and do not entail any maintenance
* are integrated with data entities created within GCP
* to fulfill basic Amundsen functionality don't require databuilder usage

From the perspective of current GCP users:
* it is super simple deployment not requiring any maintenance on backend services
* it can be useful if projects do not want to share GCP Console to end users (which is required to access Data Catalog)
* are in need of modern UI and good UX. Data Catalog has simple and unwelcome user interface lacking of modern solutions and inviting UX. In this area Amundsen can serve as a thin layer of fresh, friendly and wisely-opinionated frontend.

Another benefit is it expands Amundsens reach by supporting mainstream cloud provider.
 
## Guide-level Explanation (aka Product Details)

Instead of running and maintaining Amundsen backend services for metadata, users of GCP can use Amundsen with Data Catalog as backend.

The installation requires minimal and be gradually enriched by the use of google-provided metadata connectors (https://github.com/GoogleCloudPlatform/datacatalog-connectors)

## UI/UX-level Explanation

N/A

## Reference-level Explanation (aka Technical Details)

To support GCP Data Catalog proxy several new components are needed: 

* amundsenmetadata proxy
* amundsensearch proxy
* amundsendatabuilder serializer. It can be optional in first iteration since there are existing GCP Metadata connectors supporting basic functionalities
    under: https://github.com/GoogleCloudPlatform/datacatalog-connectors

## Drawbacks

* another datastore which brings in additional components and increases the code size of the repo
* no control over Data Catalog (and it's API) means incompatibilities might arise with time

## Alternatives

* do not offer GCP Proxy and make users use one of other proxy options

## Prior art

N/A 

## Unresolved questions

* desirability
* do we rely on amundsendatabuilder for table/dashboard core metadata or only for features like readers, owners, etc.

## Future possibilities

- Becoming GCP choice for data discovery in cloud 
