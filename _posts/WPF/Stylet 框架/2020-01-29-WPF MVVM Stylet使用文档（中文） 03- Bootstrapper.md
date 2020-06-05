---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）03-Bootstrapper
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---

## 简介

引导程序负责启动应用程序。它配置 IoC 容器，创建根 ViewModel 的新实例，并使用 WindowManager 显示它。

<!--more-->

它还提供了其他各种功能，如下所述。

引导程序有两种形式:

1. `BootstrapperBase<TRootViewModel>`，这种需要您自己配置 IoC 容器;
2. `Bootstrapper <TRootViewModel>`，使用 Stylet 的内置 IoC 容器 StyletIoC。

示例 Bootstrapper，使用 StyletIoC:

```C#
class Bootstrapper : Bootstrapper<MyRootViewModel>
{
   protected override void OnStart()
   {
      // This is called just after the application is started, but before the IoC container is set up.
      // Set up things like logging, etc
      //程序启动之后，IoC窗口配置之前调用
      //进行一些设置，比如日志
   }
 
   protected override void ConfigureIoC(IStyletIoCBuilder builder)
   {
      // Bind your own types. Concrete types are automatically self-bound.
      builder.Bind<IMyInterface>().To<MyType>();
   }
 
   protected override void Configure()
   {
      // This is called after Stylet has created the IoC container, so this.Container exists, but before the Root ViewModel is launched.
      // Configure your services, etc, in here
   }
 
   protected override void OnLaunch()
   {
      // This is called just after the root ViewModel has been launched
      // Something like a version check that displays a dialog might be launched from here
   }
 
   protected override void OnExit(ExitEventArgs e)
   {
      // Called on Application.Exit
   }
 
   protected override void OnUnhandledException(DispatcherUnhandledExceptionEventArgs e)
   {
      // Called on Application.DispatcherUnhandledException
   }
}
```



## [使用自定义的 IoC 容器(Using a Custom IoC Container)]()

使用带有 Stylet 的另一个 IoC 容器很容易。我在 [Bootstrappers project](https://github.com/canton7/Stylet/tree/master/Bootstrappers) 中包含了许多流行的 IoC 容器的bootstrappers。这些都是经过单元测试而非实战测试的: 您可以随意定制它们。

注意，Stylet nuget package/dll 不包括这些，因为它会添加不必要的依赖。同样，我也不会发布特定的 IoC 容器的包，因为这是一种浪费。

将您想要的引导程序从上面的链接复制到您的项目中。然后子类化它，就像您通常子类化`Bootstrapper<TRootViewModel>`，上面有文档说明。然后将子类添加到 App.xaml.cs 中。正如 [Quick Start]() 中说明的那样。

```c#
public class Bootstrapper : AutofacBootstrapper<ShellViewModel>
{
}
```

```html
<Application x:Class="Stylet.Samples.Hello.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet"
             xmlns:local="clr-namespace:Stylet.Samples.Hello">
    <Application.Resources>
       <s:ApplicationLoader>
            <s:ApplicationLoader.Bootstrapper>
                <local:Bootstrapper/>
            </s:ApplicationLoader.Bootstrapper>
        </s:ApplicationLoader>
    </Application.Resources>
</Application>
```

如果您想为另一个 IoC 容器编写自己的引导程序，这也很容易。看看上面的描述，看看需要做什么。



## [向 App.xmal 添加资源字典(Adding Resource Dictionaries to App,xmal)]()

`s:ApplicationLoader` 本身就是一个 ResourceDictionary。如果你需要将自己的资源字典添加到 App.xaml 中，你需要将`s:ApplicationLoader` 嵌入到你的 ResourceDictionary 中作为一个合并字典，就像这样:

```C#
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <s:ApplicationLoader>
                <s:ApplicationLoader.Bootstrapper>
                    <local:Bootstrapper/>
                </s:ApplicationLoader.Bootstrapper>
            </s:ApplicationLoader>

            <ResourceDictionary Source="YourDictionary.xaml"/>
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

