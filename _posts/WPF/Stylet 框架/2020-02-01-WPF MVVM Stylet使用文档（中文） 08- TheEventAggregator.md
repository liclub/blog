---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）08-The EventAggregator
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



## 简介

事件聚合器——EventAggregator——是一个分散的、弱绑定的、基于发布/订阅的事件管理器。

<!--more-->

## 发布者和订阅者（Publishers and Subscribers）

### 订阅者（Subscribers）

对特定事件感兴趣的订阅者可以将自己的兴趣告诉 IEventAggregator，并且每当发布者将特定事件发布到IEventAggregator 时，都会收到通知。

事件是类——可以用它们做任何你想做的事情。例如:

<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
class MyEvent { 
&nbsp;
  // Do something 
&nbsp;
}</pre>
</td><td valign="top"><pre lang="vb.net">
Class MyEvent
&nbsp;
  &#39; Do Something
&nbsp;
End Class</pre></td></tr></table>

订阅者必须实现 `IHandle<T> `，其中 `T` 是他们感兴趣的事件类型 (当然，他们可以实现多个 `IHandle<T>` 's for多个 `T` 's)。然后他们必须获得 IEventAggregator 的实例，并订阅自己，例如:

<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
class Subscriber : IHandle&lt;MyEvent&gt;, IHandle&lt;MyOtherEvent&gt;
{
   public Subscriber(IEventAggregator eventAggregator)
   {
      eventAggregator.Subscribe(this);
   }
&nbsp;
   public void Handle(MyEvent message)
   {
      // ...
   }
&nbsp;
   public void Handle(MyOtherEvent message)
   {
      // ...
   }
}</pre>
</td><td valign="top"><pre lang="vb.net">
Class Subscriber : Implements IHandle(Of MyEvent)
&nbsp;
  Public Sub New(ByRef eventAggregator as IEventAggregator)
  eventAggregator.Subscribe(Me)
  End Sub
&nbsp;
  Public Sub Handle(message as MyEvent) Implements IHandle(Of MyEvent).Handle
  &#39; ...
  End Sub
&nbsp;
  Public Sub Handle(message as MyOtherEvent) Implements IHandle(Of MyOtherEvent).Handle
  &#39; ...
  End Sub
&nbsp;
End Class</pre></td></tr></table>

VB.NET 用户，通过引用传递 eventAggregator 的 `Sub New()` 在跨命名空间时可能会失败，而且必须定义每个新订阅者，这可能会令人恼火。因此，在全局模块中定义eventAggregator，然后直接订阅它，而不是将其引用传递给调用的每个新 ViewModel，可能更容易。

``` vb
Module Global
  Public eventAggregator as IEventAggregator
End Module

Class Subscriber : Implements IHandle(Of MyEvent)

  Public Sub New()
  Global.eventAggregator.Subscribe(Me)
  End Sub
  
  'Public Sub Handle...

End Class
```

确保将 *module* 的名称空间保留为空，以便可以在整个程序中使用它。



### 发布者（Publishers）

发布者也必须获得 IEventAggregator 的实例，但他们不需要订阅自己—只需在每次发布想要发布的事件时调用 `IEventAggregator.Publish` ，例如:

<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
class Publisher
{
   private IEventAggregator eventAggregator;
   public Publisher(IEventAggregator eventAggregator)
   {
      this.eventAggregator = eventAggregator;
   }
&nbsp;
   public void PublishEvent()
   {
      this.eventAggregator.Publish(new MyEvent());
   }
}</pre>
</td><td valign="top"><pre lang="vb.net">
Class Publisher 
&nbsp;
  Dim eventAggregator as IEventAggregator
&nbsp;
  Public Sub New(ByRef eventAggregator as IEventAggregator)
    Me.eventAggregator = eventAggregator
  End Sub
&nbsp;
  Public Sub PublishEvent()
  Me.eventAggregator.Publish(New MyEvent())
  End Sub
&nbsp;
End Class</pre></td></tr></table>

再次,VB.NET用户，如果您已经设置了全局模块，那么您不需要将 eventAggregator 传递给发布者。你可以直接发布到全局事件聚合器:

``` vb
Class Publisher

  Public Sub PublishEvent()
  Global.eventAggregator.Publish(New MyEvent())
  End Sub
  
End Class
```



## 取消订阅和弱绑定（UnSubscribing adn weak binding）

因为 IEventAggregator 是弱绑定的，订阅者不需要取消订阅— IEventAggregator 不会保留它们。但是，如果订阅者想要取消订阅，也可以取消订阅。如下：

<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
IEventAggregator.Unsubscribe(this);</pre>
</td><td valign="top"><pre lang="vb.net">
IEventAggregator.UnSubscribe(Me)</pre></td></tr></table>



## 同步和异步发布（Publishing synchronously and asynchronously）

默认的 `IEventAggregator.Publish` 方法是同步发布事件。您还可以调用 `PublishOnUIThread` 来异步地调度UI 线程，或者调用 `PublishWithDispatcher`，并传递您希望充当 dispatcher 的任何操作(如果在IEventAggregator上编写自己的方法，这将非常有用)。



## 频道（Channels）

订阅者可以侦听特定的频道，发布者可以将事件发布到特定的频道。如果将事件发布到特定的频道，则只有已订阅该频道的订阅者才能接收该事件。如果在几个不同的上下文中使用相同的消息类型，那么这将非常有用。

频道是字符串，因此允许一个频道的订阅者和该频道的发布者之间的松散耦合。

默认情况下，`Subscribe()` 将订阅方订阅到默认频道 `EventAggregator.DefaultChannel`。类似地， `Publish() ` (及其所有变体) 将把事件发布到相同的默认频道。然而，你也可以指定自己的频道。



### 订阅到频道（Subscribing to channels）

若要订阅特定频道，请将其作为参数传递给 `subscribe”`: `eventAggregator.Subscribe(this,“ChannelA”) `。你也可以订阅多个频道: `eventAggregator.Subscribe(this,“ChannelA”,“ChannelB”)`。

在这两种情况下，你都不会订阅到 `EventAggregator.DefaultChannel ` -只订阅到指定的频道。你也只会收到被推送至“ChannelA”或“ChannelB”的事件。



### 发布到频道（Publishing to channels）

若要发布到特定通道，请将其作为参数传递给`publish`: `eventAggregator.Publish(message，“ChannelA”)` 或 `eventAggregator.PublishOnUIThread(message，“ChannelA”，“ChannelB”)`，等等。与上面的订阅一样，事件将发布到所有指定的通道，而不再是默认通道。



### 从频道取消订阅（Unsubscribing from channels）

要取消频道订阅，请将其传递给 `Unsubscribe`: `eventAggregator.Unsubscribe(this, "ChannelA")`。您将继续订阅您以前订阅的且没有取消订阅的任何其他频道。

调用 `eventAggregator.Unsubscribe(this)` 将从*所有* 频道取消订阅。



## 使用自己的 IoC 容器（Using your own IoC container）

如果你在 StyletIoC 中使用默认的 `Bootstrapper<TRootViewModel>`，你不需要担心这个——EventAggregator 在默认情况下是正确设置的。

如果你使用另一个 IoC 容器,那么,你需要确保 EventAggregator 注册为 IEventAggregator 的独立服务， EventAggregator 只能有一个实例,每次请求的时候，都必须返回这个实例。