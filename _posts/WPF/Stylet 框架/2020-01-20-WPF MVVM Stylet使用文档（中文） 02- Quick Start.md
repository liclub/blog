---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）02-Quick Start
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



想要尽快让 stylet 运行起来吗?就是这里!

<!--more-->

> 注意:如果您正在寻找示例应用程序，请下载源代码并查看Samples 文件夹。

下面的介绍将建立一个最小的框架项目。



## [自动设置(AutoMatic Option)]()

### .NET Framework

> 注意:如果您的项目 NuGet 包使用的是 PackageReference，或者您使用的是 VS2013 或更早的版本，这将不起作用。请改用下面的“手动选项”部分。

如果您是 Stylet 新手(并且正在运行 VS2015 或更高版本)，这是最简单的入门方法。

1. 打开 Visual Studio，并创建一个新的 WPF 应用程序项目。

2. 打开 NuGet (右键单击你的项目->管理NuGet包)，并搜索并安装`Stylet.start`包。

这将为您提供一个工作框架项目。

安装完成后，卸载style . start。

编码快乐!

### .NET Core

对于 .NET Core 项目，最快的入门方法是使用 `dotnet new`和 Stylet 的模板。

打开一个命令窗口，并定位到你想安装新项目的位置

1. 使用下面命令安装 Stylet 模板:

    ```
    dotnet new -i Stylet.Templates
    ```

2. 使用下面的命令新建一个工程

    ```
    dotnet new stylet -o MyStyletProject
    ```

3. 适当更改 MyStyletProject



## [手动设置]()

如果你不想用 Stylet.Start 包并希望创建自己的框架项目，请按照下面的的步骤：

1. 打开 Visual Studio，并创建一个新的WPF应用程序项目。

2. 打开 NuGet(右键单击你的项目->管理NuGet包)，并安装 Stylet包。

3. 首先，删除 MainWindow.xaml 和 MainWindow.xaml.cs / vb。你不需要他们。

4. 接下来，您需要一个根 View 和一个 ViewModel。根 View 必须是一个 `Window` 或者 继承于 `Window` ，除了这个，没有其它任何限制，View 如下：

    ```
    <Window x:Class="Stylet.Samples.Hello.RootView"
            xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
            xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
            Height="300" Width="300">
        <TextBlock>Hello, World</TextBlock>
    </Window>
    ```

> ViewModel 可以是任何一个旧类(现在—您可能希望它是一个 Screen 或 Conductor)。
>
> | C#                               | VB.NET                                   |
> | -------------------------------- | ---------------------------------------- |
> | public class RootViewModel <br>{ <br>} | Public Class RootViewModel <br><br>End Class > |

5. 接下来，您需要一个引导程序。现在，您不需要任何特殊的东西——只需要一些东西来标识根 ViewModel。然后，您将能够在这里配置 IoC容器 以及其他应用程序级的内容，如下：

	| C# | VB.NET |
	| ----|----|
	| public class Bootstrapper : Bootstrapper<RootViewModel> <br>{<br> } | Public Class Bootstrapper     Inherits Bootstrapper(Of RootViewModel)   <br><br>End Cl |

6. 最后，需要将其作为资源引用到您的 App.xaml 中。您需要删除 StartUri 属性，并为 Stylet 和您自己的应用程序添加 xmlns 条目。最后，您需要将 Stylet 的 ApplicationLoader 添加到资源中，并识别您在上面创建的引导程序 (Bootstrapper)。

   像下面这样：

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

就是这样!运行它，你会得到一个带有 Hello World 的窗体。



## [程序加载器(The ApplicationLoader)]()

值得注意的是，上面的`<s:ApplicationLoader>`是 ResourceDictionary 子类。这允许它加载 Stylet 的嵌入资源(参见 Screen 和  Conductors )。你可以选择不加载 Stylet 的资源，像这样:

```html
<s:ApplicationLoader LoadStyletResources="False">
   ...
</s:ApplicationLoader>
```

如果你想添加自己的 Resource/ResourceDictionaries 到应用程序，最简单的方法是这样的:

```html
<Application.Resources>
    <s:ApplicationLoader>
        <s:ApplicationLoader.Bootstrapper>
            <local:Bootstrapper/>
        </s:ApplicationLoader.Bootstrapper>

        <Style x:Key="MyResourceKey">
            ...
        </Style>

        <s:ApplicationLoader.MergedDictionaries>
            <ResourceDictionary Source="MyResourceDictionary.xaml"/>
        </s:ApplicationLoader.MergedDictionaries>
    </s:ApplicationLoader>
</Application.Resources>
```

如果这让你感到不舒服，你也可以这样做:

```html
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <s:ApplicationLoader>
                <s:ApplicationLoader.Bootstrapper>
                    <local:Bootstrapper/>
                </s:ApplicationLoader.Bootstrapper>
            </s:ApplicationLoader>

            <ResourceDictionary Source="MyResourceDictionary.xaml"/>
        </ResourceDictionary.MergedDictionaries>

        <Style x:Key="MyResourceKey">
            ...
        </Style>
    </ResourceDictionary>
</Application.Resources>
```