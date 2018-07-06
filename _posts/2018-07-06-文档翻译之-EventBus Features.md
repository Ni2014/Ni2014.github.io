---
layout:     post
title:      文档翻译之-EventBus Features
subtitle:   EventBus Features
date:       2018-07-06
author:     宸笙
catalog: true
tags:
    - EventBus
---



# 导语

上周刚翻译完包老师的新书《Android插件化技术》的中的混淆章节，回想自己之前总是倾向于翻译后的文档和中文教程，这一两年慢慢有了变化，也可能是对技术的理解慢慢到位，对文档的阅读很快很自然甚至更喜欢地道的英文文档，避免了中间的曲解，而看到圈内一些前辈除了技术很过关之外还有不错的写译功底，也许是进步了，后续要逐步翻翻一些不错的三方库文档沉淀下，这次，先拿EventBus练练手吧。

## EventBus Features

EventBus功能特性

What makes greenrobot’s EventBus unique, are its features:

使得greenrobot独一无二的特性分别有：

Simple yet powerful: EventBus is a tiny library with an API that is super easy to learn. Nevertheless, your software architecture may great benefit by decoupling components: Subscribers do not have know about senders, when using events.

简单但强大：EventBus是一个Api超级易学的轻量级库，尽管如此，你的软件架构还是能从解耦组件中获益良多：因为当使用事件时，订阅者不需要知道发送消息的一方；

Battle tested: EventBus is one of the most used Android libraries: thousands of apps use EventBus including very popular ones. Over a billion app installs speak for themselves.

久经测试(打磨)：EventBus是最流行的Android库之一，无数的App使用EventBus，据开发者们反馈，超过十亿个App使用了EventBus；

High Performance: Especially on Android, performance matters. EventBus was profiled and optimized a lot; probably making it the fastest solution of its kind.

高性能：在Android平台上性能尤为重要，EventBus被提升和优化了很多，也许是在同类中最快的解决方案；

Convenient Annotation based API (without sacrificing performance): Simply put the @Subscribe annotation to your subscriber methods. Because of a build time indexing of annotations, EventBus does not need to do annotation reflection during your app’s run time, which is very slow on Android.
Android main thread delivery: When interacting with the UI, EventBus can deliver events in the main thread regardless how an event was posted.

方便的基于注解的Api(不用牺牲性能)：只需要给你的监听事件的方法加一个@Subscribe注解即可。得益于编译时生成注解的索引(索引加速)，EventBUs不需要在App运行时使用反射去处理注解，那种方式是很慢的；

Background thread delivery: If your subscriber does long running tasks, EventBus can also use background threads to avoid UI blocking.

后台线程分发：如果你的订阅者需要执行耗时任务，EventBbus将会使用后台线程(去处理)以避免UI(线程)阻塞；

Event & Subscriber inheritance: In EventBus, the object oriented paradigm apply to event and subscriber classes. Let’s say event class A is the superclass of B. Posted events of type B will also be posted to subscribers interested in A. Similarly the inheritance of subscriber classes are considered.

支持事件或者订阅者继承：在EventBus中，在事件和订阅者的类族中应用了面向对象的典范；假设事件类A是B类的父类，B类事件也会被分发到订阅了A事件的订阅者们。类似的，订阅者的类族也考虑到了(也支持继承)；

Zero configuration: You can get started immediately using a default EventBus instance available from anywhere in your code.

0配置：你可以随时使用一个默认的可用EventBus实例，在你代码中的任何地方；

Configurable: To tweak EventBus to your requirements, you can adjust its behavior using the builder pattern.

支持可配置的：为了根据你的需求去定制化EventBus，你可以使用Build模式去调节EventBus的行为；
