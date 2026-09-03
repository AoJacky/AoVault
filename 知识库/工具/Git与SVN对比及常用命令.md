---
title: "Git与SVN对比及常用命令"
date: 2026-07-30
tags: [知识库, 工具, Git, SVN, 版本管理]
source: "云加社区/strongerHuang"
---

# Git与SVN对比及常用命令

> Git vs SVN核心区别、Git工作流概念、Git-SVN桥接命令、Git常用命令速查。

## 摘要

Git与SVN三大区别（分布式vs集中式、复杂vs简单、廉价分支vs昂贵分支），Git工作区/暂存区/仓库区核心概念，Git-SVN桥接命令，以及初始化/配置/增删/分支/提交/拉取/撤销/查询常用命令。

## 原文要点

### Git vs SVN三大区别
1. **分布式vs集中式**：Git支持离线工作，SVN必须联网
2. **复杂概念多vs简单易上手**：Git命令多功能多，SVN对新手友好
3. **分支廉价vs昂贵**：Git分支是指针指向提交，切换迅速；SVN分支是目录拷贝

### Git核心概念
- **工作区(Workspace)**：实际目录
- **暂存区(Index)**：临时保存改动
- **仓库区(Repository)**：本地仓库+远程仓库
- 提交流程：git add → git commit → git push

### Git-SVN桥接
- `git svn clone -s [repo]` 下载SVN项目
- `git svn rebase` 取回并变基合并
- `git svn dcommit` 上传到SVN远程

### 常用命令速查
- 初始化：git init / git clone
- 配置：git config --global
- 增删：git add / git rm / git mv
- 分支：git branch / git checkout -b / git merge / git cherry-pick
- 提交：git commit -m / git commit --amend
- 拉取：git fetch / git pull / git pull --rebase
- 撤销：git checkout / git reset / git revert / git stash
- 查询：git status / git diff / git log / git show

## 原文链接

https://mp.weixin.qq.com/s/YlnJ_9jZ2S4ZH6DQx8eJbQ
