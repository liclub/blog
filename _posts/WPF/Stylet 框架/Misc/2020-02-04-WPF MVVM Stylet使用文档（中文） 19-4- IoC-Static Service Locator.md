---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）19-4 IoC：Static Service Locator
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



<!--more-->

Caliburn.Micro 附带了一个名为 IoC 的静态服务定位器。这让你可以从代码中的任何地方访问 IoC 容器，就像这样:

```C#
var vm = IoC.Get<MyDialogViewModel>();
this.windowManager.ShowDialog(vm);
```

Stylet 没有包含这个，而且有很好的理由:我不想鼓励人们编写如此糟糕的代码。服务定位器模式经常被称为反模式。现在每个类都有一个对 IoC 的依赖(而不是它所依赖的实际类)，您不能仅通过查看类的构造函数就知道它的依赖关系是什么，相反，您必须遍历代码以才能得知它对`IoC.Get`的调用。

IoC 也在 Caliburn 内部使用导致产生了一些糟糕的设计选择。这些已经在 Stylet 中重新架构，因此内部不再需要IoC。

如果你真的需要 IoC 的支持(尽管它会导致糟糕的代码风格)，那么你可以很容易地编写自己的 IoC。首先创建这个静态 IoC 类:

```C#
public static class IoC
{
    public static Func<Type, string, object> GetInstance = (service, key) => { throw new InvalidOperationException("IoC is not initialized"); };

    public static Func<Type, IEnumerable<object>> GetAllInstances = service => { throw new InvalidOperationException("IoC is not initialized"); };

    public static Action<object> BuildUp = instance => { throw new InvalidOperationException("IoC is not initialized"); };

    public static T Get<T>(string key = null)
    {
        return (T)GetInstance(typeof(T), key);
    }

    public static IEnumerable<T> GetAll<T>()
    {
        return GetAllInstances(typeof(T)).Cast<T>();
    }
}
```

然后在引导程序中添加下面的代码：

```C#
protected override void Configure()
{
   IoC.GetInstance = this.Container.Get;
   IoC.GetAllInstances = this.Container.GetAll;
   IoC.BuildUp = this.Container.BuildUp;
}
```

