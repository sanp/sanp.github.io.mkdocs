# {{ posts.gitflow.part2_title }}

!!! abstract
    Welcome to part two of my series on GitFlow. If you're just getting
    here, check out part one first. 

      - [{{ posts.gitflow.part1_title }}](gitflow_1.md)
      - [{{ posts.gitflow.part2_title }}](gitflow_2.md)

## GitFlow and Jira

Before going through the step by step guide, I want to mention how GitFlow fits
into the larger project management context.

GitFlow works well in conjunction with a ticket tracking software. There are
plenty of tracking tools out there, such as
[Jira](https://www.atlassian.com/software/jira) or
[Trello](https://trello.com/en-US). Whatever tool you use, though, each feature
you work on should correspond to a single ticket which is tracked in your tool
of choice.

In Jira, for example, tickets are organized into
[boards](https://confluence.atlassian.com/jirasoftwarecloud/what-is-a-board-764477964.html),
which are areas where a team can keep track of all of its open tickets. When
working on a new feature, I like to create feature branches with the naming
convention: 

    feature/<board_name>-<ticket_nunmber>_<ticket_description>

For example, say you have a Jira board named `ABC`. If you're working on
implementing a "cool new feature," you can make a ticket in your `ABC` board
named "Cool New Feature," and then create a feature branch named
`feature/ABC-101_cool_new_feature`.

## Working on a new feature: step by step guide

In Vincent Driessen's original
[post](https://nvie.com/posts/a-successful-git-branching-model) on GitFlow, the
basic process he described for working on a feature can be broken down into four
main steps:

  1. Create a feature branch off of `develop`
  2. Work on that branch and push it to the remote repo
  3. Merge the feature branch into `develop`
  4. Clean up your branches

This process works great for small features, but in my experience, it has one
main drawback: it makes keeping a clean commit history more difficult.

Ideally, before merging any branch, you should clean up your commit history by
following the steps in my interactive rebasing [post](rebasing.md). However,
because rebasing involves rewriting history, it's best not to rebase commits on
branches that have already been pushed to a remote repository.

This presents a slight problem when using GitFlow. If you're working on a
feature branch that takes several days or even weeks to complete, you'll
naturally want to push your changes to a remote periodically to ensure you don't
lose any work if your laptop crashes. However, if you do this, then rebasing is
trickier since you would be rewriting published history.

The best solution I've found is to do all of your feature work on an
intermediate individual branch first, and then merge that branch into your
feature branch. The basic workflow then becomes slightly more involved:

  1. Create a feature branch off of `develop`
  2. Create an individual branch off of the feature branch
  3. Work on the individual branch, pushing to the remote often to avoid losing
     history
  4. When work on the feature is complete, merge the individual branch into the
     feature branch
  5. Then merge the feature branch into `develop`
  6. Clean up your branches

In the next sections, I'm going to go over each of these steps, and show the
commands necessary to execute them.

### Step 1: Create a feature branch

Let's say you're a developer working on the feature `ABC-101_cool_new_feature`.
Create a feature branch from `develop` and then push the branch:

``` bash
git checkout develop
# Make sure you have the latest changes to develop
git pull
git checkout -b feature/ABC-101_cool_new_feature
# Set -u (upstream) so that git remembers what remote to push to on this branch
git push -u origin head
```

### Step 2: Create an individual branch off of the feature branch

To make your intermediate branch, fork a branch off of the feature branch you
just created. For the individual branches, I like to use the naming convention:

    <your-name>/<feature-branch-name-after-slash>

So, if Alice is working on the `feature/ABC-101_cool_new_feature` branch, then
she would make an individual branch named `alice/ABC-101_cool_new_feature`:

``` bash
# Make sure you're on the feature/ABC-101_cool_new_feature branch
git checkout feature/ABC-101_cool_new_feature branch
git checkout -b alice/ABC-101_cool_new_feature
git push -u origin head
```

### Step 3: Work on the individual branch and push to remote often

Now that Alice has her individual branch set up, she can work on it and push to
the remote as often as she likes. I usually make several commits per day and
push to the remote once at the end of each day. But you can feel free to push as
often as you like: you never know when your laptop will crash and you'll lose
important work!

``` bash
# Do work on the feature and continuously commit changes on the individual
# branch and push to the remote
git add -A
git commit -m 'My commit message 1'
# Do some more work
git add -A
git commit -m 'My commit message 2'
git push
```

### Step 4: Merge the individual branch back into the feature branch

Once all work is done on the feature, commit your changes and push to the remote
copy of your individual branch one last time. The next step is to merge your
individual branch into the feature branch.

Before merging, though, you'll want to make sure you have a clean commit
history. Your individual branch likely has tens of commits at this point, many
of which have unhelpful commit messages such as "Fixed a typo" or "Made a small
update." Rather than keep all of these as separate commits, you should squash
these into one or a few more meaningful commits before merging. To do this, you
can follow the instructions in my [blog post](rebasing.md) on interactive
rebasing.

Once you have cleaned your commit history, then merge the individual branch
into the feature branch. You'll also want to merge in `develop` to the feature
branch as well, to make sure there are no merge conflicts between your feature
code and what's already pushed in `develop`.

``` bash
git checkout feature/ABC-101_cool_new_feature
git pull
git merge alice/ABC-101_cool_new_feature
git merge develop
git push
```

!!! Note
    You may need to resolve some merge conflicts at this stage. I recommend
    [DiffMerge](https://sourcegear.com/diffmerge/) for this.

### Step 5: PR from feature branch into develop

Step five is to merge the feature branch into `develop`. To merge the feature
branch, you can use a [pull
request](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)
(PR). A PR is a way to submit code changes for review and request that they be
merged into a branch.

Once the PR is approved, merge it and then delete the feature branch on the
remote.

### Step 6: Clean up local branches

Once the PR is merged, then work on the feature is almost complete. The last
step is to clean up your local branch list by deleting the old branches.

!!! Note
    You can use `git fetch -p` to prune your
    remote branch list to remove any remote branches you're tracking that no longer
    exist.

``` bash
git checkout develop
git pull
git branch -d alice/ABC-101_cool_new_feature
git branch -d feature/ABC-101_cool_new_feature
# Prune the inactive remote branches
git fetch -p
```

The feature is now complete.

## Conclusion

The above shouldn't be taken as the definitive guide or the only way to do
GitFlow. This is just a method of working on features that has worked well for
me in the past. Depending on the circumstance, you might find yourself wanting
to adjust this workflow. For example, if you have a very small feature which
will only take a day or two, you can of course skip creating an individual
branch to work on and just work directly on the feature branch.

Like anything related to workflow, you should design a workflow that fits your
needs, not shoehorn your needs into a particular workflow.
