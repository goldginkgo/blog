---
title: Git Workflow in Gitlab
date: 2018-05-20 21:05:01
categories: DevOps
tags:
  - Git
---
A Git Workflow is a recipe or recommendation for how to use Git to accomplish work in a consistent and productive manner.

There are different kinds of Git workflows. But here we suggest the following Git workflow to be used.
<!-- more -->

In the future, we will move towards trunk based development.

### Clone a repository
```
git clone https://<url>.git
cd <project-name>
```

### Create a branch
```
git checkout -b <branch-name>
create a feature branch:  git checkout -b feature/<name>
create a bug-fix branch:  git checkout -b bug/<name>
create a release branch:  git checkout -b release/1.0
create a bug-fix branch from a release branch: git checkout -b release/1.0  release/1.0/<name>
create branch from a tag: git checkout -b <Hotfix branch> <TAG>
```

### Commit to local branch
After code development in our local machine, do proper testing and commit changes to our local branch
[A guideline to commit messages](https://gist.github.com/robertpainsi/b632364184e70900af4ab688decf6f53)

```
git status
git add .
git commit -m "<commit-message>"

Undo a change: https://blog.github.com/2015-06-08-how-to-undo-almost-anything-with-git/
change a commit message https://help.github.com/articles/changing-a-commit-message/
```

### Rebase new branch with master
https://docs.gitlab.com/ee/university/training/topics/merge_conflicts.html)
```
git checkout master
git pull origin master
git checkout <branch-name>
git rebase master
If there are conflicts, fix conflicts in the files.
git add .
git rebase --continue
```

### Push local branch to remote repository
```
git push origin <branch-name>
```

### Create a merge request on master branch

### Review the commit and merge the code if it's ok
If testing for master branch fails, go back to step 3.

### Create [tags](https://docs.gitlab.com/ee/university/training/topics/tags.html) and [releases](https://docs.gitlab.com/ee/workflow/releases.html)

### Future Reading

[Gitflow model](https://nvie.com/posts/a-successful-git-branching-model/)

[Comparing Workflows](https://www.atlassian.com/git/tutorials/comparing-workflows)

[What is Trunk-Based Development?](https://paulhammant.com/2013/04/05/what-is-trunk-based-development/)

[Trunk based development](https://trunkbaseddevelopment.com/)

[Trunk-based Development vs. Git Flow](https://www.toptal.com/software/trunk-based-development-git-flow)
