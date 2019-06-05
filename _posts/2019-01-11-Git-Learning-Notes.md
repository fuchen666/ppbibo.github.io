---
layout: post
title: "Git Learning Notes"
subtitle: "Git Learning Notes"
date: 2019-01-11
author: ppbibo
category: Notes
finished: true
---
[TOC]

# GIT 学习笔记

Git 学习笔记

git是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目.



[![2](./GIT 学习笔记 _ bibo @ 安全小生_files/2.png)](/static/img/2.png)

### 正文

```
git config --global user.name "yourusername"
git config --global user.email youremail



git config --list //查看配置信息
git config user.name
git config user.email



git init // 创建版本库

git add . // 将此目录的文件添加到暂存库

git add <file> // 指定文件添加到暂存库

git commint -m "first commit" // 将暂存库的文件提交到仓库

git commint -m "xxxxxxx"

git status // 查看仓库当前的状态

git diff // 查看对比两次文件内容具体修改了什么内容（默认工作区与暂存取做比较）

git diff HEAD -- <filename> // 查看工作区和版本库里面最新版本的区别

git log // 查看历史记录

git log --pretty=oneline // 指出commit id



HEAD // 指定当前版本

git reset --hard HEAD^ //当前版本回退到上一个版本

git reset --hard HEAD^^ //回退到上两个版本

git reset --hard HEAD~100 // 回退上一百个版本



git reset --hard HEAD^

/*

最新的版本已经不见了，（上面的命令行窗口没有关掉找到最新版本的commit id）

*/

git reset --hard <commit id> // 最新版本commit id 写前面的几位就👌



git reflog // 记录你的每一次命令



git checkout -- readme.txt // 把readme.txt 文件在工作区的修改全部撤销

* readme.txt 自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样状状态。

* readme.txt 已经存在暂存区并作了修改，现在撤销修改就回到了添加到暂存区后的状态

#总之就是让这个文件回到最近一次的git coomit 或git add 时的状态。



git reset //既可以回退版本，也可以把暂存区的修改回退到工作区。用HEAD时，表示最新版本



# 小结：

*当你改乱了工作区的某个文件的内容，想直接丢弃工作区的修改时用命令 git checkout -- file

*当你不但改乱了工作区某个文件，还添加到暂存区时，想丢弃修改，分两步，第一步使用命令git reset HEAD <file> 就回到了场景一，第二部按场景一操作



# 删除文件

rm <file> //Git知道你删除了文件，工作区和版本库就不一致

git status // 命令会告诉你哪些文件被删除了



Now 你有两个选择

*确实要从版本库汇总删除该文件，使用命令git rm <file> 删除，并且git commit

*误删，我们可以通过 git checkout -- <file> 把误删的文件恢复到最新版本

// git checkout 其实是用版本库里的版本替换工作区的版本，无论工作区的是修改还是删除，都可以“一键还原”



# 远程仓库



第1步：ssh-keygen -t rsa -C “youremail” //生成一个ssh秘钥 在.ssh 目录下

第2步：登陆GitHub，打开“Account settings”，“SSH Keys”页面：然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件（公钥）的内容点“Add Key”，你就应该看到已经添加的Key。



# 使本地的仓库与github仓库同步

git remote add origin https://github.com/ppbibo/git-Practice.git

//添加后，远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。

git push -u origin master // 推送到远程库



/*

把本地库的内容推送到远程，用git push命令，实际上是把当前分支master推送到远程。

由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来。

推送成功后，可以立刻在GitHub页面中看到远程库的内容已经和本地一模一样

从现在起，只要本地作了提交，就可以通过命令：git push origin master //把本地master分支的最新修改推送至GitHub。

*/



# 小结

要关联一个远程库，使用命令git remote add origin git@server-name:path/repo-name.git；

关联后，使用命令git push -u origin master第一次推送master分支的所有内容；

此后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改；



git clone git@server-name:path/repo-name.git //克隆一个本地库

*Git支持多种协议，包括https，但通过ssh支持的原生git协议速度最快*/



# 分支管理
```