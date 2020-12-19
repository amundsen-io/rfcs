- Feature Name: Automated Release Process
- Start Date: 2020-12-19
- RFC PR: [amundsen-io/rfcs#20](https://github.com/amundsen-io/rfcs/pull/20)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Automated Release Process

## Summary

This RFC proposes to automate release with github actions. 

## Motivation

Currently release process requires manual release creation and changelog rendering (by going through list of commits).

The motivation behind this RFC is to automate, unify and simplify release process.

## Guide-level Explanation (aka Product Details)

Having enforced naming convention for MRs we can leverage this to automatically categorize MRs and render them in 
changelog. By using automated github actions we can remove manual step from release process (releasing, rendering changelog) 
and replace it with simple git tag creation.

## UI/UX-level Explanation

Pushing new tag by user creating release triggers and makes releasing almost effortless.

## Reference-level Explanation (aka Technical Details)

- Implement new github action `Release` that would be triggered on tag push

## Drawbacks

- Someone can push tag by mistake creating unwanted release

## Alternatives

N/A

## Prior art

N/A 

## Unresolved questions

- So far I've only looked into https://github.com/marketplace/actions/generate-changelog-action but some testing is required to confirm it'd work with our MR naming convention or different tool should be used 

## Future possibilities

- Notifying Slack community automatically when new release is created
