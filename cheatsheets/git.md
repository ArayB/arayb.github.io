# Git

#### Merging
`master` is the branch you are merging into so could be another branch name i.e. `feature/guff`.

`branchname` is the branch you are working and that needs merging.
```
git fetch origin

git rebase -i origin/master

gp --force-with-lease branchname

gco master

gm branchname --ff-only

gp

gp origin --delete branchname

git branch --delete branchname

```

#### Untrack a file

`git rm --cached <file path>`