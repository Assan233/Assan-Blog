---
title: Git 笔记
date: 2020-06-02 16:28:07
summary: Git 笔记
categories: 笔记
tags: Github
top: 
cover: 
img:
keywords: 
- Git 
- Github
---

## 1.解决每次拉取、提交代码时都需要输入用户名和密码
在~/.gitconfig目录下多出一个文件，用来记录你的密码和帐号
```
git config --global credential.helper store
```
再最后输入一次正确的用户名和密码，就可以成功的记录下来，这是最后一次麻烦啦！
```
git pull
```
## 2. 撤销merge
```
git reset --merge merge前的任何一次提交的hash串
```

## 3. 分支操作
  + 使用命令git branch -a 查看所有分支
  + 使用命令 git push origin --delete Chapater6   可以删除Chapater6 远程分支  
  + 使用命令 git branch -d Chapater8 			可以删除Chapater8本地分支。-D 强制删除
  
## 4. 更新远程分支信息
git remote update -p

