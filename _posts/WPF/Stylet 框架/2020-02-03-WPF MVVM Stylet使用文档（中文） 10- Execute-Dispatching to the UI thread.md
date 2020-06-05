---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）10-Execute：Dispatching to the UI thread
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



## 简介（Summary）

`Execute` 是一个小的静态帮助类，它使得在UI线程上运行委托变得更容易。它包装了 `Application.Current.Dispatcher`，并提供使其更容易和更简洁使用的方法。

它还提供了一个属性的帮助类：`Execute.InDesignMode`。当且仅当 Visual Studio 或 Expression Blend 设计器处于活动状态，并且代码为了设计时显示提供虚拟数据时。

下表给出了它提供的方法的总结，后面给出了更深入的解释：

| Method                      | Inline if possible | Waits for Completion |
| --------------------------- | :----------------: | :------------------: |
| Execute.OnUIThread          |         ✔          |          ✘           |
| Execute.OnUIThreadSync      |         ✔          |      ✔ (Blocks)      |
| Execute.OnUIThreadAsync     |         ✔          |       ✔ (Task)       |
| Execute.PostToUIThread      |         ✘          |          ✘           |
| Execute.PostToUIThreadAsync |         ✘          |       ✔ (Task)       |

**Inline if possible**: 该方法将检查当前线程是否为UI线程。如果是，则委托将同步运行。如果不是，那么它将以某种形式被分派到UI线程。

**Waits for completion:** 要么阻塞直到委托完成执行，要么返回一个任务，该任务在委托完成执行时完成。



## 详细（Details）

### Execute.OnUIThread

检查当前线程是否为 UI 线程。如果是，则委托将同步运行。如果不是，委托将被分派到UI线程，并在将来的某个时候运行。在这种情况下，`Execute.OnUIThread` 将不会等待委托完成。

这反映了传统的模式:

``` C#
public static void InvokeIfRequired(Action action)
{
    if (Application.Current.Dispatcher.CheckAccess())
        action();
    else
        Application.Current.Dispatcher.BeginInvoke(action);
}
```



### Execute.OnUIThreadSync

检查当前线程是否为 UI 线程。如果是，那么它将同步运行委托。如果不是，那么它将分派委托在UI线程上运行，并且阻塞，直到它完成执行。

因此，它与 `Execute.OnUIThread` 非常相似。除了它会在委托完成执行后才返回。



### Execute.OnUIThreadAsync

检查当前线程是否为 UI 线程。如果是，那么它将同步运行委托，并返回一个已完成的任务。如果不是，那么它将分派委托在将来的某个时候在 UI 线程上运行，并返回一个任务，该任务将在委托完成执行后完成。

因此，它实际上是 `Execute.OnUIThreadSync` 的异步版本。



### Execute.PostToUIThread

无论当前线程是否为 UI 线程，都将在将来的某个时候在 UI 线程上发布要运行的委托。



### Execute.PostToUIThreadAsync

无论当前线程是否为 UI 线程，都将在将来的某个时候在 UI 线程上发布要运行的委托，并返回一个任务，该任务将在委托执行完成时完成。

**BEWARE** 你绝对不能做 `Execute.PostToUIThreadAsync(() => something(foo)).Wait()` 这样的事情。如果从UI线程执行此操作，将导致死锁。这种方法对 `Execute.PostXXX` 没有意义，请使用`Execute.OnUIThreadSync` or `Execute.OnUIThreadAsync` 来代替。



### 高级：单元测试（Advanced: Unit Testing）

### 分配器（The Dispatcher）

`Execute` 实际上有一个来自 `Application.Current.Dispatcher` 的抽象级别。

`Execute.Dispatcher` 是类型 `IDispatcher` 的静态属性，被 `Execute` 用于分派委托。

该属性永远不能为空，默认为 `IDispatcher` 的实现，它同步执行所有操作。然后在 `BootstrapperBase` 中覆盖它，成为 `Application.Current.Dispatcher` 的包装器。

这种行为意味着使用 `Execute` 方法的方法可以进行单元测试，或者在设计时使用。

在单元测试中，所有的 `Execute` 方法将同步运行它们的委托(因为调度程序不可用)。

如果需要，你也可以设置  `Execute.Dispatcher`, 为您的单元测试提供一个自定义的 `IDispatcher` 实现。



### 设计模式（Design Mode）

`Execute.InDesignMode` 也是可设置的，这将覆盖*“实际”*值。

预计您几乎永远都不需要这样做，但有时为了单元测试奇怪的小边缘情况(在Stylet中有一些这样的情况)，这种情况它是不可用的。