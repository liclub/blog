---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）01-Introduction
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---

在 Github 上发现一个开源轻巧的 MVVM 框架，发现网上缺少相应的中文文档，便进行了翻译总结，英文水平有限，MVVM也才初学，若有错误，还请斧正。

<!--more-->

## 概述

Stylet 是一个小巧，但强大的 MVVM 框架，它的灵感来自于 [Caliburn.Micro](https://caliburnmicro.com/)。*它通过进一步减少复杂性和各种事件*，让不熟悉任何 MVVM 框架的人(同事)更快地使用 MVVM 框架。

它还提供了在 Caliburn.Micro 里没有的功能。包括独有的【IoC容器】，更简单的【 ViewModel验证】和兼容 mvvm 的消息框架。

少量的代码（LOC : Line Of Code）和非常全面的测试组件使得 Sytlet 对于那些使用、验证或确认 SOUP 导致高开销的项目是一个很有吸引力的选择。Stylet 的模块化 toolkit-inspired 架构意味着它很容易让你自由使用，使用你喜欢的部分,或替掉你不喜欢的部分。

下面是一个简洁的特点列表。也可以点击链接了解更多信息。



## [视图模型优先（A ViewModel-First apporach）](https://github.com/canton7/Stylet/wiki/ViewModel-First)

典型的 MVVM 结构：视图（View）知道如何实例化它的视图模型（ViewModel），而 ViewModel 通常不直接通信，这种结构被称之为 视图优先（View-First）。然而，反转这个模式——使用者自己实例化 ViewModels，然后框架自动将 View 附加到实例化的 ViewModel 上——这种方式有很多优点，它允许你以一种非常熟悉的方式组合ViewModels。这种 ViewModel-First 方式是 Stylet 唯一支持的。 



## [响应（Action）]()

WPF 使用的 ICommand 接口非常强大，但是在 MVVM 体系结构中使用时显得有些笨拙。ViewModel 对一些过程的响应似乎是不对的，比如按钮的单击操作应该表示为属性，而不是方法。一个简单的语句`<Button Command="{s:Action DoSomething}"/>`使得在每次单击按钮时调用 ViewModel 上的 DoSomething() 。

另外，如果您有一个名为 CanDoSomething 的 bool 属性，它将被观察并用于判断按钮应该启用还是禁用。这被称为防护属性。

响应还可以与事件一起工作，允许您执行以下操作：`<Button MouseEnter="{s:Action DoSomethingElse}"/>`



## [ Screen 类和 Conductors ](noctiflorous.gitee.io)

Screen 类提供了很多功能，使得它成为 ViewModel 的一个很有吸引力的基类：属性改变通知，验证，当它 显示/隐藏/关闭 时被通知的功能，当它关闭时，是否能够被控制的功能。



## [事件聚合器（The Event Aggregator）]()

Stylet 的事件聚合器与 Caliburn 非常相似。允许订阅者在不知情或不保留对方的情况下接收发布的消息。这对于 viewmodel 之间的消息传递特别有用，当然它还有许多其他用途。



## [窗体管理器（The WindowManager）]()

使用 ViewModel-first 方式，通过引用要显示的 ViewModel 来显示窗口和对话框，视图会自动附加。WindowManager可以轻松地完成这一任务。

它还提供了一个mvvm兼容的消息框实现，因此您不必自己动手。



## [验证（Validation）]()

传统的 MVVM 中的验证有一个痛点：它需要在每个需要验证的视图模型中使用大量的样板文件，关于如何做好这一点的资料却很少。

Stylet 提供了一个框架来获取您最喜欢的验证库（例如[FluentValidation](https://fluentvalidation.codeplex.com/)），并处理运行的验证和向视图报告结果。



[Stylet 控制反转（StyletIoC）]()

Stylet 自带了自己的轻量级和极快(但仍然很强大)的IoC容器，当然如果您愿意，可以很容易地使用另一个IoC。



## [MIT 许可（MIT license）]()

Stylet 是在 MIT 许可下发布的，它允许您修改 Stylet，并将其包含在商业项目中，而不需要声明(惟一的限制是必须包含许可的副本)。如果你需要的话，我可以根据具体情况重新授权。

## 名词解释

1. IoC: Inversion of Control,控制反转
2. LoC: Line of Code，代码行
3. DI: Dependency Injection，依赖注入
4. MIT license: [软件授权条款之一，详见百度](https://baike.baidu.com/item/MIT%E8%AE%B8%E5%8F%AF%E8%AF%81/6671281?fr=aladdin)