---
title : 'Git 笔记'
date : 2024-04-10T10:39:20+08:00
draft : true
isCJKLanguage : true
categories:
- Development
tags:
- git
enableComment : true
---

# Git

> [Git](https://en.wikipedia.org/wiki/Git) is a distributed version control system that tracks changes in any set of computer files...

分布式、版本控制系统(跟踪文件的变化)

## git object model

把Git看作一个键值数据库。

键是40位的哈希(文件内容计算，不可能冲突)，也就是git对象名:
好处：通过对象名可以快速区分两个对象是不是同一个；相同内容，相同对象名；修改内容，则hash会改变；

值则是git对象，有blob, tree, commit, tag四种类型，都是文件：
1. blob: 文件
2. tree: 像文件夹，指向其他文件或树
3. commit: 指向一棵树，表示某个时间点的项目(存储所有文件)；commit并不包含文件的变化，而是通过比较其指向树的内容而得到
4. tag: 标记特定的commit


2. `working directory/working tree`, `index/staging area`, `local repo`, `remote repo`
3. normal workflow: `git add`, `git commit`
4. distributed workflows: `git clone`, `git push`, `git pull`

5. branch and merging
6. commit history: `git log`
7. compare commit: `git diff`

8. cherry-pick and rebase
9. stash
10. undo: `git reset`, `git checkout`, `git revert`
11. finding files with words or phrases: `git grep`

12. `git blame`
13. `git bisect`

14. git hooks
