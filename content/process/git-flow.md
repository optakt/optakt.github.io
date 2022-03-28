# Git

This guide assumes that you are already familiar with the basic functionalities of `git`:

* Cloning repositories
* Checking branches out
* Committing changes
* Pushing
* Pulling

And it will walk you through slightly more advanced concepts that will come in handy when working on Optakt projects.

## Rebasing

The most important thing you'll likely need to master is _rebasing_.
Whenever the base branch one of your pull requests is against gets updated with changes that conflict with yours, you will need to rebase your branch before it can be properly reviewed and merged.

!!! warning "git merge"

    We do not use `git merge` at Optakt, because it would result in a messier commit history.
    Rebasing, and squashing commits before merging pull requests ensures that when looking at master, each commit corresponds to a single pull request.

1. Fetch the latest version of the base branch you need to rebase against. In most cases this will be `master`.
2. Run `git rebase <base branch>` from your local branch.
3. This will make `git` rewind history and put your HEAD back to the first commit that diverges between the two branches.
4. One by one, commits will be automatically applied to your local branch until one with conflicts is reached.
5. You will now need to resolve the conflicts
   1. This can be done with a GUI using most development environments, or manually by running `git status` to see which files have conflicts that need to be resolved and modifying them manually.
6. Now, run `git rebase --continue` to continue rebasing.
7. Once the final commit is reached, you can run `git push --force` to overwrite the previous version of your branch with the new rebased one.

??? hint "If you made a mistake"

    If you made a mistake and overwrote your remote and local branches with one that does not work, no worries!
    You can always use [`git reflog`](#git-reflog) to get back to the previous state of your branch, before you started rebasing.

For more details on `git  rebase`, please read the [git manual](https://git-scm.com/book/en/v2/Git-Branching-Rebasing).

### Alternative to Rebasing

In some cases, if the base branch was modified by force and now has tons of conflicts that are not related to your own changes, you might be better off not rebasing, so that you do not have to deal with resolving many conflicts you are unsure of.

In that situation, you can follow these simple steps:

1. `git log` on your branch to know the hashes of your commits
2. `git checkout <base branch>`
3. `git pull`
4. `git checkout -b tmp`
5. `git cherry-pick <commit hash>` for each of your commits
6. `git branch -D <your branch>` to remove your local branch
7. `git checkout -b <your branch>` to use the contents of the now clean `tmp` branch as your new local branch
8. `git push --set-upstream=<remote> <your branch> --force` to overwrite the remote version of your branch with the clean one

If there was a pull request that used your branch, it will now automatically be updated and no longer have any conflicts.

??? hint "If you made a mistake"

    If you made a mistake and overwrote your remote and local branches with one that does not work, no worries!
    You can always use [`git reflog`](#git-reflog) to get back to the previous state of your branch, before you deleted anything.

## Git Reflog

Reference logs, or "reflogs", record when the tips of branches and other references were updated in the local repository.
This is very useful for us, in order to revert changes that were made by mistake and to get back to the previous state of a modified branch.

You can use `git reflog show` to look at your reference logs and find the hashes of the references you are interested in, and then simply `git checkout <hash>` to get back to their state. From there, you can use `git checkout -b <branch name>` to create a branch from that point in history.

## Git Restore

`git restore` is a relatively new command in `git` which allows you to restore files from any point in history and even from other branches.
This can be very useful for example if you want a specific file from a specific commit on another branch, you don't need to `cherry-pick` the whole commit, but instead of you run `git restore --source=<other branch> -- /path/to/file`.

## Clean Commits

Ideally, we want to keep the commit history of our projects clean, in order to make it easy to navigate, to track and understand changes.
On the master branch, each commit should correspond to a single PR, and within each PR, ideally each commit should correspond to a single change, in a way that reviewing a PR commit-by-commit should make sense.

This is not simple however, since it means that if you go ahead and code huge amounts at once, you will then need to either:

* Create multiple commits from these changes, by [interactively adding](#interactive-add) the specific parts that belong together into one commit for each feature/component of the changes.
* Create one huge commit for now, and [split it later on](#amending-commit-history) by using `git rebase` and amending the commit history.

### Interactive Add

When using `git add -i`, you have access to an interface that lets you go through the changes on your local branch and choose whether to add them to staging or not, block-by-block.
This means you do not need to add a whole file to your next commit, but instead can add only the parts of it that are relevant to that commit.

### Amending Commit History

The other solution is to use `git rebase -i <base branch>` to amend the commit history.
In order to split a commit into multiple parts, you need to specify the `edit` directive for that commit.
Once you get to that point in the rebasing process, running `git status` will show you all the changes from that commit.
You can then use `git restore --staged <file names>` to remove them from staging.
You are now free to run `git add -i` or use your IDE to create the relevant commits from those changes, and to create new commits.
When you are done, simply run `git rebase --continue` to continue the rebasing process where we left it.
