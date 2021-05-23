- Feature Name: Unify shared dependencies
- Start Date: 2021-05-22
- RFC PR: [amundsen-io/rfcs#38](https://github.com/amundsen-io/rfcs/pull/38) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Unify shared dependencies

## Summary

Major motivation behind migrating to monorepo was to solve `Dependency Management` issue. However, it still remains unsolved.

This RFC serves as a platform to figure out the best approach for unifying shared dependencies and contains preliminary proposal for how that could be solved.

## Motivation

Following original RFC:

```
Dependency Management: Python packages are pretty much the same for each proxy, which makes it hard to sync across all the above repositories. As a result most of the time, each proxy is running its own version of the dependency.
A few examples:

| Package             | amundsenmetadata   | amundsenfrontend  |
| --------------      |:------------------:| -----------------:|
| amundsen-common     | >=0.8.1            | ==0.6.0           |
| flake8              | ==3.5.0            | ==3.8.4           |
| flake8-tidy-imports | ==1.1.0            | ==4.2.1           |

and many more... This change will put all the packages in just one place, making practically to keep dependencies similar accross verisons, simply because it will be possible to open a single PR that updates multiple packages. Practically speaking this will improve the status quo slightly.
```

Most common pain-points so far:

- mismatch between version of amundsen-common in frontend/metadata/search
- mismatch between version of marshmallow in frontend/metadata/search 

## Guide-level Explanation (aka Product Details)

n/a

## UI/UX-level Explanation

n/a

## Reference-level Explanation (aka Technical Details)

Following steps are proposed to achieve goal of this RFC. 

Multiple times you might find the term `subdirectory`, which refers to following folders:
- common
- databuilder
- frontend
- metadata
- search

Steps:

0. Undertake new approach where all dependencies land in `install_requires` or `extras_require` through reading in appropriate requirements* files (except databuilder due to it's complexity)
1. Split `requirements.txt` for each subdirectory into `requirements.txt` and `requirements-dev.txt` (there is no need to install flake8 when doing for example `pip install amundsen-metadata` which is currently the case)
2. Identify libraries that are shared across all (or majority - see `Drawbacks`) of subprojects
3. Create `requiremnts-common.txt` file in root folder of amundsen repo containing pinned dependencies for shared libraries
4. Remove libraries listed in `requirements-common.txt` from `requirements.txt` of each subdirectory
5. Create `requirements-dev.txt` file in root folder of amundsen repo containing pinned dependencies for shared dev libraries (mypy, flake8, etc.)
6. Remove libraries listed in `requirements-dev.txt` from `requirements.txt` of each subdirectory
7. Create symbolic link to `requirements-common.txt` and `requirements-dev.txt` in each subdirectory
8. Shift context of building docker images to root folder (otherwise we cannot share `requirements-common.txt` and `requirements-dev.txt` symlinks across subdirectories)
9. Extend every `setup.py` to contain following entries (`extras_require` might contain more keys)"
```python
install_require=requirements + requirements_common
extras_require(dict(all=requirements + requirements_common + requirements_dev, dev=requirements_dev)),
``` 
Where:
    - `requirements` - requirements.txt specific for subdirectory
    - `requirements_common` - requirements-common.txt /symlink/ from project root
    - `requirements_dev` - requirements-dev.txt /symlink/ from project root)
10. Introduce `install_deps` action in Makefile to be used by users/in cicd, doing:
```python
pip3 install -e ".[all]"
```
10. In CI/CD apart from `make install_deps` do also `pip3 uninstall amundsen-common && pip3 install ../common` (apart from common subdirectory which would not re-install `../common`)

### Testing

We test this with `make test` on each subproject + building Amundsen locally and testing functionalities manually relying on test data

## How We Communicate This

- We'll promote this RFC frequently on Amundsen Slack and other social media channels, to get feedback from the community.
- During the community meetings, update on the progress
- Update docs for each component

## Drawbacks

- As we don't have 100% test coverage it's difficult to asses complete impact of evening out dependencies so we might enter period of ironing out potential issues
- Using symlinks might be tricky for users using Windows
- There are dependencies that are shared by some but not all libraries. For those we have two choices:
    - Include them in `requirements-common.txt` (preferred by the author) - this will make some packages contain deps that are not used
    - Populate `requirements-common.txt` only with deps used by all packages - this will still leave a lot of room for inconsistencies
- It might be difficult to keep the dependencies balanced especially if some custom entry in one of subdirs `requirements.txt` requires different version of lib from `requirements-common.txt` (should be a low risk)
- Local installation requires mandatory `-e` flag when doing `pip install .` (might be solved by thoroughly communicating/documenting the change)

## Alternatives

- Using pip functionality `constraint` but I've identified this as less intuitive for end users as:
    - Users would need to remember about adding constraints while doing pip install - if they don't, Amundsen might not work properly which might result in:
        - more difficult onboarding amundsen in new projects
        - increased no of github issues
        - increased no of questions on Slack
- Keep the current state with different versions of the same libraries across subdirectories

## Prior art

- [Monorepo RFC](https://github.com/amundsen-io/rfcs/pull/31)

## Unresolved questions

- Should we scope all subdirectories with that change or would we be ok excluding some and focusing on microservices only (for `requirements-common.txt`)

## Future possibilities

- Easier upgrade possibilities (for example bumping to Flask 2+)