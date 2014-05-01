---
layout: post
category : intro
title : C++14语言特性一览
tags : [c++, c++11, c++14]
---

{% include JB/setup %}

<link rel="stylesheet" type="text/css" href="{{ root }}/css/pygments/native.css" />


原文连接：[http://cpprocks.com/an-overview-of-c14-language-features/](http://cpprocks.com/an-overview-of-c14-language-features/)

这篇文章中，我将着重讲解`C++14`标准草案中的一些语言新特性。这些是从我的《[C++11 Rocks: VS2013 Edition](http://cpprocks.com/)》中摘录出来的。在[上一篇文章](http://cpprocks.com/c1114-compiler-and-library-shootout/)中，我已经介绍了不同编译器对`C++14`支持的情况。

---

## 普通函数的返回类型推演

编译器可以推演这种函数的返回类型：

{% highlight cpp%}
auto square(int n) 
{
    return n * n;
}
{% endhighlight %}
    
如代码所示，你使用`auto`开始函数的声明，但是并没有在结尾处指定返回类型（即使用`trailing decltype`表达式）。此时，编译器可以自己推演出返回类型。这基本上是把`VS2013`中对匿名函数（即`lambda`表达式）返回类型的推演支持扩展到普通函数上.

---

## 泛型lambda
当需要给`lambda`表达式的参数指定类型时，你可以不使用具体类型，而使用`auto`。

{% highlight cpp%}
auto lambda = [](auto a, auto b) { return a * b; };
{% endhighlight %}

如果用`C++`伪代码表示，那么这样的定义等价于：    

{% highlight cpp%}
struct lambda1
{
    template<typename A, typename B>
    auto operator()(A a, B b) const -> decltype(a * b)
    {
        return a * b;
    }
};
auto lambda = lambda1();
{% endhighlight %}

所以使用`auto`作为函数参数类型等价于将函数调用模板化。因此普通`auto`变量的推演规则在这里并不适用，此处使用的是模板参数的推演规则。

---

## 扩展的lambda捕获

另一个对`lambda`表达式的修改牵扯到变量捕获。在`C++14`中，参数捕获可以包含一个初始化的表达式：

{% highlight cpp%}
auto now = [val = system_clock::now()] { return val; }; 
now();   // returns current time
{% endhighlight %}

在这个例子中，val赋值成了当前时间，然后`lambda`表达式将其返回了出去。val并不需要是一个已经存在的变量，所以这是一种给`lambda`表达式增加数据成员的有效方法。这些成员的类型由编译器来推演。

这样，仅能`move`（不能`copy`）的变量就可以被捕获了，而这在`C++11`中是不可能实现的。

{% highlight cpp%}
auto p = make_unique<int>(10);
auto lmb = [p = move(p)] { return *p; }
{% endhighlight %}

虽然真正发生的事情有点复杂，但是这就相当于通过`move`对一个变量进行了捕获。实际上，`lambda`表达式中的捕获语句块声明了一个名为`p`的新数据成员，而这个成员是通过外面已经被转化为右值引用的`p`初始化的。

---

## 对constexpr函数限制的修订

标准提案列出了`constexpr`函数中新的可以使用的东西，包括：

- 在`constexpr`中声明变量，但是不能声明`static`，`thread_local`以及未初始化的变量
- `if`和`switch`语句
- 循环结构，包括`range-based for`
- 对对象的修改，这些对象要求其生命周期开始于常量表达式

这应该会使`constexpr`函数更灵活。

---

## constexpr变量模板化

除了类型模板和函数模板，`C++14`还将允许`constexpr`模版化。这是个展示其如何工作的例子。

{% highlight cpp%}
template<typename T>
constexpr auto T pi = T(3.1415926535897932385);
{% endhighlight %}

没有新的语法，那些模板上已经存在的语法简单应用在了这里。这种变量可以用在泛型函数中。如：

{% highlight cpp%}
template<typename T>
T area_of_circle_with_radius(T r) 
{
    return pi<T> * r * r;
}
{% endhighlight %}

这条修改的意思是说尽管已经有了一个简单的初始化表达式，但是泛型函数中类型定制有时候还是有用的。

---

## 更多修改
`C++14`还包括一些其他的语言特性修改：

- 增加了另外一种推演规则可以允许你将`decltype`上的推演规则应用在`auto`变量上。
- 集合初始化对于类的成员初始化同样有效。
- 增加了内置的以`0b`开头的二进制字面量。
- 可能允许以单引号分割的数的字面量。
- 增加了另一个名为`[[deprecated]]`标准属性，用来标记部分代码是不推荐使用的。 

---

`C++14`标准有望今年完成。好消息是向编译器中添加新特性的工作已经全面展开，这里提及的部分特性现在已经可以使用了。
