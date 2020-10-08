- Feature Name: Add a abstraction layer to the databuilder's record model
- Start Date: 2020-10-02
- RFC PR: [amundsen-io/rfcs#5](https://github.com/amundsen-io/rfcs/pull/5)

- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Add a abstraction layer to the databuilder model

## Summary

Currently the databuilder's records are all subclasses of the class `Neo4jCsvSerializable` and the output of these models are Dictionaries formatted for neo4j. This RFC proposes this coupling is broken and neo4j serialization is removed from the models and moved into the the loader portion.

## Motivation

While SeatGeek was in the process of writing the Neptune integration portion for the databuilder a problem came up of seperating Neo4j from the databuilder's records. The way Neo4j is receives nodes and edges is different from the way Neptune or other graph databases do. So SeatGeek wanted to come up with a process that:
> Maintained feature parity between the datastores.
> Did not add additional overhead for adding new features to the databuilder
> Made it easier to add other data stores in the future. 

## Guide-level Explanation (aka Product Details)

This is less of a feature and more of a refactor. For this rfc this section is duplicate of the `Reference-level Explanation` section.

## UI/UX-level Explanation

This would be a backend only change no UI level changes are required. 

## Reference-level Explanation (aka Technical Details)

Each of the models in the databuilder are built off of the `GraphSerializable` class. The `GraphSerializable` class is an abstract class that is responsible for validating and returning the nodes and relationships of its subclasses. The interface on the `GraphSerializable` is `next_node` with a return type of `Union[GraphNode, None]` and `next_relation` with a return type of `Union[GraphRelationship, None]`. 

GraphNode would be defined as named tuple with the attributes: 
- key: str
- label: str
- attributes: Dict

GraphRelationship would be a named tuple with the attributes: 
- start_label: str
- end_label: str
- start_key: str
- end_key: str
- forward_label: str
- reverse_label: str
- attributes: Dict

The loader would be then be responsible for taking these graph nodes and graph relationships and converting them to the format expected by the publisher. 

The unpolished changes mentioned above can be found: https://github.com/AndrewCiambrone/amundsendatabuilder/pull/6

## Drawbacks

This adds a level of complexity because it adds an additional layer to the databuilder. Instead of having the neo4j serialization built into the models it becomes a separate process. This also would add a small performance degradation because the models are producing named tuples and then they are being convered into another format. 


## Alternatives

1. Duplicate all the models for each datastore but that opens the possibility of missing features in one datastore over the other. 

2. Translating from neo4j models into Neptune. However this would require future contributions to remember to make sure the changes are compatible with the neptune translator. 

## Prior art

N/A

## Unresolved questions

> What parts of the design do you expect to resolve through the RFC process before this gets merged? 
General agreement over the approach

## Future possibilities

This is the first step in introducing other data stores into the databuilder. In the future SeatGeek would like to introduce the other portions of the Neptune integration that they have been working on but since this part of the change is a bit of a overhaul of all the models, its important to get this portion in first. 
<<<<<<< HEAD

=======
>>>>>>> b4cbebf0f32fa8f7e2682ef3981775b5c1ff4fa8
