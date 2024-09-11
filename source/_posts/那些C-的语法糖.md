---
title: 那些C++中的语法糖
tags: C++
categories: C++
date: 2024-09-08 21:22:58
updated: 2024-09-09 13:36:58
---


C++ 的许多特性，如基于范围的 for 循环、结构化绑定、lambda 表达式等，都是一些语法糖。下面我们一起看一下，这些语法糖的背后，编译器替我们做了哪些工作。

## 1. 基于范围的for循环（C++11）
```c++
std::vector<int> nums{1, 2, 3};
// C++11之前
std::vector<int>::const_iterator iter;
for (iter = nums.cbegin(); iter != nums.cend(); ++iter) {
  ...
}

// C++11开始
for (const auto &num : nums) {
  ...
}
```
> 如果我们需要删除或新增容器内的元素，基于范围的for循环就不好用了。

相较之下，基于范围的for循环代码更加简洁。
两者本质上是一样的，基于范围的for循环作为语法糖的一种，在被编译器展开后和C++11之前的写法几乎一致, 展开后的代码如下：
```c++
std::vector<int> nums = std::vector<int>{std::initializer_list<int>{1, 2, 3}};
{ // 多了一层范围，避免局部变量命名冲突。
  std::vector<int> & __range1 = nums;
  __gnu_cxx::__normal_iterator<int *> __begin1 = __range1.begin();
  __gnu_cxx::__normal_iterator<int *,> __end1 = __range1.end();
  for(; __gnu_cxx::operator!=(__begin1, __end1); __begin1.operator++()) {
    int const & num = __begin1.operator*();
    // 如果我们没有写&
    // const int num = __begin1.operator*();
    ...
  }
}
```
在实际使用中，我们的容器内的元素较基本数据类型复杂不少，因此，为了减少不必要的对象拷贝开销，需要在遍历时修改元素就使用 **&**，不需要修改就使用 **const &**。

## 2. 结构化绑定（C++17）

结构化绑定适用于C风格数组、tuple-like(标准库为支持结构化绑定定义的一组API，我们可以为自定义的类型实现这组API来使其支持结构化绑定，标准库中的 std::pair、 std::tuple、 std::array都实现了这些API )的对象以及 所有非静态数据成员都是public的结构体和类。

结构化绑定的出现允许我们写出更加简洁的for循环代码：
```c++
std::map<std::string, std::string> xattrs;
// C++17之前
for (const auto& xattr : xattrs) {
  auto &key = xattr.first;
  auto &value = xattr.second;
  ...
}

// C++17开始
for (const auto &[key, value] : xattrs) {
  ...
}
```
上面例子中的结构化绑定被编译器处理后的代码如下：
```c++
std::map<std::string, std::string> xattrs;
{
  std::map<std::string, std::string> & __range1 = xattrs;
  std::_Rb_tree_iterator<std::pair<const std::string, std::string>> __begin1 = __range1.begin();
  std::_Rb_tree_iterator<std::pair<const std::string, std::string>> __end1 = __range1.end();
  for(; operator!=(__begin1, __end1); __begin1.operator++()) {
    // 引用
    const std::pair<const std::string, std::string> & __operator9 = __begin1.operator*();
    const std::string & key = std::get<0UL>(__operator9);
    const std::string & value = std::get<1UL>(__operator9);
    
    //非引用
    // const std::pair<const std::string, std::string > __operator9 = std::pair<const std::string, std::string>(__begin1.operator*());
    // const std::string && key = std::get<0UL>(static_cast<const std::pair<const std::string, std::string> &&>(__operator9));
    // const std::string && value = std::get<1UL>(static_cast<const std::pair<const std::string, std::string> &&>(__operator9));
  }
}
```
从上面的代码可以看到，如果我们在使用结构化绑定时，没有使用引用，那生成的代码将会创建一个临时对象，然后再将变量绑定到临时对象的数据成员上。所以，和基于范围的for循环一样，在使用结构化绑定时，能带上引用尽量带上，能减少不必要的对象赋值构造函数的调用。

## 3.lambda 表达式(c++11)

在cppreference中对lambda的解释如下：
> Constructs a closure: an unnamed function object capable of capturing variables in scope.

Lambda 表达式**既然是一个函数对象，那就一定会有一个与之相关的类，并且重载了opernator() 方法**。看下面这个例子。
```c++
int main()
{
  auto comparator = [](int a, int b) {
    return a > b;
  };

  using func = bool(*)(int, int);
  func f = comparator;

  int a = 1;
  int b = 2;
  f(a, b);
  return 0;
}
```
编译器处理后如下：
```c++
int main()
{
  class __lambda_2_19
  {
    public: 
    inline /*constexpr */ bool operator()(int a, int b) const
    {
      return a > b;
    }
    
    // 类型转换，将 __lambda转换为 函数指针
    using retType_2_19 = bool (*)(int, int);
    inline constexpr operator retType_2_19 () const noexcept
    {
      return __invoke;
    };
    
    private: 
    static inline /*constexpr */ bool __invoke(int a, int b)
    {
      return __lambda_2_19{}.operator()(a, b);
    }
  };
  
  __lambda_2_19 comparator = __lambda_2_19{};
  int a = 1;
  int b = 2;
  comparator.operator()(a, b);
  return 0;
}
```
可以看到，编译器为其添加了类型转换运算符来支持lamdba表达式向函数指针的转换。不过只有没有捕获任何外部变量的lamdba表达式才能转换为函数指针。

再来就看一个捕获了外部变量的的lambda表达式：
```c++
int main() {
  int a;
  int b;
  int c;

  auto f1 = [=]() {
    int _a = a;
  };

  auto f2 = [&]() {
    int _b = b;
    int _c = c;
  };

  auto f3 = [a , &b] {
    b = 2;
  };
  return 0;
}
```

其被编译器展开后的代码如下：
```c++
int main()
{
  int a;
  int b;
  int c;
    
  class __lambda_6_13
  {
    public: 
    inline /*constexpr */ void operator()() const
    {
      int _a = a;
    }
    
    private: 
    int a;
    
    public:
    __lambda_6_13(int & _a)
    : a{_a}
    {}
    
  };
  
  __lambda_6_13 f1 = __lambda_6_13{a};
    
  class __lambda_10_13
  {
    public: 
    inline /*constexpr */ void operator()() const
    {
      int _b = b;
      int _c = c;
    }
    
    private: 
    int & b;
    int & c;
    
    public:
    __lambda_10_13(int & _b, int & _c)
    : b{_b}
    , c{_c}
    {}
    
  };
  
  __lambda_10_13 f2 = __lambda_10_13{b, c};
    
  class __lambda_15_13
  {
    public: 
    inline /*constexpr */ void operator()() const
    {
      b = 2;
    }
    
    private: 
    int a;
    int & b;
    
    public:
    __lambda_15_13(int & _a, int & _b)
    : a{_a}
    , b{_b}
    {}
    
  };
  
  __lambda_15_13 f3 = __lambda_15_13{a, b};
  return 0;
}
```

可以看到，在捕获外部变量后，生成的类没有重载类型转换运算符；此外，按值默认捕获和引用默认捕获都是按需捕获，用到哪些就捕获哪些。最后强调一点，使用lambda表达式时，无论是使用引用捕获还是按值捕获，都要特别注意捕获变量的生命周期，避免出现悬空指针。

## 总结
C++中的语法糖极大地提高了代码的可读性和开发效率。通过上述示例，我们可以看到这些特性的实现原理。通过编译器生成中间代码，我们能更专注于逻辑本身。掌握这些语法糖的运用，可以帮助我们写出更简洁、更高效的C++代码。

> 最后，我们可以通过[cppinsights](https://cppinsights.io/)这个网站来查看我们的代码被编译器展开后的样子。
