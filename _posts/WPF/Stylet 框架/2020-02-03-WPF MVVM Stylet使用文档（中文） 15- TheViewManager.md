---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）15-The ViewManager
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



ViewManager 是 Stylet 的核心组件之一，负责获取 ViewModel，并为其定位正确的视图。然后实例化该视图，并将其绑定到 ViewModel。

本文将告诉您这个过程是如何工作的，以及如何修改它。

<!--more-->



## 定位视图（Locating a View）

默认的视图定位策略非常简单: 将在 ViewModel的类型和名称空间中将字符串 “ViewModel” 替换为字符串 “View”，然后搜索所有已配置的程序集，寻找具有该名称的类型。

就是这样。

“已配置程序集” 是指 ViewManager 的 viewassembly 属性中的所有程序集。如果您希望搜索更多的程序集，请将它们添加到此列表中(参见下面的示例)。

这个过程由 `ViewManager.LocateViewForModel` 控制。所以如果你想修改它，继续子类化 `ViewManager`——参见本文底部的章节。



## 创建视图（Creating a View）

一旦确定了视图类型，就通过调用提供给 ViewManager 的工厂方法来实例化它。在默认情况下，引导程序将其设置为调用 IoC 容器。然后调用 `InitializeComponent`，这意味着如果需要，您可以删除您的后台代码。

这是由`ViewManager.CreateViewForModel` 处理的。你也可以覆盖它。



## 将视图绑定到它的视图模型（Binding the View to its ViewModel）

绑定视图模型及其对应的视图非常简单:

- 视图的 DataContext 被设置为 ViewModel。

- 视图的 ActionTarget 被设置为 ViewModel(参见 [[Actions]])。

- 如果 ViewModel 实现了 `ViewAware`，则调用它的 `AttachView` 方法，并传递视图(参见 [[Screens and conductor]])。



## 处理 `View.Model` 的更改（Dealing with `View.Model` Changes）

ViewManager 还负责响应 `ContentControl` 中 `View.Model`  附加属性的更改——参见 [[ViewModel First]]。

它将获取附加属性的新值(应该绑定到 ViewModel 实例)，并使用 `CreateViewForModel` 和 `BindViewToModel` 创建一个新的配置视图实例，然后将其设置为 `ContentControl` 的内容。



## 配置视图管理器（Configuring the ViewManager）

ViewManager 是可配置的。要配置它，在你的 `Configure` 方法中获取它的一个实例:

```csharp
protected override void Configure()
{
    var viewManager = this.Container.Get<ViewManager>();
    // ...
}
```

一旦你有了它，有几个事情你可以配置:

- `ViewSuffix`: 视图类名的后缀。默认为 “View”。

- `ViewModelSuffix`: 视图类名的后缀。默认为 “ViewModel”。

- `Viewassembly`: 要扫描哪些程序集的视图。默认包含引导程序的程序集。

- `namespacetransformation`: 这是一个 `from -> to` 替换的字典，允许你的视图和视图模型存在于不同的名称空间中。如果你的视图位于`Foo.Frontend.Views` 中。视图模型位于 `Foo.FrontendLogic.ViewModels` 中。你可以添加一个条目（键值对）到这个关键字为 "Foo.Frontend“ 和值为 “Foo.FrontendLogic”字典。注意，您必须包含所有的根名称空间: 您不能在这里使用 从 “Frontend” 到 “FrontendLogic” 的转换。



## 创建自己的 ViewManager（Creating your own ViewManager）

当 Stylet 需要 ViewManager 时，它从 IoC 容器中检索一个 `IViewManager` 实例。这意味着您可以创建一个您想要的 IViewManager 实现，并将其注册到 IoC 容器中，Stylet 将获取并使用它。

一个非常简单的例子:

```csharp
// CustomViewManager.cs

public class CustomViewManager : ViewManager
{
   public CustomViewManager(ViewManagerConfig config)
      : base(config)
   { }

   public override UIElement CreateAndSetupViewForModel(object model)
   {
      // This dummy application only has one view, and it's this one :)
      return new SingletonView();
   }
}

// Bootstrapper.cs

public class Bootstrapper : Bootstrapper<MyRootViewModel>
{
   protected override void ConfigureIoC(IStyletIoCBuilder builder)
   {
      builder.Bind<IViewManager>().To<CustomViewManager>();
   }
}
```

这是它!

更完整的示例可以在 [`Stylet.Samples.OverridingViewManager` sample](https://github.com/canton7/Stylet/tree/master/Samples/Stylet.Samples.OverridingViewManager)。

