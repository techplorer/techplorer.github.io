---
layout:     post
title:      "C++继承"
subtitle:   "C++ public vs private inherit"
date:       2021-03-10 12:00:00
author:     "zhouhaijun"
catalog: false
header-style: text
tags:
  - 被夹
---

### 公共继承与私有继承

- 公共继承

对于类A来说，如果使用public继承类B，那么对于类B中的public方法都会变成类A的一部分



- 私有继承

对于类A来说，如果使用private继承类B，那么对于类B的中public方法只对类A的方法可见，也就说类A从类B继承的方法不能直接对外使用



- 比较

表面上看，公共继承与私有继承是方法可见性的一种表达，实质上，两者在使用场景上是有很大差异的。

还是以上述的类A和类B为例，公共继承表示类A和类B的关系是is-a，在实际的使用中，一般用公共继承来实现接口：

```
#include <iostream>

class B {
public:
    virtual void work() = 0;

};

class A : public B {

public:
    void work() {
        std::cout << " this is A..." << std::endl;
    }


};

void App(){
    std::shared_ptr<B> b = std::make_shared<A>();
    b->work();
}

int main(void)
{
    App();
    return 0;
}
```



私有继承表示类A和类B的关系是has-a，也就是说私有继承实际上继承的是实现而不是接口，在实际使用中，一般是用于代码复用：

```
#include <iostream>

class Mouse {
public:
    void Click(){
        std::cout << "Hello, I'm Mouse, please click me..." << std::endl;
    }

};

class Computer : private Mouse {

public:
    void PowerOn() {
        std::cout << "Welcomt to compute world." << std::endl;
        Click();
    }

};

void App(){
    std::shared_ptr<Mouse> mouse = std::make_shared<Mouse>();
    mouse->Click();

    // 下面的这行代码无法通过编译，使用私有继承的类已经不再是is-a的关系，也就是两个不一样的类实体
    //std::shared_ptr<Computer> computer = std::make_shared<Mouse>();

    std::shared_ptr<Computer> computer = std::make_shared<Computer>();

    // 下面的这行代码也无法通过编译，因为使用私有继承的方法仅能在内部使用，无法对外呈现
    //computer->Click();

    computer->PowerOn();

}

int main(void)
{
    App();
    return 0;
}
```



### 私有继承与组合

组合也表示类之间的关系是has-a，那么组合与私有继承的区别是什么呢？

两者之间实质上没有太大的区别，其中一个比较明显的点：对于被继承的类来说，私有继承只允许有一个，而组合可以允许有多个。

以上述的Computer和Mouse为例，一个Computer只能继承一个Mouse，但是使用组合的方式，则没有这个限制。



### 总结

私有继承与公共继承表面上体现的是继承的方法可见性，实际上是不同的应用场景下的两种产物：私有继承用于代码复用，一般实际工作中用的比较少，大多数场景下都可以用组合替代；而公共继承则表示是接口实现，一般是配合C++里的虚函数一起使用。
