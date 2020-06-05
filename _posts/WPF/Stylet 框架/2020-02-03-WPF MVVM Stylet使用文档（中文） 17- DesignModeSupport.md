---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）17-Design Mode Support
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---





## 简介（Introduction）

“设计模式”或“设计时”指的是项目加载到 Visual Studio XAML 设计器或 Expression Blend 中时，显示的是 XAML 的呈现版本。大多数情况下，设计人员不会尝试评和估任何绑定，也不会为它们提供任何智能感知。然而，通过一些配置，您可以获得可爱的智能感知，并在视图中显示来自 ViewModel 的一些虚拟值。

Stylet 对设计模式有一些基本的支持。本文记录了它，并提供了如何使用它以及如何利用现有的 XAML 特性来增强设计时体验的说明。

<!--more-->

这里显示的所有示例都可以在 [DesignMode sample project](https://github.com/canton7/Stylet/tree/master/Samples/Stylet.Samples.DesignMode)中的 "ready to run" 里面找到。



## 只有智能感知，没有绑定（IntelliSense only, no bindings）

这是最基本的技巧，你只需要做很少的额外工作。您将获得绑定的智能感知(至少在 Visual Studio 2013 及以上版本中)，但是您不会在视图中看到来自 ViewModel 的任何虚拟数据。

首先，需要在视图的根目录中声明以下内容。如果你在 Visual Studio 2013 中创建了一个 UserControl，这些将会被默认添加。

``` XML
xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
mc:Ignorable="d" 
```

你还需要为你的视图模型添加一个命名空间:

```xml
xmlns:vms="clr-namespace:DesignMode.ViewModels"
```

一旦你有了这个，你需要一个额外的神奇的一行，它告诉 XAML 设计器，这个视图的 `DataContext` 是你的 `SampleViewModel`，绑定智能感知应该使用这个属性:

```xml
d:DataContext="{d:DesignInstance vms:SampleViewModel}"
```

把它们放在一起，你会得到这样的结果:

``` XML
<UserControl x:Class="DesignMode.Views.SampleView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             mc:Ignorable="d"
             xmlns:vms="clr-namespace:DesignMode.ViewModels"
             d:DataContext="{d:DesignInstance vms:SampleViewModel}">
   ...
</UserControl>
```



## 智能感知和虚拟数据，让设计器实例化视图模型（Intellisense and Dummy Data, letting the Designer instantiate the ViewModel）

除了让 XAML 设计人员为我们实例化 ViewModel 之外，这种技术与前面的技术非常相似。设计器将使用这个ViewModel 实例来获取绑定的虚拟数据。

为了让设计器能够做到这一点，ViewModel 必须有一个无参数的构造函数。这既是祝福也是诅咒。好的方面是，它为您提供了一个将一些虚拟数据注入 ViewModel 属性以供设计人员使用的好地方。不好的一面是你的 ViewModel 现在包含了只会被设计师使用的代码…

> **注意：**在设计时没有访问 IoC 容器的权限(任何请求访问的人将被带到后台…处理)。因此，如果您的ViewModel 有任何依赖项，它们在设计时将不可用。通常这不是问题: 只有属性被访问(没有方法被调用)，所以您不应该做任何需要访问某个依赖项的事情。只要把它记在心里。

以这种方式编写的支持设计模式的示例视图模型如下所示:

``` C#
public class SampleViewModel
{
    private readonly IUserService userService;

    public string CurrentUserName { get; private set; }

    public SampleViewModel()
    {
        this.CurrentUserName = "Dummy Username";
    }

    public SampleViewModel(IUserService userService)
    {
        this.userService = userService;
        this.CurrentUserName = this.userService.CurrentUser.UserName;
    }
}
```

>  **注意：**StyletIoC 总是会选择具有最多可解析参数的构造函数，它也会调用接受 `IUserService` 的重载。另一方面，设计器总是调用无参数的构造函数。

如果 ViewModel 通常只有一个无参数的构造函数，那么可以使用 `Execute.InDesignMode` 或者 `Execute: Dispatching to the UI thread`，如下:

``` C#
public class SampleViewModel
{
    public string SomeText { get; set; }

    public SampleViewModel()
    {
        if (Execute.InDesignMode)
            this.SomeText = "Dummy Text";
        else
            this.SomeText = "Actual Text";
    }
}
```

不管怎样，一旦你有了一个 ViewModel 的无参数构造函数，设计师可以使用，你可以告诉设计师实例化它使用:

``` XML
d:DataContext="{d:DesignInstance vms:SampleViewModel, IsDesignTimeCreatable=True}"
```

或者，如下完整版本：

``` XML
<UserControl x:Class="DesignMode.Views.SampleView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             mc:Ignorable="d"
             xmlns:vms="clr-namespace:DesignMode.ViewModels"
             d:DataContext="{d:DesignInstance vms:SampleViewModel, IsDesignTimeCreatable=True}">
   ...
</UserControl>
```



## 智能感知我虚拟数据，使用视图模型定位器 （Intellisense and Dummy Data, using a ViewModelLocator）

前面的方法有一个很大的缺点: 它要求 ViewModel 知道设计模式，并且包含只在设计时调用的代码。这被一些人认为是一种巨大的代码味道。

另一种方法是使用 ViewModelLocator——一个只在设计时使用的类，它可以实例化和配置您的 viewmodel。这意味着任何设计时都可以进入 ViewModelLocator，并且不进入各个视图。

如果这听起来很复杂，请原谅我。这一切在一分钟内就会变得明晰。

首先，让我们构建一个 SampleViewModel：

``` C#
public class SampleViewModel
{
    private readonly IUserService userService;

    public string CurrentUserName { get;set; }

    public SampleViewModel(IUserService userService)
    {
        this.userService = userService;
        this.CurrentUserName = this.userService.CurrentUser.UserName;
    }
}
```

拉下来，我们需要一个 ViewModelLocator，这是一个简单的类，每个视图模型包含一个属性，我们可能想在设计时访问:

``` C#
public class ViewModelLocator
{
    public SampleViewModel SampleViewModel
    {
        get
        {
            var vm = new SampleViewModel();
            vm.CurrentUserName = "Dummy Username";
            return vm;
        }
    }
}
```

注意到 ViewModel 是如何仅在需要时实例化的吗? 这是因为 ViewModelLocator 本身将在运行时实例化，但是它的属性将只在设计时访问。

接下来，让我们把它添加到我们的应用程序的资源中，这样它就可以用于我们的视图:

``` XML
<Application x:Class="DesignMode.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet"
             xmlns:local="clr-namespace:DesignMode">
    <Application.Resources>
        <s:ApplicationLoader>
            <s:ApplicationLoader.Bootstrapper>
                <local:Bootstrapper/>
            </s:ApplicationLoader.Bootstrapper>
            
            <local:ViewModelLocator x:Key="ViewModelLocator"/>
        </s:ApplicationLoader>
    </Application.Resources>
</Application>
```

然后在视图中使用：

``` XML
<UserControl x:Class="DesignMode.Views.SampleView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
             mc:Ignorable="d"
             xmlns:vms="clr-namespace:DesignMode.ViewModels"
             d:DataContext="{Binding Source={StaticResource ViewModelLocator}, Path=SampleViewModel}">
   ...
</UserControl>
```



## 在设计时启用/不启用按钮（Enabling/Disabling Buttons at Design Time）

在上面的所有例子中，我们只绑定了视图的 `DataContext` ：我们没有对它的 “View. actiontarget |Actions#the-viewactiontarget” 做任何事情。这意味着按钮的可启用性不会反映保护属性的值(如果它存在的话)——它总是被启用的。

**注意：**默认情况下，Stylet 会绑定视图。当它实例化视图时，`ActionTarget` 指向相应的视图模型。然而，在设计时，Stylet 并不负责实例化视图，因此 `View.ActionTarget` 不绑定。

如果你想让按钮的 enabledless 反映它的保护属性，你需要添加: `s:View.ActionTarget="{Binding}"`到你的视图，例如:

``` XML
<UserControl x:Class="DesignMode.Views.SampleView"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:s="https://github.com/canton7/Stylet"
             mc:Ignorable="d"
             xmlns:vms="clr-namespace:DesignMode.ViewModels"
             d:DataContext="{d:DesignInstance vms:SampleViewModel, IsDesignTimeCreatable=True}"
             s:View.ActionTarget="{Binding}">
   ...
</UserControl>
```



## 使用替代视图模型（Using Substitute ViewModels）

另一个解决"视图模型知道设计时，但它其它不应该知道"这个问题的办法是你的视图实现一个接口(真正的接口,或在头部假装实现一个)，然后，构造 design-time-only 视图并实现相同的接口，同时包含虚拟数据。然后在设计时以相同的方式绑定它: `d:DataContext="{d:DesignInstance vms:DummyViewModel, IsDesignTimeCreatable=True}"`。但是，对于大多数开发人员来说，这样的开销太大了。

WPF也有一个设计时数据的概念，[参见例子](http://blogs.msdn.com/b/wpfsldesigner/archive/2010/06/30/sample-data-wpf -and-silverlight-designer.aspx)。



## 为什么 Stylet 不能自动找到我的 ViewModel ?（Why can't Stylet find my ViewModel automatically?）

由于 Stylet 能够获取一个 ViewModel，并查找和实例化它的视图，您可能会想问为什么它不能以另一种方式做到这一点: 也就是说，在设计时，为给定的视图自动找到正确的 ViewModel，并用正确的依赖项实例化它。这是一个非常糟糕的想法，原因有很多:

1. 我们需要添加一种将 View 名称转换为 ViewModel 名称的方法，这增加了 `ViewManager` 的复杂性(特别是对于任何提供自己的 `ViewManager` 的人来说)。

2. 我们需要在设计时实例化一个适当的 `IViewManager`实现。由于用户可以提供自己的实现，这意味着打开整个 IoC 容器。因为这依赖于正确地设置了引导程序的程序集属性，所以我们需要打开整个引导程序。这可能会产生令人讨厌的副作用(请考虑启动执行网络提交、文件系统访问等的服务)。

3. 我们需要提供一个具有所有依赖关系的 ViewModel。这意味着实例化服务，这可能会产生令人讨厌的副作用。

4. ViewModel 可能不会包含合适的虚拟数据，所以我们没有获得太多。

5. 由于我们太过聪明而导致某些服务以错误的方式启动，因此出现设计器错误(或者更糟，出现网络/文件系统副作用)，调试起来很痛苦，我们不应该把这种情况强加于人。



## s:View.Model and Embedded Views

如果你有一个内嵌视图(例如 `<ContentControl s:View.Model="{Binding SomeChildViewModel}"/>`)在您的视图中，然后 Stylet 将在 View for ViewModelType.SomeChildViewModel 行简单地显示一条消息。而不是试图找到正确的视图。这与我们没有自动定位 ViewModels 的原因非常相似: 这意味着打开 IoC 容器(我们需要找到用户的 `IViewManager` 实现)，这意味着运行引导程序，这让我们可以做真正危险的事情。最好避免。