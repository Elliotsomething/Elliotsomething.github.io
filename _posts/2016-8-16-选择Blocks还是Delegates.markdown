---
layout:     post
title:      "iOS 之 选择Blocks还是Delegates"
subtitle:   "Block Or Delegate？"
date:       2016-8-16
author:     "Elliot"
header-img: "img/post-bg-ios9-web.jpg"
catalog:    true
tags:
    - iOS
    - Block
    - Delegate
---

**版权声明：本文为博主原创文章，未经博主允许不得转载**

### 选择Blocks还是Delegates

在设计接口是常常纠结是选择Block还是Delegate回调数据，之前看到一个博客，总结一下仅作为学习之用；

1、如果对象有超过一个以上不同的事件源，使用delegation（比如多个参数的delegate，block最好是只返回确定意义的1~2个参数）。

2、如果一个对象是单例，不要使用delegation，直接block增加代码可读性

3、如果对象的请求带有附加信息，更应该使用delegation（比如需要返回值，像DataSource）

4、delegate的回调更多的面向过程，而block则是面向结果的。如果你需要得到一条多步进程的通知（比如，will...、did...等），你应该使用delegation。而当你只是希望得到你请求的信息（或者获取信息时的错误提示），你应该使用block。（如果你结合之前的3个结论，你会发现delegate可以在所有事件中维持state，而多个独立的block确不能）
