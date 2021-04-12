- Feature Name: lineage-graph-visualization
- Start Date: 2021-04-01
- RFC PR: [amundsen-io/rfcs#33](https://github.com/amundsen-io/rfcs/pull/33)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Amundsen Lineage Stage 1 - Graph Visualization

## Summary

This is the next in the series of https://github.com/amundsen-io/rfcs/pull/25

Implement a navigable canvas-based graph visualization for lineage. Lineage is traditionally represented this way, suboptimally, in the various existing tools like CompilerWorks, Castor, Apache Atlas, etc.

This will replace the existing `Upstream` and `Downstream` tabs on the table detail page, and introduce a new CTA "Lineage" on the table detail page.

## Motivation

> Why are we doing this? What use cases does it support? What is the expected outcome?

A map graph UI can more fully, transparently, and frictionlessly convey the lineage information. By co-locating data from multiple sources, Amundsen is positioned to create a better map graph UI experience vs. competitors.
v0 of lineage is already available in the Amundsen project today in the form of a list.

## Guide-level Explanation (aka Product Details)

This feature will expose upstream and downstream tables and columns within the Table Details page, using a navigable graph visualization.

- Lineage: Lineage is a term that describes the flow of data from one entity to another. While this term can broadly include everything from services, events, ETLs, and dashboards, we will focus on table-to-table and column-to-column data lineage in this RFC.
- Upstream: Upstream is a relative term that describes data sources from which we inherit. Data flows from upstream to downstream.
- Downstream: Downstream is a relative term that describes data entities that consume our data.
- Level: The depth of the graph in terms of relationships with another table. By default, this will only display first-level (immediate) upstream and downstream tables.
- Direction: Upstream, Downstream, or Both. Users will be able to filter out the graph based on the direction. by default, we will display `Both` upstream and downstream tables.

Since this is a UI/UX feature, more details about the implementation in the next section.

## UI/UX-level Explanation

> Explain the UI changes that your proposal would need (if applicable). This could mean:

The feature will be disabled by default and can be enabled via a configuration flag. (This is how Amundsen enables/disables lineage today)
Once enabled,

- a new CTA/Button will appear on the top right of the table detail page.
- a new icon in front of the column name (to support column-level lineage).

Clicking on any of the above will navigate the user to a new page displaying lineage graph along with the filters to select the `level` and `direction` for the graph.

The sidebar will have the filters available and will be used for other interactions like displaying a complete list of nodes, and/or view the metadata of a selected node.

Interaction Details:

- By default, we will display 1 level (or depth) of the lineage. Users will be able to control the level by clicking Expand/Collapse via each node. Or simply use the filter to change the number of levels shown.
- User will be able to Pane and Zoom, and also reset the whole graph.
- Clicking any node will display the metadata details of that particular node in the sidebar.

Important to note: We will not display the complete list of upstream/downstream tables. We will only display X number of tables by default based on their usage and will control the number by a config flag.

See the Screenshots in [this folder](https://github.com/amundsen-io/rfcs/tree/98c240d29d8644a81b4eaf6a1481a126d08b5732/assets/033).

## Reference-level Explanation (aka Technical Details)

This does not change anything on the backend and uses the existing endpoints exposed in the v0 of lineage.
All the UI/UX explanations mentioned in the above section.

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
