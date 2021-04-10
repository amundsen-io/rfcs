- Feature Name: monorepo
- Start Date: 2021-03-30
- RFC PR: [amundsen-io/rfcs#0030](https://github.com/amundsen-io/rfcs/pull/30)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Move to Monorepo

## Summary

At the moment, Amnundsen project is divided into 6 repositories:
	- amundsen
	- amundsenfrontendlibrary
	- amundsenmetadatalibrary
	- amundsensearchlibrary
	- amundsencommon
	- amudnsendatabuilder

This RFC proposes merging all of these repositories into a single repo.

There will be no change to the built artifacts. Users who use PyPi packages or Docker images will have no change to their deployments. Users with forks or submodule based customization will have to make minor changes to their CI/CD pipelines to pull from the correct directories, but the required changes will be minor.


## Motivation

The submodule architecture creates the following issues and is making it hard for companies to adopt Amundsen, and to customize it accordingly. It also causes substantial developer toil in the form of having to create multiple small PRs to merge one

1. Dependency Management: Python packages are pretty much the same for each proxy, which makes it hard to sync across all the above repositories. As a result most of the time, each proxy is running its own version of the dependency.
A few examples:

| Package             | amundsenmetadata   | amundsenfrontend  |
| --------------      |:------------------:| -----------------:|
| amundsen-common     | >=0.8.1            | ==0.6.0           |
| flake8              | ==3.5.0            | ==3.8.4           |
| flake8-tidy-imports | ==1.1.0            | ==4.2.1           |

and many more... This change will put all the packages in just one place, making practically to keep dependencies similar accross verisons, simply because it will be possible to open a single PR that updates multiple packages. Practically speaking this will improve the status quo slightly.

1. Development Efforts: Having multiple repositories makes it really hard to implement a feature. Implementation and testing require efforts to synchronize and then code reviews, and finally, all the PRs across multiple repositories need to land in master at a certain time or at the same time. This change results in faster development, one PR to fix a bug or a feature, no dependency hell as we are facing today, hence attract more contributors.

1. Customization & Deployment: One of the frequently mentioned topics re Amundsen is the complexity of the architecture, and the efforts required to customize Amundsen. Having multiple repositories results in multiple components to install, customize, manage and fix. If you have forks, there is more work to apply your changes on a per-repo basis. Having a single repository to work from will reduce toil.

1. Code Duplication: From requirements to config files, models, helper functions, exceptions handling, CI/CD pipelines, Docker files, and other repository management like licenses, PR/Issues templates, etc., is pretty much the same in all these microservices. Change in one repository deviates that repository from others at the moment. This change will move all the above into a single repository making it easy to change anything without keeping track of multiple repositories, and redundant code.

Having one repo doesn't preclude us having incubator repositories, they can be handled similarly as today.


## Transition Path

The transition will be done on a new branch on the `amundsen` repo, named `main`. We will move all code there and leave `master` as the default branch in GitHub until the migration is complete (plus a waiting period).

### Phase 1:
We create a new branch on the `amundsen` repo, named `main`. All work that follows in this description will occur in this branch.

### Phase 2:
Raw code import and CI. We import all of the code as a snapshot into the repository.

At this point, we migrate the CI/CD pipeline, and automated release mechanisms, from the consituent repositories to the main repository. They will publish packages using a `-beta` postfix.

### Phase 3:
We freeze code in all the submodules: no new PRs in. Existing PRs will have 1 week to land. We will assist authors in getting nearly-mergable PRs landed. For PRs that don't make it, we will provide instructions to port their patches to the new repo, if they choose to re-submit.

### Phase 4:

We re-copy all code from the submodules into the main repository using [https://github.com/shopsys/monorepo-tools](https://github.com/shopsys/monorepo-tools) to preserve git commit history for each of the branches.

PRs against the main repo will now be allowed from the community. This means there was a 1 week period where no new PRs were accepted to the project. This is necessary for us to merge out the old remaining PRs, and to avoid any annoying merge conflicts once those land.

The old repositories will now be frozen to new contributions from non-maintainers (maintainers can still create critical hotfixes if absolutely necessary). There will be a deprecation notice added to the top of the README for each subrepo.

### Phase 4:
We mark the `main` branch on GitHub as the main branch, so that new pulls use the new project. We update the documentation pipeline to build from the Main branch, so that the new documentation goes onto the website.


### Phase 5:
After 1 year, we will remove all of the sub-repos, as well as the `master` branch on the Amundsen repository.


## How We Communicate This

- We'll promote this RFC frequently on Amundsen Slack and other social media channels, to get feedback from the community.
- During the community meetings, update on the progress and phases.
- We'll write a Blog/Tutorial about this change on how it's recommended to do customizations (we have been workshopping it here https://medium.com/stemma/amundsen-deployment-best-practices-740a1800518e and would like to bring it upstream once it's better tested with the monorepo).

## Drawbacks

There is a transition cost both internally, and for certain users who do customizations.

Currently, it is possible for implementors who do code-level customization to keep different versions of the various microservices. This will make that challenging or impossible within a single repo.


## Alternatives

We've discussed alternatives that also bundle together service-level


## Unresolved questions

None, yet

## Future possibilities

It may be desirable to make architectural changes after this code structure change is done, for example consolidating components or changing how shared dependencies are managed. However, we'd like to keep that out-of-scope for this RFC and keep those discussions separate (unless if it impacts this portion).

We will also have the option to use [pip constraint files](https://pip.pypa.io/en/stable/user_guide/#constraints-files), which would likely help with the dependency issues we've encountered. This could be considered deeper and implemented as a PR after the transition occurs.
