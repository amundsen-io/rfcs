- Proposal Name: Use the Black formatter to format all Python code
- Start Date: 2020-11-10
- RFC PR: [amundsen-io/rfcs#13](https://github.com/amundsen-io/rfcs/pull/14)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# <RFC title>

## Summary

Format all Python code across Amundsen codebases with [Black](https://black.readthedocs.io/en/stable/).

## Motivation

### Motivation for code formatting in general

As an open source contributor - and in general, a contributor to any codebase - having to think and care about code formatting adds an extra hurdle to contribution.
While writing code, you have to ask yourself "is this conforming to the style of the code around it?". We use flake8, but its standards are very lax and it can't fix
anything for you, so when you think you're done coding, you have to go through a tedious loop of "lint -> fix -> lint -> fix" until everything is good. If a PR needs
any changes, the cycle begins again.

It is also easy to forgot to fix your code formatting, push your PR and find that you have to go back and fix it, and PRs can languish in this state. I myself have pushed
PRs which passed tests but not linting. In my case, a maintainer ended up taking their time to fix it. 

All of these problems moot with a formatter. You write your code, run the formatter, and then everything is in shape.

### Motivation for Black specifically

There are not many great Python formatters. @feng-tao pointed out that Airflow [evaluted](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=99844429) three options (YAPF, autopep8, Black).
The proposer favoured YAPF for its flexibility, but they ultimately chose Black.

Black has two major advantages:

- It is deterministic. Black parses code into an AST and then renders it again. It allows practically no room for aesthetic choice, except by modifying code itself.
- It is _not_ very configurable, and therefore leaves little room for unnecessary debate. Black's only substantiative option is line length.

autopep8 fails on both counts. YAPF is highly configurable, but I argue that is a drawback.

## Guide-level Explanation (aka Product Details)

_Teaching black to codebae contributors..._


_If we do not use a precommit hook_: The Amundsen codebase is formatted with Black, which enforces a consistent code style. Before pushing your changes, all you have to do to pass the lint step in CI is run `black .`. 
_If we **do** use a precommit hook_: The Amundsen codebase is formatted with Black, which enforces a consistent code style. When you commit any changes, a pre-commit hook will run `black .` to format your code.

## Reference-level Explanation (aka Technical Details)

[This PR](https://github.com/amundsen-io/amundsendatabuilder/pull/405) implements the change for the Amundsen-Databuilder codebase. It:

- makes a few changes to unnest some deeply nested data structures
- applies `black` to the entire codebase
- configures `black` to use 120 lines (which was how `flake8` had been configured)
- configures `flake8` to be compatible with Black

We may also want to figure out how to prevent a huge change to `git blame`. I am ambivalent on this. My argument against is that if anyone is using git blame and finds a line which was reformatted, they
can just look at the previous blame. However, there are [solutions](https://github.com/psf/black#migrating-your-code-style-without-ruining-git-blame).

## Drawbacks

- Updating `blame` for hundreds of lines would be minor drawback, but as mentioned, that can probably be resolved.
- The new style is a significant deviation from the existing style.
    - Some apsects are trivial, e.g. quotes are swapped to `"`,
    - I argue that most of the changes are improvements,
    - ... but Black does have trouble with very long identifiers/strings and deeply nested data structures, and Amundsen includes a fair number of these
      due to the nature of its configuration.
        - ... but in my opinion, having decided that the codebase is Python 3.6+ only, I think using `f"strings"` could help alleviate those areas.
        - ... and also, you can factor highly nested structures, declare long strings as module level constants, and stuff like that, which I think help with code quality anyway.

## Alternatives

- The main alternative is YAPF. If the majority here really hate Black's decisions, YAPF still solves the core issue.
- If we like Black except for `"double quotes"`... there is an alternative called [Brunette](https://github.com/odwyersoftware/brunette) which includes a `single-quotes` option.
- Other possibilities are less strict formatters e.g. autopep8. If people desparately still want some formatting flexibility, then this would be a compromise which is better than nothing.

Ultimately, the stakes here aren't actuall _that_ high. If we do not do this, we will just be stuck with the problems I described in the Motivation section.

## Prior art

Black has a [used by](https://github.com/psf/black#used-by) section (which could probably be much bigger!) which includes Facebook, Dropbox, Pytest. Brex (my employer) also uses it successfully.
Google uses YAPF.

## Unresolved questions

- Should we include a pre-commit hook?
- Will we resolve the `git blame` issue? And how?
- If we choose Black, should we go with a line-length of 120?
    - I chose that in my PR because flake8 had been configured that way
    - Since the Databuilder repo seems prone to some particularly long lines, I think 120 probably makes sense; but I'm ambivalent.
- Which repos will this apply to?

## Future possibilities

This ought to be a one-and-done thing. All we will have to do is occasionally update `black` to new versions.

The only aspect of formatting not really covered by Black is import ordering. `isort` does that, and I'm a mild fan, but the tool itself is pretty slow,
and import ordering is a very minor detail.
