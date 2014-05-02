---
layout: post
category : tip
title : gtest运行参数使用技巧
tags : [c++, gtest]
---

## 简介

gtest全称为 [Google C++ Testing Framework](https://code.google.com/p/googletest/), 是Google开发的一个单元测试框架，现阶段，我们使用它作为各个平台的单元测试框架基础。

## 资料
- [易上手的教程](https://code.google.com/p/googletest/wiki/Primer)。
- 上面是官方英文版教程，这是另一份[中文教程](http://www.cnblogs.com/coderzh/archive/2009/04/06/1426755.html)。

## 运行参数

为了方便使用，gtest提供了一系列的命令行参数，详情可以参看[这里](http://www.cnblogs.com/coderzh/archive/2009/04/10/1432789.html)。

关于参数，这里有几个技巧：

- `--gtestfilter`
    - 使用`--gtestfilter`可以只运行某一个单元测试，使开发过程不受其他单元测试的影响，加快开发节奏。
    - 如`--gtestfilter=FooBar.*`便可以只运行FooBar族的单元测试。
- `--gtest_break_on_failure --gtest_catch_exceptions=0`
    - 默认情况下，为了便于自动化测试，`gtest`在遇到错误和异常时并不会通知调试器。不过这并不方便我们开发，尤其是`EXCEPT_*()`触发时不能直接定位问题现场。
    - 如果要修改此默认行为，可以在运行参数中添加`--gtest_break_on_failure --gtest_catch_exceptions=0`。

