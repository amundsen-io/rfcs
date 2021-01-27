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
and replace it with three operations:

- creating a git tag triggers a release
- periodical releases every month
- pre-releases with each change added to master

## Reference-level Explanation (aka Technical Details)

The plan for each release option is:

### Continuous Pre-releases

- Implement a new github action called `Pre-releases` that will trigger on every PR merged into master
- The action does:
  - Uses checkout action to get latest code for the workflow.
  - Uses python-semantic-release to auto generate the latest pre-release version number:
    - Updates changelog file.
    - Run :ref:`cmd-version`.
    - Push changes to git.
    - Run :ref:`config-build_command` and upload the created files to PyPI as a prerelease.
    - Run :ref:`cmd-changelog` and post to your vcs provider.
    - Attach the files created by :ref:`config-build_command` to GitHub releases as a pre-release.

### Periodical Releases

- Implement new github action `Monthly Release` that would be triggered on a monthly basis.
- The action does the following:
  - Uses checkout action to get latest code for the workflow.
  - Uses python-semantic-release to auto generate the latest version number. The following steps occur from the new-release branch:
    - Update changelog file.
    - Run :ref:`cmd-version`.
    - Push changes to git.
    - Run :ref:`config-build_command` and upload the created files to PyPI.
    - Run :ref:`cmd-changelog` and post to your vcs provider.
    - Attach the files created by :ref:`config-build_command` to GitHub releases.
  - Uses create-pull-request to create a PR with the changes from new-release to get approved and merged into master.

### Tagged Releases

- Keep the current github action `pypipublish` that already publishes a release on uploaded tags.

## Drawbacks

- Someone can push tag by mistake creating unwanted release
- Master branch is protected which makes it impossible to make releases fully automated without hacky workarounds.
- Need to make sure to get the latest from master into new-release before semantic release action is performed.

## Alternatives

N/A

## Prior art

N/A

## Unresolved questions

- Is the current monthly release with PR good enough for us?
- Would we need to do the same for pre-releases?

## Future possibilities

- Notifying Slack community automatically when new releases are created
