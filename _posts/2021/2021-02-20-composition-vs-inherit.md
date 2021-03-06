---
layout:   post
title:   "组合优于继承"
subtitle:  ""
date:    2021-02-20 21:00:00
author:   "zhouhaijun"
catalog: false
header-style: text
tags:
 - 编程思想
---

### 业务场景

有一个业务模块Fetcher，它的功能是获取视频流和图片数据，支持的协议有rtsp、http，同时也要支持一些消息中间件(如kafka、zeromq等)。



### 解决方案一(继承)

我们先来分析一下，怎么实现这个功能：

- 首先，我们会在Fetcher这个模块里定义一个接口FetchData
- 其次，我们会让不同实现层继承Fetcher这个接口
- 最后，在不同的实现层实现FetchData这个接口

根据上面的一番思考，我们可能写出来的代码是这样：

```
#include <iostream>
#include <memory>

class Fetcher {
public:
    virtual void FetchData() = 0;

    void SendTo() {
        std::cout << " send data to ...." << std::endl; 
    }
};

class RtspFetcher : public Fetcher {
public:
    void FetchData() {
        std::cout << "RtspFetcher...." << std::endl;
    }
};


void ShowAppDemo() {
    // 业务代码组装
    std::shared_ptr<Fetcher> fetcher = std::make_shared<RtspFetcher>();
    fetcher->FetchData();
}

int main(void)
{
    ShowAppDemo();
    return 0;
}
```

编译一下，嗯，程序编译没有报错，功能确实也能跑通。



但现在功能开发完成后，如果我想给Fetcher这个模块做单元测试，该怎么处理呢？实际工作中这个Fetcher模块的业务逻辑可能比我们写的demo要复杂很多，所以单元测试是避免不了的。



### 解决方案二(组合)

上面的这段代码让Fetcher这个类变成了抽象类，所以答案是做不了单元测试，那如果要解决这个问题，就应该使用组合的方案，分析一下：

- 首先，将Fetcher这个模块与实现层依赖的接口修改一下形式，独立成一个接口类，与此同时在Fetcher定义这样的一个接口字段
- 其次，将Fetcher的构造函数增加一个入参，传入的参数即为这个接口

代码见下面：

```
#include <iostream>
#include <memory>

class DataFetcher {
public:
    virtual void FetchData() = 0;
};

class Fetcher {
private:
    std::shared_ptr<DataFetcher> dataFetcher;

public:
    Fetcher(std::shared_ptr<DataFetcher> dataFetcher) {
        this->dataFetcher = dataFetcher;
    }

    void FetchData() {
        this->dataFetcher->FetchData();
    }

    void SendTo() {
        std::cout << " send data to ...." << std::endl; 
    }
};

class RtspFetcher : public DataFetcher {
public:
    void FetchData() {
        std::cout << "RtspFetcher...." << std::endl;
    }
};


void ShowAppDemo() {
    // 业务代码组装
    std::shared_ptr<DataFetcher> dataFetcher = std::make_shared<RtspFetcher>();
    std::shared_ptr<Fetcher> fetcher = std::make_shared<Fetcher>(dataFetcher);
    fetcher->FetchData();
}

int main(void)
{
    ShowAppDemo();
    return 0;
}
```

这样一来，我们既做到了Fetcher与实现层的解耦，又让Fetcher的可测试性得到了保障。



### 总结

面向对象编程里，有一个设计原则：组合优于继承，也就是说当一种实现场景下，既能用组合也能用继承的时候，优先使用组合。
