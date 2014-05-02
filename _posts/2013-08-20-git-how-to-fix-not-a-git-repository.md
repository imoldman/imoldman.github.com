---
layout: post
category : trap
title : fatal &#58; Not a git repository 的解决办法
tags : [git]
---

{% include JB/setup %}

<link rel="stylesheet" type="text/css" href="{{ root }}/css/pygments/native.css" />


## 问题

在 `Git Bash` 中执行与 `submodule` 相关操作时，有可能会出现以下提示：

    fatal: Not a git repository: ../../../../..//c/some/path/.git/modules/third_party/boost
---

## 解决方案

这其实是由于目录路径太长导致的，处理方法如下：

- 找到此 `submodule` 的 `workspace` 目录。
    - 在本例中，其为 `C:/some/path/third_party/boost`。
- 在上述目录中查找.git文件（**注意是文件不是目录**），其内记载了此submodule的.git目录的实际存放位置，打开它。
    - 在本例中，它看起来大致是这样的： `gitdir: ../../../../..//c/some/path/.git/modules/third_party/boost`
- 很明显，这个目录跟报错的目录相同，也就是说这个问题是由于gitdir目录太长导致的，在这里修改的办法有两种：
    - 将不需要的父目录移除，如可将上例改为 `gitdir: ../../.git/modules/third_party/boost`
    - 将其改为绝对路径，如可将上例改为 `gitdir: /c/some/path/.git/modules/third_party/boost`
- 执行 `git submodule update --init --recursive -- third_party/boost`

这样就可以解决了。
