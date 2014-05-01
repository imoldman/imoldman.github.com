---
layout: post
category : tip-trap
title : fatal &#58; Needed a single revision 的解决办法
tags : [git]
---

{% include JB/setup %}

<link rel="stylesheet" type="text/css" href="{{ root }}/css/pygments/native.css" />

## 问题

在 `Git Bash` 中执行 `git submodule update --init --recursive` 或之类的操作时可能会出现这样的提示：

    fatal: Needed a single revision
    Unable to find current revision in submodule path 'third_party/foobar'
---

## 解决方案一
这个多半是由于 `third_party/forbar` 这个 `submodule` 的 `url` 有所修改，处理步骤如下：

{% highlight bash linenos %}
$ git submodule sync -- third_party/foobar
$ git submodule update --init --recursive -- third_party/foobar
{% endhighlight %}
---

## 解决方案二

如果仍然出现 `fatal: Needed a single revision` 的提示，则进行以下步骤

- 找到记载此 `submodule` `url` 的配置文件 `config` ，将其内有关此 `submodule`的配置删除。
    - 假设根repo目录为 `C:/some/path` , 则 `config` 文件便位于 `C:/some/path/.git/config`, 编辑之，删除5-6两行：
{% highlight ini linenos%}
[submodule "third_party/boost"]
url = ssh://wisp@git.lab.infra.mail:1046/data/git/wisp/boost.git
[submodule "third_party/gtest"]
url = ssh://wisp@git.lab.infra.mail:1046/data/git/wisp/gtest.git
[submodule "third_party/foobar"]
url = git://github.com/imoldman/asio.git
{% endhighlight %}

- 将这个 `submodule` 的 `.git` 目录删掉。
    - 在本例中其目录为 `C:/some/path/.git/modules/third_party/foobar`，将其删掉。
- 将这个 `submodule` 的 `workspace` 目录删掉。
    - 在本例中其目录为 `C:/some/path/third_party/foobar`，将其删掉。
- 执行 `git submodule update --init --recursive -- third_party/foobar`

这样就可以解决了。
