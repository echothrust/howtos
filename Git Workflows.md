---
---
- [Git Workflows](#git-workflows)
  - [Forks workflow](#forks-workflow)
  - [Cleaning Up Your Work](#cleaning-up-your-work)
  - [Diffs between branches](#diffs-between-branches)
  - [Pull a single file from head](#pull-a-single-file-from-head)
  - [Reset fork based on upstream](#reset-fork-based-on-upstream)
  - [Reset your upstream repository with current](#reset-your-upstream-repository-with-current)
  - [Pushing an existing repo with history to a new blank repo](#pushing-an-existing-repo-with-history-to-a-new-blank-repo)
  - [List changed files](#list-changed-files)
  - [Using worktree](#using-worktree)
  - [Maintenance and Repairing](#maintenance-and-repairing)

# Git Workflows
## Forks workflow
1. Go to the original project and click fork
2. Clone your fork `git clone git@github.com:username/repo.git`
3. Add an `upstream` remote for the original repository `git remote add upstream https://github.com/origuser/repo.git`.
4. Checkout a new branch `git checkout -b BRANCH_NAME`
5. Make desired changes to the local repository on this branch.
6. Pull new changes from remote
```
git checkout main
git pull upstream main
git checkout BRANCH_NAME
git merge main
```
7. Push changes to your remote repository `git push origin BRANCH_NAME`.
8. Open a pull request on GitHub merging your changes with `upstream` (original) repository.

Once the pull request is accepted, you’ll want to pull those changes into your origin (forked repository).
1. Change to main `git checkout main` 
2. pull the changes `git pull upstream main`.
3. delete your branch using the GitHub website or local `git branch -d BRANCH_NAME`
4. delete the remote `git push origin --delete BRANCH_NAME`

## Cleaning Up Your Work
Prior to submitting your pull request, you might want to do a few things to clean up your branch and make it as simple as possible for the original repo's maintainer to test, accept, and merge your work.

If any commits have been made to the upstream main branch, you should `rebase` your development branch so that merging it will be a simple fast-forward that won't require any conflict resolution work.

Fetch upstream main and merge with your repo's main branch
```bash
git fetch upstream
git checkout main
git merge upstream/main
```

If there were any new commits, `rebase` your development branch
```bash
git checkout newfeature
git rebase main
```

## Diffs between branches
```bash
git diff master:foo foo
git diff <local branch> <remote>/<remote branch>
```

## Pull a single file from head
```bash
git checkout FETCH_HEAD -- {file}
```


## Reset fork based on upstream
```bash
git fetch -p
git fetch -p upstream
git checkout master
git reset --hard upstream/master
git push origin master --force
```

## Reset your upstream repository with current
https://stackoverflow.com/a/24768381

## Pushing an existing repo with history to a new blank repo
```bash
git remote set-url origin https://new.repo/url
git push origin master --force
```

or if origin is the same
```bash
git push origin master --force
```

alternatively to keep both existing (as `upstream`) and new (as `origin`)
```bash
git remote rename origin upstream
git remote add origin URL_TO_ORIGIN_GITHUB_REPO
git push origin master
```

## List changed files
Ref: https://stackoverflow.com/a/18957885
```
git diff-tree -r --name-only --no-commit-id ORIG_HEAD HEAD
```

or

```
git diff-tree -r --name-only --no-commit-id HEAD@{1} HEAD
```

In case of `git pull --ff-only` command, when many commits can be added, `HEAD@{1}` (inside post-merge hook) gives the last commit before this command, while `ORIG_HEAD` gives just `HEAD^` commit.

## Using worktree
```
git --work-tree=/var/www/html --git-dir=/usr/src/proj checkout -f
```

## Maintenance and Repairing
```
git fsck --full
git gc --prune=now
```

## Git LFS
```
git lfs migrate import --no-rewrite pattern
```

or 
```
git lfs track pattern
git add --renormalize . 
```
