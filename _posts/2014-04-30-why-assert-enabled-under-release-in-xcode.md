---
layout: post
category : trap
title : assert出问题了？
tags : [c++, objective-c, ios]
---

{% include JB/setup %}

<link rel="stylesheet" type="text/css" href="{{ root }}/css/pygments/native.css" />

## 背景

刚学习`Objective-C`那会儿，还不太了解这个世界的惯用法，所以有些地方使用了`C/C++`的方式，虽然后来做过一定的修改， 但是项目中还是遗留了一些无关紧要的`C/C++`代码。比如对断言的运用。

{% highlight objc%}
assert(some != nil); 
{% endhighlight %}
___

## 问题

使用`assert`倒是没遇到啥问题，不过有一次在查阅测试同学提交上来的`crashlog`, 发现竟然崩在了`assert`调用上。

{% highlight objc%}
Thread 0 Crashed:
0   libsystem_kernel.dylib        0x3aaf61fc __pthread_kill + 8
1   libsystem_pthread.dylib       0x3ab5fa2e pthread_kill + 54
2   libsystem_c.dylib             0x3aaa6ff8 abort + 72
3   libsystem_c.dylib             0x3aa86d22 __assert_rtn + 178
4   test                          0x000f8b36 -[AppDelegate application:didFinishLaunchingWithOptions:]
......
15  CoreFoundation                0x300c014a __CFRunLoopRun + 1394
16  CoreFoundation                0x3002ac22 CFRunLoopRunSpecific + 518
17  CoreFoundation                0x3002aa06 CFRunLoopRunInMode + 102
18  UIKit                         0x328d2dd4 -[UIApplication _run] + 756
19  UIKit                         0x328ce044 UIApplicationMain + 1132
20  test                          0x000f8bde main (main.m:16)
21  libdyld.dylib                 0x3aa3fab4 start + 0
{% endhighlight %}

还真是稀奇，这到底是怎么回事呢？
___
## 缘由

首先，`assert`不是应该只在`NDEBUG`没有定义的时候有效么？难道`Release`默认没有设置这个宏？

通过`Xcode`日志很容易就发现`Release`下果然没有设置`NDEBUG`，验证了前面的假设。

![](http://imoldman-blog.qiniudn.com/assert_1.png)

可是为什么`Release`下默认没有设置这个宏呢？我记得`NDEBUG` 是语言标准的一部分啊？

查阅文档， 结果如下。

![](http://imoldman-blog.qiniudn.com/assert_3.png)

也就是说， `NDEBUG`确实是语言的标准，但是标准只定义了它是怎么影响`assert`的， 并没有定义编译器应该在什么情况下定义`NDEBUG`，所以`Xcode`在`Relase`模式下没有定义也是合乎标准的。
___
## 继续挖掘

问题找到原因了，可是我还是不死心，那如果`Xcode`是这样对待`assert`的， 那么自家的`NSCAssert`呢？

在`Foundation/NSException.h`中，`NSCAssert`大致是这样的定义的：

{% highlight objc%}
#if !defined(NS_BLOCK_ASSERTIONS)
#define NSCAssert(condition, desc, ...) \
    do {				\
	__PRAGMA_PUSH_NO_EXTRA_ARG_WARNINGS \
	if (!(condition)) {		\
	    [[NSAssertionHandler currentHandler] handleFailureInFunction:[NSString stringWithUTF8String:__PRETTY_FUNCTION__] \
		file:[NSString stringWithUTF8String:__FILE__] \
	    	lineNumber:__LINE__ description:(desc), ##__VA_ARGS__]; \
	}				\
        __PRAGMA_POP_NO_EXTRA_ARG_WARNINGS \
    } while(0)
#else
#define NSCAssert(condition, desc, ...) do {} while (0)
#endif 
{% endhighlight %}

与`assert`由`NDEBUG`来控制类似, `NSCAssert`是由`NS_BLOCK_ASSERTIONS`控制的。

那么`Release`下的`NSCAssert`会不会也会触发程序崩溃呢？

通过查阅`Xcode`在`Release`模式下的构建日志，发现答案是不是的。

![](http://imoldman-blog.qiniudn.com/assert_2.png)
___
## 总结

我只能说`Xcode`这里做的略不厚道，既然你给自家的`NSCAssert`定义了开关，那么也应该关照到`assert`。

作为开发人员，我们有两个处理办法。

- 不用`assert`, 完全改成`NSCAssert`（注意不要使用`NSAssert`，详见[这里](http://billwang1990.github.io/blog/2014/03/26/nsassert-vc-nscassert/)）。但是要同时注意你使用的第三方代码。
- 在工程设置`Build Settings -> Preprocessor Macros -> Release`中添加`NDEBUG=1`, 如下图。

![](http://imoldman-blog.qiniudn.com/assert_4.png)
