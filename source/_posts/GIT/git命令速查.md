---
title: git命令速查
date: 2020-03-30 09:08:46
tags: git
---

统一概念：
* 工作区：改动（增删文件和内容）
* 暂存区：输入命令：`git add`改动的文件名，此次改动就放到了暂存区
* 本地仓库(简称：本地)：输入命令：`git commit 此次修改的描述`，此次改动就放到了本地仓库，每个`commit`，我叫它为一个版本。
* 远程仓库(简称：远程)：输入命令：`git push 远程仓库`，此次改动就放到了远程仓库（GitHub 等)

## 配置用户信息
新安装git后，需要先配置下用户信息。
```
git config --global user.name "username"
git config --global user.email "xxx@xx.com"
```
## 展示所有configs
config的三个作用域
```
git config --local // local只对某个仓库有效
git config --global // global对当前用户所有仓库有效
git config --system // system对系统所有登录的用户有效
```
```
git config --list // 所有配置
git config --list --local // 当前仓库
git config --list --global // 全局
git config --list --system // 系统
```
## 建本地git仓库
两种场景：
1. 把已有的项目代码纳入git管理
```
cd 项目所在文件夹
git init
```
2. 新建的项目直接用git管理
```
cd 某个文件夹
git init your_project // 会在当前路径下创建和项目名称同名的文件夹
cd your_project
```

# github高级搜索
```
created:<2020-03-29
git in:readme
git stars:>1000
git in:readme stars:>1000
```