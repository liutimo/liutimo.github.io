---
uuid: 248a6ce5-8d26-11ef-87e9-758771b60cf9
title: 从另一个视角来了解lambda表达式
date: 2024-09-08 23:13:36
updated:
tags:
categories:
---

C++的lambda表达式大家应该不陌生，或者说是非常了解。所以我今天不再介绍其使用方式，而是从另一个视角来介绍lambda表达式究竟是一个什么东西。

在cppreference中对其的解释如下：
> Constructs a closure: an unnamed function object capable of capturing variables in scope.

Lambda 表达式是创建匿名函数对象的一种简易途径，常用于把函数当参数传，例如：
```c++
std::sort(v.begin(), v.end(), [](int x, int y) {
    return Weight(x) < Weight(y);
});
```

lambda表达式**既然是一个函数对象，那就一定会有一个与之相关的类，并且重载了opernator() 方法** 接下来，我们一起来尝试还原一下这个类。

先看一个栗子：
```c++
using func = bool(*)(int, int);
func f = [](int a, int b) {
  return a < b;
};
```
可以看到, lambda 表达式是可以隐式的转换为函数指针的。因此，其对应的类里面一定会实现一个类型转换运算符来将lambda表达式转换为函数指针。
因此，我们针对这个栗子，我们实现的lambda类如下：
```C++
class lambda {
public:
  bool operator()(int a, int b) const {
    return a < b;
  }

  using func = bool(*)(int, int);
  operator func() const {
    return _func;
  }

  static bool _func(int a, int b) {
    return lambda()(a, b);
  }
};

using func = bool(*)(int, int);
func f = lambda();
```

换个栗子：
```c++
int a = 10;
int b = 20;

auto lambda = [a, &b]() {
  return a < b;
};

```

这次，我们捕获了两个变量a（copy的方式捕获）, b(引用的方式捕获)，这意味着，lambda表达式对应的类中需要保存这些捕获的变量，实现如下：
```c++
class lambda {
  int a;
  int& b;
public:
  lambda(int& _a, int& _b) : a(_a), b(_b) {}

  bool operator() () {
    return a < b;
  }
};
```

> 额，有个疑问，这里为什么没有实现函数指针的隐式转换？
> 
> 因为我们没法在静态成员函数中调用带参数的构造函数。事实上，标准库的实现也是如此，对于简单的lambda表达式(不涉及捕获)，其生成的类里面会实现一个类型转换运算符，而使用了捕获的lambda表达式，则没有实现一个类型转换运算符。

那如果我们使用默认的按值捕获[=] 和引用捕获[&]，生成的类里面是不是会捕获
上下文中所有能被lambda看到的变量呢？

当然不会是所有，只有lambda表达式主体中用到的变量才会被捕获。



