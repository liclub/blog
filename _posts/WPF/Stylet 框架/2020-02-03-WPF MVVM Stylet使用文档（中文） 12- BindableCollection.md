---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）12-BindableCollection
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---





## 概述（Overview）

`BindableCollection<T> ` 是 [`ObservableCollection<T>`](http://msdn.microsoft.com/en-us/library/ms668604%28v=vs.110%29.aspx) 的一个子类。如果你在你的 ViewModel 中有一个集合，并且想要将它用作你视图中的某个控件的 `ItemsSource` 等等，那么你就可以使用这个类（当一个项目被添加到/从集合中移除时，视图会得到通知）。

<!--more-->

而且，它增加了一些有用的额外功能:



- 新增`AddRange`、`RemoveRange`和`Refresh`方法

- 是线程安全的



## 新方法

`ObservableCollection<T> `缺少两个非常有用的方法: `AddRange` 和 `RemoveRange ` 。

它们的作用与您的预期相差无几，允许您立即添加/移除范围内的元素，而无需手动遍历每个元素，并在每个元素添加时调用 `collection.Add (element)`(这使得触发大量的元素添加事件)。`AddRange` 和 `RemoveRange` 只会在每个范围增加/删除时引发一组事件。

`刷新`方法很方便。

它不会以任何方式修改集合，但会触发 `PropertyChanged` 和 `CollectionChanged` 事件，向任何 UI 元素表明集合已被修改，它们应该重新加载数据。

这不是经常需要的，但当它真的有需要的时候。



## 线程安全（Thead Safety）

线程安全是通过将所有操作 (添加、删除、清除、重置等) 分派给 UI 线程来实现的。分派使用`Execute.OnUIThreadSync`，意思是:

- 这些操作是同步的:被调用的方法在操作完成之前不会返回。

- 它们是自由的，如果你已经在 UI 线程，操作将在这种情况下同步进行。

- 所有 `PropertyChanged` 和 `CollectionChanged ` 事件总是在 UI 线程上引发。

这最后一点意味着与 `PropertyChagedBase` 有 `PropertyChangedDispatcher` 不同，`BindableCollection < T >` 上是没有 `PropertyChangedDispatcher`  属性的——事件总是在 UI 线程上,因为操作的属性与总是在 UI 线程上执行，所以，也没有 `CollectionChangedDispatcher` 概念。

