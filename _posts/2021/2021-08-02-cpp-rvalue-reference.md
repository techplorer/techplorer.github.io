---
layout:   post
title:   "c++11特性：右值引用、移动语义、完美转发"
subtitle:  "C++ rvalue reference vs move semantic"
date:    2021-08-02 20:00:00
author:   "zhouhaijun"
catalog: false
header-style: text
tags:
 - c++
---

c++ 11新引入了一个大的特性，lvalue reference，即右值引用，它的主要用途是两个：一个是move semantic的语义实现，另外一个是用于perfect forwarding。本文主要是结合自己的经验，谈一谈对这二者的理解。

但是在开始正文之前，需要了解一些基础概念，所以我把这些概念的知识点放在第一节。如果你不理解这些概念，跳过第一章节依然可以往下看，如果是一些概念没理解的话，你可能还需要回过头来消化一下这些基础概念。



### 基础概念

- lvalue vs rvalue

先来了解一下lvalue和rvalue的定义(参考自[1]):

>  An lvalue is an expression that refers to a memory location and allows us to take the address of that memory location via the & operator. An rvalue is an expression that is not an lvalue.

简单通俗的讲，能通过操作符&拿到表达式的内存地址，那么我们就称这个表达式为左值，其他情况我们称为右值，举几组例子：

```c++
 // lvalues:
  int i = 42;
  i = 43; // ok, i is an lvalue
  int* p = &i; // ok, i is an lvalue
  int& foo();
  foo() = 42; // ok, foo() is an lvalue
  int* p1 = &foo(); // ok, foo() is an lvalue

  // rvalues:
  int foobar();
  int j = 0;
  j = foobar(); // ok, foobar() is an rvalue
  int* p2 = &foobar(); // error, cannot take the address of an rvalue
  j = 42; // ok, 42 is an rvalue
```



- lvalue reference vs rvalue reference

左值引用和右值引用的定义(参考自[1])：

> If X is any type, then X&& is called an rvalue reference to X. For better distinction, the ordinary reference X& is now also called an lvalue reference.

这里需要注意两点：第一点是不管是左值引用还是右值引用，它们描述的都是变量类型，而上面提到的左值和右值描述的则是表达式本身，所以这是两个完全不同的概念，这两个概念容易混淆；

另外一个是不管是左值引用还是右值引用，对于表达式本身来说，它都是左值。

我们还是以一组实例来说话：

```c++
class Metadata {
public:
	...
    Metadata(Metadata& other);// other的类型是一个左值引用，但other本身是一个左值
    Metadata(Metadata&& other);// other的类型是一个右值引用，但other本身是一个左值
    ...
};
```

此处不理解没关系，等到perfect forwarding章节，我们会用到这个知识点。

- universal reference

当然，如果一看到&&就以为是右值引用，那你就大错特错了，在c++ 11里边还有一种引用，长的和右值引用一模一样，即universal reference，在c++ 11的模板库代码里边很常见。我知道你肯定会问，既然有了右值引用的语义，为什么又要搞出这么个玩意？其实这里可以先保留疑问，等到perfect forwarding章节里提到的使用场景，你就会明白其中缘由。



既然是引用，那肯定需要初始化，universal reference的初始化规则也是有点另类，甚至是有点太过于灵活：

> 如果表达式初始化的是一个左值，那么它就是左值引用；
>
> 如果表达式初始化的是一个右值，那么它就是右值引用；



既然这家伙这么灵活，那问题来了，我们如何区分右值引用和universal referene？这里给出Scott Meyers在文章里边的定义([2])，其实这个名词也是他发明的：

> If a variable or parameter is declared to have type T&& for some deduced type T, that variable or parameter is a universal reference.

也就是说如果当涉及到类型推导，比如auto和模板函数，&&的涵义就是universal referene，其他情况下则对应的是右值引用。



上面的概念有点绕，这里我们还是举几组例子来具体说说：

第一个例子是模板参数：

```c++
template <typename T>
void foo(T&& t) {
    // ...
}

int main(void)
{
    int v = 1;
    foo(v); // 因为传入的是v，而v是左值，所以此时参数t的类型是左值引用

    foo(100); // 因为传入的是100，而100是一个右值，所以此时参数t的类型是右值引用
    return 0;
}
```

foo是一个模板函数，编译器编译程序时会对foo做类型推导，那么这个时候参数变量t的类型就是universal reference，它根据表达式的类型确定参数t的类型是左值引用该是右值引用；

第二个例子是auto：

```c++
int v1 = 1;
auto&& v2 = v1;
```

乍一看可能会以为v2的类型是右值引用，但实际因为auto涉及类型推导，所以这里的&&其实是universal reference，而v1在这里又是左值，所以编译器推导后的v2的类型实际是左值引用；

第三个例子是一个右值引用：

```c++
template <typename T>
class Metadata {
public:
    ...
    Metadata(Metadata&& other);
    ...
};
```

虽然Metadata是一个模板类，但是Metadata的移动构造函数里边，并不涉及类型推导，是具体的类型，所以这里的&&是右值引用，不是universal reference。



- reference collapsing rules

我们拿universal reference的一个场景来说：

```c++
template <typename T>
void foo(T&& t) {
    ...
}

int main(void)
{
    int v = 1;
    foo(v); // T推导为int&, 变量t本身是一个左值，但变量t的类型是左值引用

    foo(1);// T推导为int，变量t本身是一个左值，但变量t的类型是右值引用
    return 0;
}
```

此处我们需要关注两点：一点推导后的类型T，另一点是推导后的变量t的类型。

对于类型T来说，有这么两点约束：

假设A是一个变量类型：

> 如果传入的表达式是一个左值，则类型T推导为A&；
>
> 如果传入的表达式是一个右值，则类型T推导为A；

按照上面的知识点介绍，我们知道t在此处是一个universal reference，那么问题来了，T在推导后的类型，再加上&&，那最后会变成什么？c++ 11给这个使用场景定义一个规则，即reference collapsing rules：

> A& & becomes A&
>
> A& && becomes A&
>
> A&& & becomes A&
>
> A&& && becomes A&&

最后再来看一下变量t的类型，如果传入的是一个左值，那么推导后的变量t的类型是int& &&  == int&，即左值引用；如果传入的是一个右值，那么推导后的变量t的类型是int&&，即右值引用，这样就对上了。

### 

### move semantic

有了上面的背景知识，现在来看第一个问题：move semantic，c++ 11里对move semantic的定义是这样：

> `std::move` is used to indicate that an object `t` may be "moved from", i.e. allowing the efficient transfer of resources from `t` to another object.

以一个实际项目中经常会用到的场景：

```c++
vector<int> createVec(int n) {
    return vector<int>(n);
}

int main(void)
{
    vector<int> v {};
    v = createVec(10);
    int a = 10;
}
```

也就是有时我们需要创建一个临时的(temporary)的对象，并不需要deep copy，这时有一种聪明的做法就是把指向临时对象的指针和一些状态数据"偷"过来即可。

为了实现这个move语义，c++ 11里新增了两个概念：一个是移动构造，另一个是移动赋值：

```c++
// move constructor
A(A&& arg) : member(std::move(arg.member)) // the expression "arg.member" is lvalue
{} 
// move assignment operator
A& operator=(A&& other) {
     member = std::move(other.member);
     return *this;
}
```



为了区别于拷贝构造和拷贝赋值函数，c++ 11发明了右值引用，这样的话编译器就有了对应的区分。但是光编译器知道还不够，我们还得在程序调用起来，所以c++ 11又定义了std::move关键字。

这里要特别注意的是，这个std::move关键字并没有真正的去做数据移动之类的操作，它其实做的事很简单，就是将变量的类型转换为右值引用，好让移动构造和移动拷贝能被正确调用。

到此我们就把c++ 11里边move语义的发明背景和使用场景讲完了。



### perfect forwarding

右值引用的第二个作用，是perfect forwarding，如字面意思，这是一个跟参数转发有关的特性。

为了解释这个概念，我们先来构造一个应用场景：定义一个工厂函数，根据传入的参数，返回一个具体的类型。

- 版本1

  

```c++
#include <iostream>
using namespace std;
template <typename T, typename... Args> 
T create(Args... a) {
    return T(a...);
}

class Metadata {
public:
    Metadata() {
        cout << "Metadata cstr:" << this << endl;
    }

    Metadata(Metadata& other) {
        cout << "Metadata cstr:" << this << " copy from " << addressof(other) << endl;
    }

    Metadata(Metadata&& other) {
        cout << "Metadata cstr: " << this <<" move from " << addressof(other) << endl;
    }

    ~Metadata() {
        cout << "Metadata dstr: " << this << " bye" << endl;
    }
};


int main(void)
{
    Metadata m1 = create<Metadata>(); // okay
    Metadata m2 = create<Metadata>(m1); // okay 调用两次拷贝构造
    Metadata m3 = create<Metadata>(std::move(m1)); // not okay 调用两次构造：一次是移动构造，另一次是拷贝构造，move语义没有实现
    return 0;
}
```



第一个版本的代码，我们发现当我们试图用std::move调用时，move语义并没有实现，程序一共调用了两次构造：第一次是移动构造，第二次是拷贝构造。而且我们的这种写法也导致编译器的RVO失效。

我们希望的是，如果传入的是左值，那么create内部调用拷贝构造；如果传入的是右值，那么create内部调用移动构造，所以第二个版本里边，我们考虑将create的参数改为universal refercence。



- 版本2

  

```c++
template <typename T, typename... Args> 
T create(Args&&... a) {
    return T(a...);
}

...

int main(void)
{
    Metadata m1 = create<Metadata>(); // okay
    Metadata m2 = create<Metadata>(m1); // okay 调用一次拷贝构造
    Metadata m3 = create<Metadata>(std::move(m1)); // not okay 竟然也调用了一次拷贝构造
    return 0;
}
```

在版本2里边，我们发现编译器的RVO生效了，不管是拷贝还是移动构造，都只调用了一次，但是诡异的是我们的move语义非但没有实现，还变成了拷贝构造。

这是因为我们上面也说过，不管是左值引用还是右值引用，其变量本身还是一个左值，所以在create匹配类的构造函数时，都匹配的是拷贝构造。那怎么改呢？方法也比较简单，使用static_cast做类型强转，这就是接下里的我们的版本3.



- 版本3

  

```c++
template <typename T, typename... Args> 
T create(Args&&... a) {
    return T(static_cast<Args&&>(a)...); // okay
    //return T(std::forward<Args>(a)...); // okay
}

...
   
int main(void)
{
    Metadata m1 = create<Metadata>(); // okay
    Metadata m2 = create<Metadata>(m1); // okay 调用一次拷贝构造
    Metadata m3 = create<Metadata>(std::move(m1)); // okay 调用一次移动构造
    return 0;
}
```

很好，版本3实现了我们的 需求，也即当传入左值时，调用类的拷贝构造；当传入的是右值时，调用类的移动构造。这个就是c++ 11里边的perfect-forwarding，也就是完美转发。当然c++ 11里还提供了一种模板函数std::forward，它做的工作和我们版本3里的代码是一样的。

到此为止，我们就把完美转发的逻辑以实例的方式讲完了。



### 参考资料

[1] [rvaue reference/move semantic/perfect forwarding]( http://thbecker.net/articles/rvalue_references/section_01.html)

[2] [universal reference](https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers)

