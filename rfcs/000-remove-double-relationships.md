- Feature Name: Remove double relationships between Neo4j nodes
- Start Date: 2021-05-25
- RFC PR: [amundsen-io/rfcs#0000](https://github.com/amundsen-io/rfcs/pull/0000)
- Amundsen Issue: 

# Remove double relationships between Neo4j nodes

## Summary

Currently all (most?) relationships between Neo4j nodes are modeled with
two directional edges. This RFC proposes we only use one directional edge
between the nodes.

## Motivation

The double relationship graph model makes Neo4j queries more complicated
than they need to be, with no performance or functional benefit. Additionally
due to conflicting changes in the past (PR 70 and PR 206), Amundsen instances
that have been running and ingesting data since prior to PR 70 or PR 206 have
inconsistent Neo4j database state. Some nodes have a single relationship between
then, some have two.

There are two relevant Slack discussions regarding this change:

https://app.slack.com/client/TGFR0CZM3/CGFBVT23V/thread/CGFBVT23V-1615391140.022900
- a thread where I asked the question on why the double relationships exist
https://app.slack.com/client/TGFR0CZM3/CGFBVT23V/thread/CGFBVT23V-1563487877.254000
- a thread with Joshua Haskins' benchmarking results for modeling the relationships
  with one or two edges

## Transition Path

The implementation work involved:

For each double relationship pair, we need to decide which relationship type
to keep and which one to remove. I suggest we do this by keeping the directionality
and tag name consistent (e.g. remove all relationships with _BY suffix). There
might be instances where we need to change the direction of the relationship,
because FOO_BY and BAR_BY relationships might've been defined using the opposite
directionality.

Delete the insertion of the relationship type we've decided to remove from the
code, and, if needed, change the direction of the relationships we've decided to
keep.

Create a database migration script that deletes the relationship types we've
decided to remove from the Neo4j database, and change the direction of any
relationship we've decided to keep, if needed.

There is no transition path for new installations. They will simply start using
the single relationships graph model from the get go.

For existing Amundsen instances we'll need to make sure the database migration
script is run after upgrading Amundsen to a version where the double relationships
have been removed.

## How We Communicate This

TBD

## Drawbacks

I think the biggest reason for not doing this is the potential for the database state
to become even more inconsistent for Amundsen instances that have been running since
before PR 70 or PR 206. We'll need to make sure that the db migration scripts either
run automatically, running the db migrations is made abundantly clear and not optional
to any Amundsen users, or there's some sort of clear notification provided in Amundsen
UI if the db migration scripts haven't been run.

It is possible dataset ownership information, as an example, and other information linked
with a double relationship could become inaccessible, if the database state isn't migrated
properly.


## Alternatives

The alternative of not doing the change will leave existing Amundsen instances' Neo4j
databases in an inconsistent state, unless they were stood up after PR 206. The other
reason in favor of implementing the change rather than not, is that using single
relationships is actually the right way to model Neo4j graphs. There isn't a single
Neo4j tutorial or data modeling guide that advocates for double relationships. It's
just maintenance overhead and complicates the creation of nodes/edges, as well as
querying the Neo4j data.

## Unresolved questions

The exact implementation of Neo4j database migration scripts and how they get executed
is something that requires a bit more Amundsen expertise that I currently have. Same
goes for the communications plan.

Additionally the list of double relationship pairs need to be compiled and a decision
made on which relationship from each pair to remove, and should we change the direction
of the remaining relationship. I can compile the list and make a first pass proposal
on which relationships to delete and any direction changes once we get consensus on
whether this change is warranted or not.

## Future possibilities

It fixes a source of database inconsistencies, makes the Neo4j database model
consistent with Neo4j database modeling best practices and simplifies the database
operations implementations for Neo4j within Amundsen. All of those benefits should
make future changes in the Neo4j database model easier to implement.