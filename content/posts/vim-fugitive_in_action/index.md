---
title : 'vim + git = vim-fugitive'
date : 2024-04-09T10:19:30+08:00
draft : true
isCJKLanguage : true
categories:
- Development
tags:
- vim
- git
enableComment : true
---

# vim-fugitive

1. 在vim中，通过`:G`命令直接使用git
2. 通过`fugitive-object`浏览git对象数据库： `:Gwrite` `:Gread`
3. 借助`vim_diff`实现`git diff`：`:Gdiffsplit`
4. git历史: 文件内容、提交信息、提交历史
5. `fugitive-summary`：
6. 各种git操作的映射键：Staging/unstaging, Diff, Commit, Checkout/branch, Stash, Rebase, Navigation(),

0. 场景: `:G blame`了解代码，`:G difftool`查看代码修改，`:G mergetool`处理代码合并冲突

