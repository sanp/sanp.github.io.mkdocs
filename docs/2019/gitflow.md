# Using GitFlow to improve your git workflow

## Intro

GitFlow is a basic `git` workflow created by Vincent Driessen and described in
his blog post [here](https://nvie.com/posts/a-successful-git-branching-model).
I've been using a modified version of GitFlow for a while now, and thought it
would make a good first blog post.

In this post, I'm going to describe GitFlow and talk about a couple additional
things I do when using it.

!!! Disclaimer
    GitFlow isn't for everyone. It works best when your deployment cycle is
    based around releases. If your team deploys to production every day or
    several times a day, GitFlow may be impractical. In this case, [github
    flow](https://githubflow.github.io/) might be a better, more lightweight
    alternative.

## Basic GitFlow

![zoomify](img/gitflow.png){.center} *Fig. 1: Image taken from [here](https://nvie.com/posts/a-successful-git-branching-model).*{.img-caption}
<br>

### Development lifecycle

The basic GitFlow workflow is outlined in Fig. 1. GitFlow assumes that your
deployment lifecycle is like the following:

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
    of `develop` and then merged back into `develop` and deleted when complete.
  * **Release** branches contain all the new features which will be deployed to
    production in a release. A release branch is basically a bunch of features
    which will be merged into `master` at the same time. When a sprint ends, a
    release branch is forked from `develop` and then merged back into `master`
    and `develop`.
  * A **hotfix** is a small change that needs to be made immediately and
      requires minimal or no testing. A hotfix branch is forked from `master`
      and then merged back into `develop` and `master`.

## Working with gitflow: step by step code walkthrough

I will now describe what commands you can use to follow GitFlow with your own
projects.

### Starting a project

#### Creating a new repo

If you're starting a new project from scratch, you can create a repo for your
project and then clone it to your local. Then create the `develop` branch off of
`master`, and push it to the remote:

``` bash
git clone <repo address> /path/on/local
cd /path/on/local
git checkout -b develop
git push -u origin head
```

#### Working with an existing repo

If a repo for your project already exists, clone the repo to your local:

``` bash
git clone <repo address> /path/on/local
```

### Working on a new feature

Each feature should correspond to a single ticket in
[Jira](https://www.atlassian.com/software/jira), or whatever software you use to
track tickets. All code changes for that ticket should be contained in a single
feature branch with the naming convention:
`feature/<board_name>-<ticket_nunmber>_<ticket_description>`[^1].

[^1]: In Jira, tickets are organized into *boards*, which are areas where a team
  can keep track of all of its open tickets. For more information, see
  [here](https://confluence.atlassian.com/jirasoftwarecloud/what-is-a-board-764477964.html).

Let's say your team uses a Jira board named `ABC` In that case, an example
feature branch name would be: `feature/ABC-2701_cool_new_feature`. All Jira
tickets should have their own feature branch so that feature branches contain
just the minimum code changes needed to close out their corresponding ticket.

The following is a step-by-step guide to working on a new feature.

#### One developer working on a feature

##### Step 1: Create a feature branch

Let's say you're the only developer working on the feature
`ABC-2701_cool_new_feature`. Create a feature branch from `develop` and then
push the branch:

``` bash
git checkout develop
# Make sure you have the latest changes to develop
git pull
git checkout -b feature/ABC-2701_cool_new_feature
# Set -u (upstream) so that git remembers what remote to push to on this branch
git push -u origin head
```

##### Step 2: Work on the feature and push to remote

Once all work is done on the feature, commit your changes and push to the remote
repository. 

!!! Note
    Before pushing, it's a good idea to make sure you have a clean commit
    history. Follow the instructions in my [blog post](rebasing.md) on
    interactive rebasing to ensure that your commit history is clean before
    merging.

``` bash
# Make sure you're on the feature branch
git checkout feature/ABC-2701_cool_new_feature
# Pull in latest changes
git pull
# Ensure your changes are compatible with current develop before pushing
git merge develop
# No need to specify origin head if you did -u earlier
git push
```

Note that you may need to resolve some merge conflicts at this stage. I
recommend [DiffMerge](https://sourcegear.com/diffmerge/) for this.

##### Step 3: PR from feature branch into develop

To merge the feature branch, open a *pull request* (PR)[^2] into `develop`. Once the
PR is approved, merge it and then delete the feature branch on the remote.

[^2]: A *pull request* is a way to submit code changes for review and request
  that they be merged into a branch. For more info, see
  [here](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests).

##### Step 4: Clean up local branches

Finally, clean up your local branches:

``` bash
git checkout develop
git pull
git br -d feature/ABC-2701_cool_new_feature
# Prune the inactive remote branches
git fetch -p
```

The feature is now complete.

#### Multiple developers working on a feature

If more than one developer is working on the same feature, the process is only
slightly more complicated than if only one developer is working on the feature.

Let's say Alice, Bob, and Carol are all working on the feature from above. The
following is how they would each work together.

##### Step 1: Create a feature branch

Same as in the one developer scenario, make a feature branch from `develop`.

``` bash
git checkout develop
git pull
git checkout -b feature/ABC-2701_cool_new_feature
git push -u origin head
```

##### Step 2: Each developer creates an individual branch

Rather than all work on the main feature branch, Alice, Bob, and Carol should
all create their own individual branches to work on from the main feature
branch. This way, all of their changes are separated.

On Alice's local:

``` bash
git checkout feature/ABC-2701_cool_new_feature
git pull
git checkout -b alice/ABC-2701_cool_new_feature
git push -u origin head
```

On Bob's local:

``` bash
git checkout feature/ABC-2701_cool_new_feature
git pull
git checkout -b bob/ABC-2701_cool_new_feature
git push -u origin head
```

On Carol's local:

``` bash
git checkout feature/ABC-2701_cool_new_feature
git pull
git checkout -b carol/ABC-2701_cool_new_feature
git push -u origin head
```

##### Step 3: Individual work and push to remote

Each developer can work on their individual branch. Once a developer is done
with their share of the work, they should merge their individual branch into the
main feature branch with a PR, while making sure all merge conflicts are
resolved. So, if Alice finishes her portion of the ticket, she would do:

``` bash
# Make sure she's on her individual branch
git checkout alice/ABC-2701_cool_new_feature
# Pull in latest changes
git pull
# Merge in the latest changes from the other developers and from develop
git merge bob/ABC-2701_cool_new_feature
git merge carol/ABC-2701_cool_new_feature
git merge develop
git push
```

As in the earlier case, make sure all merge conflicts are resolved before
pushing.

!!! Note
    Just as in the one developer case, at this stage it is a good idea to rebase
    your commit history before pushing to the remeote. Follow the instructions
    [here](rebasing.md) to do that.

##### Step 4: PR from individual branch into main feature branch

Once all conflicts are done and merges complete, Alice should create a PR for
her feature branch into the main feature branch. Once the PR is approved, merge
it, and then delete the individual branch on the remote.

##### Step 5: Repeat for other developers

Bob and Carol should repeat steps 3 and 4 for their individual branches. 

##### Step 6: PR from feature branch into develop

When all individual branches are merged into the main feature branch, the next
step is to merge the feature branch into develop. First, ensure that any updates
to develop are pulled into the feature branch:

``` bash
git checkout feature/ABC-2701_cool_new_feature
git pull
git merge develop
git push
```

Then create a PR from `feature/ABC-2701_cool_new_feature` into `develop`. When
that is approved, merge the PR and then delete the feature branch from the
remote repo.

##### Step 7: Clean up local branches

Finally, clean up your local branches:

``` bash
git checkout develop
git pull
git br -d feature/ABC-2701_cool_new_feature
# Prune the inactive remote branches
git fetch -p
```

The feature is now complete.

## Cutting a release branch

When you are finished working on all of the features for a release, cut a new
release branch off of `develop`, then merge it into `master` and `develop`.
Branches are named according to [semantic versioning](https://semver.org/):

  * MAJOR version when you make incompatible API changes,
  * MINOR version when you add functionality in a backwards compatible manner, and
  * PATCH version when you make backwards compatible bug fixes.

``` bash
git checkout develop
git pull
git checkout -b release/1.0.0
git push -u origin head
```

Then merge the branch into `master` with a PR, and delete the release branch
in the remote and locally:

``` bash
git checkout develop
git pull
git br -d release/1.0.0
git fetch -p
```

You have now released a new release.

## Appendix

### One developer optional variation: using an individual branch

In the above section on GitFlow for [one
developer](#one-developer-working-on-a-feature), it calls for that developer to
create a feature branch, work on it, and then eventually merge that feature
branch back into `develop` with a PR. 

Ideally, before a branch is merged, its commit history should be clean by
following the steps in my interactive rebasing [post](rebasing.md). However,
because rebasing involves rewriting history, it's best not to rebase commits on
branches that have already been pushed to a remote repository.

This leaves two options. The first option is to do all work on your feature
branch locally and only push to the remote once work on the branch is complete
and the commit log has been rebased locally. This is okay for features that do
not take a long time to complete; however, for features that take many days to
complete, it is not ideal.

A better approach is to create an individual branch, forked off of the feature
branch, and work on that branch until development on the feature is complete.
You can commit to this individual branch as much as you like and push
continuously to the remote repo, ensuring that no work is ever lost.

When development work is complete, you can then locally rebase your individual
branch's commits, merge the individual branch into the feature branch (not using
a PR), and then merge the feature branch into develop using a PR. The individual
branch can then be deleted. 

So, for one developer named Alice, the full workflow would be as follows:

``` bash
git checkout develop
git pull
git checkout -b feature/ABC-2701_cool_new_feature
git push -u origin head
git checkout -b alice/ABC-2701_cool_new_feature
git push -u origin head
## Do work on the feature and continuously commit and push changes on the
# individual branch to the remote repo
git add -A
git commit -m 'My commit message 1'
# Do some more work
git add -A
git commit -m 'My commit message 2'
# Finally, when all work is done, rebase and merge. See the section below on
# [interactive rebasing](#interactive-rebasing-ensuring-a-clean-commit-history) 
git rebase -i HEAD~2
# After finishing the rebase, switch to the feature branch and merge
git checkout feature/ABC-2701_cool_new_feature
git pull
git merge alice/ABC-2701_cool_new_feature
git push
git br -d alice/ABC-2701_cool_new_feature
```