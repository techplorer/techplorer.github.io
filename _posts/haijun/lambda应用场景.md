lambda，也叫closure，是自c++ 11之后引进的新语法，它的定义以及用法网上的资料很多，这里主要结合自己的理解谈一下c++ lambda这种语法背后的应用场景。

我在项目代码里见的比较多的一种情形，就是用lambda传递回调函数，但如果仔细留意的话，会发现stl里提供的一些算法，比如for_each和sort，它们本身就用到了lambda，所以我们可以研究一下它们是怎么玩的。

### STL

- for_each

for_each算法就是对容器内的元素做遍历，至于对遍历到的元素的具体操作，由上层应用者自己决定，比如说要遍历一个容器并打印元素，是有这么两种写法：

```c++
    vector<int> data {3,-2,-10,100,1};
    
    // method 1
    for(auto v : data) {
        cout << v << endl;
    }

    // method 2
    for_each(data.begin(), data.end(), [](int v){cout << v << endl;});
```

可能有人会说这是语法糖，没啥意义，的确是这样的，不过这种语法背后隐藏了一些设计的元素，我们来看一个更具体一点的例子：sort。



- sort

sort是一种stl内置的排序算法，比如说有一堆数据，如果我们现在想要对它进行从小到大的排序，那么直接使用sort即可(这里不讨论sort的实现原理，其实它内部是一个快排算法)，但是在现实中，我们的需求会千奇百怪，假如我们需要对数据按照绝对值大小的顺序从小到大排序，那么该怎么办呢？对sort函数做重载，内部再实现一个新的排序算法？

其实sort算法把这种比较两个数大小的策略做成了类似接口的形式，开放给调用者，也就是比较两个数的大小可以由上层应用自行决定，看下示例代码：

```c++
    vector<int> data {3,-2,-10,100,1};
    
    cout << "before sort:" << endl;
    for_each(data.begin(), data.end(), [](int v){cout << v << endl;});

    cout << "default sort:" << endl;// default sort: 
    sort(data.begin(), data.end());
    for_each(data.begin(), data.end(), [](int v){cout << v << endl;});

    cout << "sort by absolute:" << endl;
    sort(data.begin(), data.end(), [](int a, int b){ return abs(a) < abs(b);});
    for_each(data.begin(), data.end(), [](int v){cout << v << endl;});
```

上面的for_each、sort的内部实现，是一种很不错的设计思路，相当于是把不变的部分固化成一种算法，把变的部分开放出去，由外部自行定义，这种思路也可以借鉴到我们的业务代码里边。



### 业务代码

假设我们有一个任务管理的模块，负责管理各种实况和录像等任务，现在业务侧有一个需求：需要查询所有录像和实况任务。

方法有很多，这里主要从代码设计的角度谈谈自己的理解，经分析，我们发现上面的需求其实本质上是一个查询请求，只是查询的参数不一样，所以我们可以开放一个参数，用于区分是录像还是实况任务。

但是如果有一天来了一个新的业务需求，需要查询某一天的录像或实况任务，这时我们就要封装新的接口了。

其实进一步分析，我们会发现查询的动作没有变化，只是要过滤的数据逻辑发生变化，所以我们可以把这层变化的逻辑开放出去，由应用者自行决定，参照上面lambda的写法，我们就能写出下面的代码：



```c++
#include <vector>
#include <string>
#include <iostream>
#include <algorithm>
#include <memory>
using namespace std;

class Task {
public:
    Task(string name) : taskName_(name) {}

    string TaskName() const {
        return taskName_;
    }

private:
    string taskName_;
};

class TaskMgr {
public:
template <typename Func>
vector<shared_ptr<Task>> FindMatchTasks(Func func) {
    vector<shared_ptr<Task>> result;
    for (auto task : tasks_) {
        if (func(task)) {
            result.push_back(task);
        }
    }
    return result;
}

void AddTask(shared_ptr<Task> t) {
    tasks_.push_back(t);
}

private:
    vector<shared_ptr<Task>> tasks_;
};

int main(void)
{
    auto mgr = make_shared<TaskMgr>();
    mgr->AddTask(make_shared<Task>("realvideo-tdjdjedjdjdj-2021-5-10"));
    mgr->AddTask(make_shared<Task>("realvideo-he52djdjd00d-2021-5-10"));
    mgr->AddTask(make_shared<Task>("realvideo-77djdj223djd-2021-5-21"));
    mgr->AddTask(make_shared<Task>("videorecord-djdjtt67555-2021-5-15"));
    mgr->AddTask(make_shared<Task>("videorecord-55jdjd00998-2021-5-21"));

    // filter task by realvideo
    vector<shared_ptr<Task>> realvideo = mgr->FindMatchTasks([](shared_ptr<Task> t) { return t->TaskName().find("realvideo") != string::npos;});
    cout << "filter tasks by realvideo:" << endl;
    for_each(realvideo.begin(), realvideo.end(), [](shared_ptr<Task> t){ cout << t->TaskName() << endl;});

    // filter task by videorecord
    cout << "filter tasks by videorecord:" << endl;
    vector<shared_ptr<Task>> videorecord = mgr->FindMatchTasks([](shared_ptr<Task> t) { return t->TaskName().find("videorecord") != string::npos;});
    for_each(videorecord.begin(), videorecord.end(), [](shared_ptr<Task> t){ cout << t->TaskName() << endl;});

    // filter task by 2021-5-21
    cout << "filter tasks by 2021-5-21:" << endl;
    vector<shared_ptr<Task>> someday = mgr->FindMatchTasks([](shared_ptr<Task> t) { return t->TaskName().find("2021-5-21") != string::npos;});
    for_each(someday.begin(), someday.end(), [](shared_ptr<Task> t){ cout << t->TaskName() << endl;});


    return 0;
}
```



### 总结

C++11里引入的lambda虽然是一种新的语法，或者叫语法糖，但是它背后蕴藏的设计思路值得借鉴，如果我们能加以正确使用，的确是多了一种优化代码、提升代码维护性的一种手段。