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

- Implement new github action `Monthly Release` that would be triggered on a monthly basis.
- The action does the following:
  - Uses checkout action to get latest code for the workflow.
  - Uses python-semantic-release to auto generate the latest version number. The following steps occurr from the new-release branch:
    - Update changelog file.
    - Run :ref:`cmd-version`.
    - Push changes to git.
  - Uses create-pull-request to create a PR with the changes from new-release to get approved and merged into master.

- `Publish Monthly Release` is triggered when version is bumped and changelog file is updated and PR with these changes is merged to master. 
- The action takes the following steps:
  - Check out the latest code from master
  - Uses semantic release to get the current version
  - Create a Github release with the current version and changelong from the changelog file
  - Generate dist
  - Publish to pypi


## Drawbacks

- Someone can push tag by mistake creating unwanted release.
- Master branch is protected which makes it impossible to make releases fully automated without hacky workarounds.
- Could accidentally trigger publishing if the two files were for soem reason modified in a PR that doesn't actually bump the version.

## Alternatives

N/A

## Prior art

N/A

## Unresolved questions

N/A


## Future possibilities

- Notifying Slack community automatically when new releases are created
