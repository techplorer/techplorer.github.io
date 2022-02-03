本文是关于c++ 模板类型推导的技术细节文章，不涉及高深的技术，但是如果对这些细节不了解的话，对于c++ 模板的深入学习和应用是一个麻烦，所以花了两天的时间，整理了一下关于template类型自动推导的知识点，方便查阅，主要分为两部分：模板类型推导规则(template type deducation)、引用类型折叠(reference collapsing)。



### c++ template推导规则



以c++ 的模板函数为例来说，

```
template<typename T>
void f(ParamType param);

f(expr); 
```

编译器会根据expr推导出T和ParamType，根据ParamType的类型，也分为三种场景：

- ParamType是指针或引用，但不是universal reference(也叫forward reference)

推导的规则是这样：

1. 如果expr是引用，那么忽略expr的引用部分；
2. 之后基于ParamType推导出T；



```
template <typename T>
void printRef(T& t) {}

template <typename T>
void printConstRef(const T& t) {}

template <typename T>
void printPtr(T* t) {}

template <typename T>
void printConstPtr(const T* t) {}

struct A {
    A() {std::cout << "A default cstr " << this << std::endl;}
    A(A& other) {std::cout << "A copy cstr " << this << " from " << std::addressof(other) << std::endl;}
    A(A&& other) {std::cout << "A move cstr " << this << " from "<< std::addressof(other) << std::endl;}
};

int main(void)
{
    // test case 1: pass by lvalue
    A a1;
    printRef(a1);// T->A, ParamType->A&, no copy/move constructor
    printConstRef(a1);// T->A, ParamTpye->const A&, no copy/move constructor
    printPtr(&a1);// T->A, ParamType->A*, no copy/move constructor
    printConstPtr(&a1);// T->A, ParamType->const A*, no copy/move constructor

    // test case 2: pass by const lvalue
    const A a2;
    printRef(a2);// T->const A, ParamType->const A&, no copy/move constructor
    printConstRef(a2);// T->A, ParamTpye->const A&, no copy/move constructor
    printPtr(&a2);// T->const A, ParamType->const A*, no copy/move constructor
    printConstPtr(&a2);// T->A, ParamType->const A*, no copy/move constructor

    // test case 3: pass by const reference
    A a3;
    const A& a4 = a3;
    printRef(a4);// T->const A, ParamType->const A&, no copy/move constructor
    printConstRef(a4);// T->A, ParamTpye->const A&, no copy/move constructor
    printPtr(&a4);// T->const A, ParamType->const A*, no copy/move constructor
    printConstPtr(&a4);// T->A, ParamType->const A*, no copy/move constructor

    // test case 4: pass by rvalue
    printRef(A()); // compiler complains: cannot bind non-const lvalue reference of type ‘A&’ to an rvalue of type ‘A’
}
```

- ParamType是universal reference

推导规则如下：

1. 如果expr是一个左值，那么T和ParamType都会推导成左值引用；(这里先记住这个规则)
2. 如果expr是一个右值，那么规则和前面的一种场景是一样的；



```
template <typename T>
void printUniversalRef(T&& t) {
    T tt(t);
}

struct A {
    A() {std::cout << "A default cstr " << this << std::endl;}
    A(A& other) {std::cout << "A copy cstr " << this << " from " << std::addressof(other) << std::endl;}
    A(A&& other) {std::cout << "A move cstr " << this << " from "<< std::addressof(other) << std::endl;}
};

int main(void)
{
    // test case 1: pass by lvalue
    A a1;
    printUniversalRef(a1);// T->A&, ParamType->A&, tt->A&, no copy/move constructor

    // test case 2: pass by const lvalue
    const A a2;
    printUniversalRef(a2);// T->const A&, ParamType->const A&, tt->const A&, no copy/move constructor

    // test case 3: pass by const reference
    A a3;
    const A& a4 = a3;
    // 这个地方需要注意：printUniversalRef的参数接收a4时，引用的类型在传递中会被丢弃
    printUniversalRef(a4);// T->const A&, ParamType->const A&, tt->const A&, no copy/move constructor

    // test case 4: pass by rvalue
    /*
    这个地方有两处需要注意:
    - 参数t是一个右值引用，被初始化为A(),不涉及构造;
    - 函数内部的tt此时类型是A，被初始化为t，而t虽然是一个A&，但在传递过程中会丢失引用，最终的效果
    就是tt调用了一次拷贝构造
    */
    printUniversalRef(A()); // T->A, ParamType->A&&, tt->A
}
```



- ParamTpye既不是引用，也不是指针

推导规则如下：

1. 如果expr是一个引用，则忽略引用部分；
2. 忽略expr的const；



```
template <typename T>
void printT(T t) {}

struct A {
    A() {std::cout << "A default cstr " << this << std::endl;}
    A(A& other) {std::cout << "A copy cstr " << this << " from " << std::addressof(other) << std::endl;}
    A(A&& other) {std::cout << "A move cstr " << this << " from "<< std::addressof(other) << std::endl;}
};

int main(void)
{
    // test case 1: pass by lvalue
    A a1;
    printT(a1);// T->A, ParamType->A, a1 copy from expr


    // test case 2: pass by const lvalue
    // 下面的代码会出现不过的问题：t在构造的过程中，接收的类型来自expr，真实的类型是const A，不满足拷贝构造函数原型
    const A a2;
    printT(a2);// T->A, ParamType->A


    // test case 3: pass by const reference
    // 编译不过，问题同case 2
    A a3;
    const A& a4 = a3;
    printT(a4);// T->A, ParamType->A, 

    // test case 4: pass by rvalue
    printT(A()); // T->A, ParamType->A, move constructor
}
```





### 类型折叠(reference collapsing)

universal reference推导的过程中，会涉及类型折叠，比如说上述的场景二，那么它的规则是这样：

- A& & becomes A&
- A& && becomes A&

- A&& & becomes A&
- A&& && becomes A&&



### 参考资料

- 本文内的所有代码经过测试，放在gitee仓库

https://gitee.com/techplorer/cpp_template_type_deducation



- c++ 模板类型推导可参考Scott Meyers的<Effective Modern C++>，这里有一篇网上的链接

https://www.oreilly.com/library/view/effective-modern-c/9781491908419/ch01.html



- c++ 右值引用和universal引用的区别

https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers

