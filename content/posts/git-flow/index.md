---
title: "The Git Flow Workflow"
date: 2020-11-15T03:05:11Z
draft: false
isCJKLanguage : true
tags:
    - Git
---

### 分支类型

Main: 生产主分支，不直接提交代码，由Hotfix或Release分支合并过来；

Hotfix: 补丁分支，基于Main分支克隆，用于对线上版本BUG修复，完成后合并到Main分支和Develop分支；

Develop: 主开发分支，基于Main分支克隆，不直接提交代码，从Feature分支合并；

Feature：功能开发分支，基于Develop主开发分支克隆，开发完后合并到Develop分支，用于新功能开发，可同时存在多个；

Release：预发布分支，Feature分支合并到Develop分支后，在克隆出Release分支进行测试，修改BUG直到测试通过后，合并到Main分支打上tag，同时也合并到Develop分支；

### 开发流程

![avatar](git-flow.svg)


main分支、develop分支一直存在；

feature分支、release分支、hotfix分支在需要时创建，在完成时删除；

#### 创建develop分支

在main分支下，创建develop主开发分支，并推送到远程仓库。

```bash
# 创建分支
git checkout -b develop
# 推送到远程仓库
git push -u origin develop
```

#### feature开发

开发新feature的时候，先基于develop分支创建一个feature分支；

接着在feature分支上进行开发；

开发完成后，切换回develop分支，拉取develop分支，看是否有其他同事的修改；

然后merge自己feature分支的改动到develop分支，并推送到远程仓库；

最后把feature分支删除。

```bash
# 基于develop分支，创建feature-test分支
git checkout -b feature-test develop
# 推送feature-test分支到远程仓库
git push -u origin feature-test
# 在feature-test分支上开发，并提交
git status
git add .
git push
git commit   
# 切换为develop分支
git checkout develop
# 拉取develop分支的修改 
git pull origin develop
# 在develop分支上merge feature-test分支的修改
git merge --no-ff feature-test
# 将develop推送到远程仓库
git push origin develop
# 删除feature-test分支
git branch -d feature-test
# 删除远程仓库的feature-test分支
git push origin --delete feature-test  
```

##### release预发布

```bash
# 基于develop分支创建release分支
git checkout -b release-test develop
# 进行测试、修复bug、提交代码
...test and bug fixes and commit
# 切回main分支
git checkout main
# merge release分支相关改动
git merge --no-ff release-test
# 打上tag
git tag -a 0.1
# 切回develop分支
git checkout develop
# merge release分支相关改动
git merge --no-ff release-test
# 删除release分支
git branch -d release-test
git push origin --delete release-test  
```

##### hotfix补丁

```bash
# 基于main分支创建hotfix分支
git checkout -b hotfix-0.1.1 main  
# 修复bug且提交相关修改
...bug fixes and commit
# 切换回main分支
git checkout main
# merge hotfix分支的改动
git merge --no-ff hotfix-0.1.1
# 打上tag
git tag -a 0.1.1
# 切换回develop分支
git checkout develop
# merge hotfix分支的改动
git merge --no-ff hotfix-0.1.1
# 删除hotfix分支
git branch -d hotfix-0.1
git push origin --delete  hotfix-0.1.1
```
