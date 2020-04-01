# Gitflow Part 1: Overview of Gitflow

!!! abstract
    Welcome to part one of a two part series on GitFlow. When you're done reading
    part one, check out part two.

      - [Gitflow Part 1: Overview of Gitflow](gitflow_1.md)
      - [Gitflow Part 2: Step by step guide to Working on a feature](gitflow_2.md)

## Intro

GitFlow is a basic `git` workflow created by Vincent Driessen and described in
his blog post [here](https://nvie.com/posts/a-successful-git-branching-model).
I've been using a modified version of GitFlow for a while now, and thought it
would be a good thing to talk about on this blog.

In part one of this two-part series, I'm going to describe GitFlow and talk
about how it fits into a team's development lifecycle. In part two I'll discuss
working on a new feature, and give a step-by-step breakdown of what commands to
use when working on a new feature.

!!! Disclaimer
    GitFlow isn't for everyone. It works best when your deployment cycle is
    based around releases. If your team deploys to production every day or
    several times a day, GitFlow may be impractical. In this case, [github
    flow](https://githubflow.github.io/) might be a better, more lightweight
    alternative.

## GitFlow Overview

![zoomify](img/gitflow.png){.center} *Fig. 1: Image taken from [here](https://nvie.com/posts/a-successful-git-branching-model).*{.img-caption}
<br>

### Development lifecycle

The basic GitFlow is outlined in Fig. 1. GitFlow assumes that your deployment
lifecycle is like the following:

  * Work is divided into [sprints](https://www.atlassian.com/agile/scrum/sprints)
  * During a sprint, developers will work on a handful of features
  * When all of the sprint's features are finished, they will be packaged into a
    release
  * That release is then deployed to production

GitFlow helps to accomplish this by having designated branches that correspond
to the different stages of this lifecycle.

### Branches

Each repo should have two persistent branches which live forever: `develop` and
`master`.

  * `master` contains the code for the release which is currently deployed
     to production. It should never contain anything which is not released.
     Anything in master, at any given time, is working production code, and
     every commit to `master` is a new release.
  * `develop` contains code for the next release. During a sprint, new features
     are added to `develop` until the sprint ends.

In addition to `develop` and `master`, there are several short-lived branches
which are created for a purpose and then deleted once that purpose is served:

  * **Feature** branches are used to work on a new feature. They are forked off
    of `develop` and then merged back into `develop` and deleted when work on
    the feature is complete.
  * **Release** branches contain all the new features which will be deployed to
    production in a release. A release branch is basically a bunch of features
    which will be merged into `master` at the same time. When a sprint ends, a
    release branch is forked from `develop` and then merged back into `master`
    and `develop`.
  * A **hotfix** is a change that needs to be made immediately. They differ from
    normal bug fixes in that they can't wait for the next release to be deployed
    to production. A hotfix branch is forked from `master` and then merged back
    into `develop` and `master`.

### Starting a project

If you're starting a new project from scratch, you can create a repository for
your project and then clone it to your local. Then create the `develop` branch
off of `master`, and push it to the remote:

``` bash
git clone <repo address> /path/on/local
cd /path/on/local
git checkout -b develop
# Set -u (upstream) so that git remembers what remote to push to on this branch
git push -u origin head
```

### Working on a feature

To work on a new feature, fork a feature branch off of `develop`, as shown in
Fig. 1. Once all coding and testing are done, you can merge the feature branch
back into `develop`.

The specific commands for how to work on a feature branch and merge it back into
`develop` are described in part [two](gitflow_2.md) of this series.

### Releasing features

When you are finished working on all of the features for a release, cut a new
release branch off of `develop`. A good way to name releases is with [semantic
versioning](https://semver.org/):

  * MAJOR version when you make incompatible API changes,
  * MINOR version when you add functionality in a backwards compatible manner, and
  * PATCH version when you make backwards compatible bug fixes.

And a good naming convention for release branches is `release/{release number}`.
So, if you are releasing version `1.2.3`, you can fork a release branch off of
develop:

```sh
git checkout develop
git pull
git checkout -b release/1.2.3
```

!!! note
    As shown in Fig. 1, sometimes after creating a new release, you might notice a
    bug that needs to be fixed before merging the release branch. In this case, you
    can fix the bug with a new commit to the release branch, as shown in the
    diagram.

Then, to actually release the release, merge it into `master` and `develop`
using pull requests (PRs)[^1]: create one PR from `release/1.2.3` into `master`
and create a second PR from `release/1.2.3` into `develop`.

[^1]: A *pull request* is a way to submit code changes for review and request
  that they be merged into a branch. For more info, see
  [here](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests).

Once these PRs are merged, you can then
[tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging) the commits in `master`
with the version number, and then push the tag to the remote:

```sh
git checkout master
git pull
git tag 1.2.3
git push origin 1.2.3
```

### Making a hotfix

To make a hotfix, fork a branch off of `master` and then merge it back into
`develop` and `master`, as shown in Fig. 1.

Since hotfixes are ultimately merged back into `master`, they are also new
releases (since every commit to master is a new release). Therefore, like
release branches, hotfixes should also have a release number.

Further, because hotfixes are generally small changes, the release version
number for a hotfix should usually just increment the patch number of the
version which is currently in `master`. If the change is big enough to increment
the major or minor version number, it probably shouldn't be a hotfix.

Hotfix branches should be named with the naming convention:
`hotfix/{release-number}`. So, if the current version released in `master` is
version `1.2.3`, your hotfix should be version `1.2.4`, and the branch should be
named `hotfix/1.2.4`.

Once the hotfix is completed, follow the same steps for releasing a new release:
merge the hotfix branch into both `develop` and `master` with two PRs, and then
tag the commits with the new release number.

## Conclusion

The above was an overview of my take on GitFlow and how it incorporates into a
full development lifecycle. This is not the only way to do GitFlow; in fact,
I've made a few changes from Vincent Driessen's original
[post](https://nvie.com/posts/a-successful-git-branching-model) on GitFlow.

GitFlow is more of a general workflow and you should feel free to tweak it to
fit your team's needs.

!!! Note
    Now that you've finished part one of this series, take a look at [part
    two](gitflow_2.md).
