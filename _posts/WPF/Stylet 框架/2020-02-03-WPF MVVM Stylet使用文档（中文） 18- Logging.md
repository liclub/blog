---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）18-Logging
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



## 简介

有时候，查看 Stylet 在幕后做了什么是很有用的，特别是当它没有做一些你期望它做的事情，或者做一些意想不到的事情的时候。

值得庆幸的是，Stylet 可以很容易地配置来生成日志输出，因此您可以了解它在做什么。

<!--more-->



## 快速启动

要快速启用日志记录，请在Bootstrapper的配置方法中放入以下内容:

```C#
protected override void OnStart()
{
   Stylet.Logging.LogManager.Enabled = true;
}
```

这将把日志消息打印到 Visual Studio 的输出窗口。在内部，默认的日志记录器使用 `Trace.WriteLine`。



## 定制日志

当然，您可以向 Stylet 提供自己的日志记录器，Stylet 将使用它来打印日志消息。

首先，定义一个实现 `Stylet.Logging.ILogger` 接口的类:

```csharp
public class MyLogger : Stylet.Logging.ILogger
{
   public MyLogger(string loggerName)
   {
      // TODO
   }

   public void Info(string format, params object[] args)
   {
      // TODO
   }

   public void Warn(string format, params object[] args)
   {
      // TODO
   }

   public void Error(Exception exception, string message = null)
   {
      // TODO
   }
}
```

然后，配置 LogManager 来使用它。与之前一样，在你的Bootstrapper的配置方法:

```csharp
protected override void OnStart()
{
   Stylet.Logging.LogManager.LoggerFactory = name => new MyLogger(name);
   Stylet.Logging.LogManager.Enabled = true;
}
```



## 在应用程序中进行日志记录（Logging within your Application）

我建议不要用 Stylet.Logging 在其他地方进行日志记录。它是非常轻量级的，几乎没有任何特性——编写它的唯一原因是为了让 Stylet 不依赖于日志框架来支持一个几乎永远不会被使用的特性。

[NLog](http://nlog-project.org/) 和 [log4net](http://logging.apache.org/log4net/) 是两个主要的c#日志记录框架。如果您不想将您的应用程序与任何特定的日志记录框架耦合，那么可以考虑 [Common.Logging](https://github.com/net-commons/common-logging)，它提供了一个与框架无关的接口，在这个接口后面可以连接 NLog、log4net或其他框架。

