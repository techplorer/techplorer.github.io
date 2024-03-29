---
layout:   post
title:   "谈谈入职阿里后的感受"
subtitle:  ""
date:    2022-07-28 8:00:00
author:   "zhouhaijun"
catalog: false
header-style: text
tags:
 - 感悟
---

### 交代背景

2022年的第一篇文章，其实很早之前就想写一篇关于入职阿里的文章，但一直也没啥时间。

自从去年11月入职阿里以来，无论是业务、技术、文化，都是以初学者的心态保持学习，从刚开始的很不适应，到后来的慢慢融入，收获了一些成长，当然这中间也经历很多痛苦的时刻。



### 关于技术

我所在的部门主要负责饿了吗外卖核心调度链路改造，简单点说，是配合算法同事，做工程侧链路改造，着重于性能提升，最终的目的是给公司降本增效。

因为之前经历的两家公司，都是偏重业务，对性能这块不是很敏感。阿里这边的新工作对性能要求很高，比方说最近我们在改造的其中一个指标：万人万单秒级响应，所以相应地，对我们做代码设计和开发的工作来说，要求的确是要高一些，总结起来是这么几点：

- 代码追求：简单优雅

  功能实现是第一层，但如果只是实现业务功能的堆砌，对代码没有追求的话，在阿里这边基本是要被扣上“低水平”的帽子。所以在做代码cr之前，一定要自己打磨，准确地说，一定深思熟虑和反复修改，改到自己觉得无可挑剔的水准，否则一定会被喷的体无完肤，对于自身而言不仅没有成长还容易挨骂。

- 利用工具提效

  互联网公司干活尤其是注重效率，出活快又能保证质量，那怎么做到呢？其实就是利用工具，只要工具用的好，大家智商都差不多，不存在明显的好生差生一说。

  举几个我自己工作中的切身体会：代码性能瓶颈怎么排查呢？如果是没有搞过性能优化的同学，可能第一反应是通过日志对中间过程采样耗时，但因为性能优化涉及到方方面面，CPU、内存、磁盘、网络等，如果只是简单的记日志，基本上是查不出问题的。搞过性能优化的同学肯定知道，其实就是借助合适的工具，比如vtune、perf，你要做的就是了解这块工具的基本用法，再对CPU的工作原理做一些了解，达到能正确使用工具的水平就可以了，当然了，在学习工具使用的阶段如果能对原理做一些思考，有自顶向下的思考习惯那就更好了。

  再有就是，写代码时怎么知道自己写的代码是不是垃圾代码呢，会不会有性能瓶颈呢？答案也是用工具，网上有款工具[Compiler Explorer](https://godbolt.org/),这款工具可以把C++代码转成汇编代码，让你很清晰的了解代码背后的执行过程，你要做的就是学习一点汇编的语法即可。

- 学习新东西，从原理入手

  我的直属leader学习新东西很快，当然这和他在阿里工作多年以及个人学习习惯都有关系，但是他身上的确有一个优点非常值得我学习：学东西要从原理入手，并长期保持这种思考习惯，直到它变成你的一种思维惯性。用这种方式学东西一开始一定会很慢，因为每接触一个新东西(这里的新东西，可能是一个框架、工具、模块等)，都会牵扯到一堆知识网，抽丝剥茧似的理清各层知识间的依赖。

  不过这种学习方法有一个很大的优势，就是惯性，或者说”滚雪球“，雪球会越滚越大，当随着时间的维度，你积累的知识量越多，学习新东西的东西会越来越快。



### 关于业务

我以前是搞安防行业的，来到物流这个领域，自己就是一个新人，更多地是以一个新人的视角，观察和学习周边的同事是怎么理解、学习和挖掘业务的，不过可惜地是，在业务方面，我还真的没有长进，总结起来，我觉得主要是几个点导致的：

- 自己对业务还不够主动；
- 精力不够，来到阿里的这大半年，除了工作就是工作，说句实话，吐槽一下：比我前面两家公司的强度大多了，压力也大。毕竟也有家庭的人，除了工作还得陪陪孩子和家人；



### 关于文化

关于阿里的文化价值观，用我通俗的话来说，就是四个字”人情淡薄“。从我自身的角度来说，我把它拆成两点来看：

- 好的一面

  阿里的价值观中，大部分我是认同的，比如说”不满足现状“、”不给自己设限“，对自己要求越高，成长越快，这个对于追求成长的同学来说是好事。

- 不好的一面

  阿里是一家互联网公司，本身节奏就很快，人的精神压力也很大，对新人是极其不友好的，用一个比喻来形容：好比说一个游泳初学者，阿里人的做法就是把你直接丢进深水区，自己学吧，学不会就淹死算了。

优胜略汰的丛林法则在阿里非常严重，所以我也理解为什么阿里内部会提倡”聪明”、“皮实”、“积极”，不“皮实”的人早就被吓跑了，不追求上进就没法提效，自然就会变成“不聪明”的那类人，做事“不积极”也很难推进工作，无法拿到最终的结果。

以我自身的经历来说，以前大家都是和和气气的，来到阿里之后，只要事情没有达到领导预期，就是一顿喷，不管你年龄大小，在这里没有面子可言，把事情做好并确保有正向的结果才是王道。

在这一点上一开始我的排斥心理也是很大，不过后面也习惯了，我们项目组的同事挨个被喷过，不针对个人，其实也没什么，只要能学到东西有成长，喷就让他喷吧。



### 关于方法论

部门领导很注重方法论，我从平常的言行举止中，也get到了一些我认为还不错的，记录一下吧：

- 方向不要跑偏

  方向跑偏，越努力结果越扯淡。拿我自身的例子来说，因为我们是饿了吗外卖派单引擎，午高峰、晚高峰点外卖的人很多，这期间如果线上出现故障，一定要第一时间“止血”，在几分钟内切流。

  如果遇到线上就立马排查问题，错过故障处理最佳时间，最终导致公司外卖派单派不出去，那问题就大了，要是搞出一个P1故障，直接就被公司给开了。

- 抓大放小

  工作中要并行处理的事很多，尤其是在接触盲区或者新领域时，稍不注意就容易芝麻西瓜一起抓，分不清重点和主次，注意在工作中用好二八原则。

- 控制变量法

  这种方法一般在自己不熟悉不擅长的领域方面，是很好使的，在工作中这种也是用的多的一种方法论，比如在排查自己不太熟悉的代码问题时，通过二分法逐步缩小排查范围，可以快速收敛问题排查的路径。



### 关于未来

来阿里之前，原本我想的是来这边沉淀和开阔一下技术视野，来了之后才发现是自己想的太过简单。

我原来呆过宇视、华智，也算是华为系公司，和阿里的风格真的是差别蛮大的，来了阿里之后，我发现这里没有什么人情世故，更多地是一种员工和公司的交易，用自己的话来说：”没有产出、不能帮助团队、帮助公司取得正向的的业务结果，那就说明不合适，就会走人“。这一点上，我相信很多公司都一样，但是在阿里这一点尤其突出，在阿里没有结果没有产出，就意味着你在这里呆不久了，从我这段时间的观察来说，不管是小员工还是大领导，普遍都有精神上的焦虑，就是心里都有一根绷着的弦，搞不好哪天就被”优化“了。

阿里这边怎么说呢，员工没有安全感是真的，当然它也不需要给你安全感，拿钱干活，干不出结果那就走人吧。我后面想了想，一般很多公司到最后都会有”劣币驱逐良币“，阿里为什么能做这么大，可能就是因为这种鲶鱼文化，才能让它保持企业活性。



### 总结

来到阿里，总的来说，收获的成长大于受挫，最大的收获就是解了一道困惑心底很久的题：以前在宇视和华智，无论是带团队还是做产品研发，总会遇到一些瓶颈点。解题的方法其实很简单，对自己做的事要不断的找出不足和改进点，并付诸行动，完成一个一个的迭代。

对未来，在阿里这边没有太多的想法，总之不稳定是肯定的，尤其是在现在的这种经济下行的大环境下，稳定压倒一切，走一步看一步吧，不管怎样，继续努力学习，搞搞提升。



