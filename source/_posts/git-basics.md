---
title: Git Basics
date: 2017-03-24 19:03:01
categories: DevOps
tags:
  - Git
---
### What is Git?
Distributed version control system and source code management system.
Git can keep track of changes.
<!-- more -->

### GIT vs SVN?
SVN uses a central server to store all files and enables team collaboration.
If the central server goes down, no one can collaborate at all.

Git checkout fully mirror the repository.
If the server goes down, then the repository from any client can be copied back to the server to restore it. And we can perform many operations when you are offline.

### Advantages of GIT?
Free and open source
Fast and small ( most of the operations are performed locally)
Implicit backup ( Data on any client side mirrors the repository)
Easier branching (CVCS will copy all the codes to the new branch, so it is time-consuming)

### How can conflict in git resolved?
To resolve the conflict in git, edit the files to fix the conflicting changes and then add the resolved files by running "git add" after that to commit the repaired merge,  run "git commit".  Git remembers that you are in the middle of a merger, so it sets the parents of the commit correctly.

### What is git rebase and how can it be used to resolve conflicts in a feature branch before merge?
In simple words, git rebase allows one to move the first commit of a branch to a new starting location. For example, if a feature branch was created from master, and since then the master branch has received new commits, git rebase can be used to move the feature branch to the top of master. The command effectively will replay the changes made in the feature branch at the top of master, allowing conflicts to be resolved in the process. When done with care, this will allow the feature branch to be merged into master with relative ease and sometimes as a simple fast-forward operation.

### Git workflow (the following may not fully correct)
clone the Git repository as a working copy
modify the working copy by adding/editing files
update the working copy by taking other developer's changes
review the changes before commit.
commit changes. If everything is fine, then you push the changes to the repository
if you realize something is wrong, then you correct the last commit and push the changes to the repository
  1. Create a branch of Master
  2. Work
  3. Merge it back to Master when done

Working Directory -> Staging Area -> Repository
```
git init
git status(inspects the contents of the working directory and staging area)
git add file1 file2
git add -A
git diff( shows differences between the working directory and the staging area)
git diff --staged( Shows the changes between HEAD (latest commit on current branch) and staging directory)
git diff HEAD(Shows the deltas between HEAD and working dir )
git commit -m ""
git log
git show HEAD
```

### backtrack in Git
```
git checkout HEAD filename (Discards changes in the working directory.)
git reset HEAD filename (Unstages file changes in the staging area.)
git reset commit_SHA (reset to a previous commit in your commit history.)
git revert HEAD
git commit --amend
```

### Git branch
```
git branch
git branch branch-name
git checkout branch-name
git checkout -b branch_name
git merge branch-name
git branch -d branch_name
git rebase
```

### Git teamwork
A remote is a shared Git repository that allows multiple collaborators to work on the same Git project from different locations.
```
git clone remote_location clone_name
git remote -v
git fetch
git merge orgin/master
git push origin your_branch_name
```

[Top 45 GIT Interview Questions & Answers](http://career.guru99.com/top-40-interview-questions-on-git/)

[Git 使用规范流程](http://blog.jobbole.com/88955/)
