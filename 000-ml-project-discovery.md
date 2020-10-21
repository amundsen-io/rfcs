- Feature Name: ml_project_discovery
- Start Date: 2020-10-21
- RFC PR: [amundsen-io/rfcs#000-ml-project-discovery.md](https://github.com/amundsen-io/rfcs/pull/0000) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# ML project discovery using Amundsen

## Summary

Our proposal is to extend the capabilities of Amundsen by introducing new data resources covering the ML project lifecycle.

## Motivation

The rising number of ML projects force organizations to address challenges related to model governance and model discovery.
According to latest market surveys one can expect significant acceleration in AI adoption this and next year.

## Guide-level Explanation (aka Product Details)

The concept of machine learning project refers to complex process of moving your research from local exploration environment towards production grade deployments.
Essentially, that consists of multiple steps including feature engineering, tracking experiments, model training, batch scoring, model versioning, model staging, release and deployment.
Many organizations suffers from lack of visiblity over the ML assets delivered by the different teams of data scientists.
Such model catalog would improve the collaboration within the organization through single interface to discover available ML resources and related metadata.  

The basic use cases can be described as follow:

*   search for ml projects by project name, project description, project owner
*   search for ml models by model name, model description, model owner, model stage
*   search for model deployments by deployment name, deployment description, deployment sample request/response (prediction)
*   view the data sources used to build the model 
*   view the metadata of the model, basic evaluation metrics e.g. accuracy, precision, recall, f1 etc.

## UI/UX-level Explanation

TBD

## Reference-level Explanation (aka Technical Details)

ML assets should be structured in the similiar fashion to data assets. Apache Atlas already provides ML governance capabilities starting from version 2.1.0.
That requires to build custom type definitions on top of [built-in entities](https://github.com/apache/atlas/blob/release-2.1.0-rc3/addons/models/4000-MachineLearning/4010-ml_model.json) available in Atlas to adapt it according to existing Amundsen model.

TBD describe the sample process of generating ml assets from ml pipeline by leveraging framework such as Airflow

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

* store ml features
> Think about what the natural extension and evolution of your proposal would be and how it would affect the project as a whole in a holistic way. Also consider how the this all fits into the roadmap for the project and of the relevant sub-team.
> This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related.
> If you have tried and cannot think of any future possibilities, you may simply state that you cannot think of anything.
