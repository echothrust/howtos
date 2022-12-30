---
---

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
```
git fetch upstream
git checkout main
git merge upstream/main
```

If there were any new commits, `rebase` your development branch
```
git checkout newfeature
git rebase main
```

## Diffs between branches
```
git diff master:foo foo
git diff <local branch> <remote>/<remote branch>

```

## Reset fork based on upstream
```
git fetch -p upstream
git checkout master
git reset --hard upstream/master
git push origin master --force
```

## Reset your upstream repository with current
https://stackoverflow.com/a/24768381

## Pushing an existing repo with history to a new blank repo
```
git remote set-url origin https://new.repo/url
git push origin master --force
# or
git push origin master --force
```