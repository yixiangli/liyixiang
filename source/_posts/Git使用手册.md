---
title: Git使用手册
toc: true
date: 2016-10-19 14:24:19
category: 
	- 技术贴
	- Git
tags: 
    - GitHub配置

---

# 概述
GitHub作为一个神奇的仓库，还是很6的，顺便贴一下博主的github，https://github.com/yixiangli


## 一张图
![Git命令详解](/img/git-command.png)

## 常见问题

#### git add 之后后悔了
别担心 git reset一下就好，对了 千万不要- -hard哈

<!--more-->
#### GitHub commit之后不显示contributions
作为程序狗当然都希望自己能做点惊天动地的事情，至少为软件行业做出点贡献也行啊，但是博主就遇到git commit提交之后 Github上并不显示contributions，很让博主伤心，后面通过指令
```
git config --list
```
一查发现博主所在公司项目使用的gitlab，之前配置的时候把user.name user.email配置成了公司邮箱的账号，Github就认为关联不到账号，解决方法是在仓库下
```
git config --global user.name xxxx
git config --global user.email xxxx
```

#### GIT 绑定两个远程仓库
作为公司的一枚程序狗，日常工作一般都会用公司自己搭建的GitLab(山寨版Github),正巧自己平时开发了一些工具类的小框架想造福更多的人，但是是两个不同仓库，同步自己的commit是一件很蛋疼的事，还好Git也有这个解决方案。
```
直接配置.git/config文件

在配置中添加：
[remote "all"]  
    url = https://github.com/xxxx/Mapper.git  
    url = https://git.oschina.net/xxxx/Mapper.git  
```

然后只需要在命令行输入(注意是两个- markdown好像打不出来)
```
git push all - -all
```

其他命令可以使用帮忙
```
git push all - -help
```


### 比较merge前后修改的内容
git show <merge-id>,注意不是commit id 而是merge id
通过git log查看

```
commit abe3857741be76b177e5d7bfe8575f9eb813097f
Merge: 532967b 0392f0a
Author: liyixiang <394244674@qq.com>
Date:   Wed May 3 14:27:18 2017 +0800

    Merge branch 'branch' into pre-release
```

这个532967b 0392f0a就是merge id

### 配置了.gitignore文件后远程文件如何删除
git rm -r --cached .settings