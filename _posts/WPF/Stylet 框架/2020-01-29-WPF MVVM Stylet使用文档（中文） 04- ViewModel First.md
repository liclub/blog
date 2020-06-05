---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）04-ViewModel First
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---

## 简介

ViewModel-first 方式对于 Stylet 的体系结构非常重要，但是如果您以传统的 View-first 方式学习MVVM，就会发现它并不直观。

<!--more-->

希望这篇文章能把一切都讲清楚。

## View-first 方式 (The View-First Appoach)

让我们从定义 View-First 的方式开始，我这么说是什么意思呢?

MVVM 声明 ViewModel 应该不知道 View 的任何信息，但是该 View 应该知道 ViewModel。联 系 View 和 ViewModel 最简单的方法就是让 View 在它的代码背后构造它的 ViewModel——就像这样:

|                              C#                              | VB.NET |
|----|----|
| public partial class MyView : Window <br>{<br>    public MyView()<br>    {<br>        InitializeComponent();<br>         this.DataContext = new MyViewModel();<br>    } <br>} |Partial Public Class MyView : Inherits Window<br><br>     Public Sub New()<br>     InitializeComponent()<br>       Me.DataContext = new MyViewModel()<br><br>   End Class|

这是很好。View 可以创建和拥有其他 View，这意味着可以将 View 组合成层次结构。一切都很好。

当你组合了几个视图，比如这样，一个 Shell 包含一个顶部栏和一个框架，里面的任何页面都可以显示:

```
<!-- This is a window which contains a top bar and another page -->
<Window x:Class="MyNamespace.ShellView" ....>
   <StackPanel>
      <my:TopBarView/>
      <Frame x:Name="navigationFrame"/>
   </StackPanel>
</Window>
```

TopBarView 有它自己的 ViewModel 。

现在假设 TopBarView 有一个字段，其中包含您想要更新的一些数据，例如当前页面的标题。现在，ShellViewModel 知道这一点(毕竟它决定了当前页面是什么)，但是 TopBarViewModel 不知道(它怎么知道?它什么都不知道)。这样做是为了暴露 TopBarView 的依赖属性，并将其绑定到 ShellViewModel 中，就像这样:

```
<Window x:Class="MyNamespace.ShellView" .... x:Name="rootObject">
   <StackPanel>
      <my:TopBarView CurrentPageTitle="{Binding CurrentPageTitle, ElementName=rootObject}"/>
      <Frame x:Name="navigationFrame"/>
   </StackPanel>
</Window>
```

但那太恶心了。现在您已经得到了一个绑定到 ShellViewMode l 的视图。

另一个主要的关注点是显示窗口和对话框。

在传统的 MVVM 中，这是一个痛点。第一种选择是实例化并从 ViewModel 中显示视图(使用 Show() 或 ShowDialog() ) ，这使得 ViewModel，或者至少是它的一部分，变得不可测试。

更好的选择是在你当前视图的后台代码中显示新视图的时候才实例化，并从那里显示它。这意味着您需要建立一种方式来通知当前视图，让其显示实例化的视图，同时，还需要建立一种方式来获取对话框的结果并返回到 ViewModel。

实际上，为上面的框架设置内容需要实例化一个视图并放进视图中。这也面临同样的困境——要么视图模型实例化它(使其不可测试)，要么视图实例化它(导致通信困难)。

不管怎样，这种方法都有一些缺点。



## [ViewModel-First 方式( The ViewModel-First Approach)]()

ViewModel-first 方式的思想是：ViewModel 不应该知道其 View 的任何信息，同时 View 也不负责构造 ViewModel。相反，第三个服务负责为给定的 ViewModel 定位正确的 View，并正确地设置它的DataContext。

默认实现使用命名约定对给定的 ViewModel 定位正确的 View，将 “ViewModel” 替换为其名称中的 “View”。在 [ViewManager]() 中有更详细的解释。

这允许其他 viewmodel 创建 viewmodel。它允许 viewmodel 了解和拥有其他 viewmodel。这允许您正确地组合您的视图模型。

这个技巧还有另外一个部分，可以用例子来解释:

| C#                                                           | VB.NET                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| public class ShellViewModel<br> {<br>    public TopBarViewModel TopBar { get; private set; }<br>    // Stuff to instantiate and assign TopBarViewModel } | Public Class ShellViewModel<br><br>     Public Property TopBar as TopBarViewModel<br>   ' Stuff to instantiate and assign TopBarViewModel<br><br> End Class |

```html
<Window x:Class="MyNamespace.ShellView"
        xmlns:s="https://github.com/canton7/Stylet" .....>
   <StackPanel>
      <ContentControl s:View.Model="{Binding TopBar}"/>
      <!-- ... -->
   </StackPanel>
</Window>
```

View.Model 关联的属性将获取它绑定到的 ViewModel(在本例中是 TopBarViewModel 的一个实例)，并定位正确的 View (TopBarView)。它将实例化一个实例，并将其设置为 ContentControl 的内容。

这样的结果是 TopBarView 可以从它的 TopBarViewModel 中获得当前页面的名称，而 TopBarViewModel 可以通过 ShellViewModel 得知这一点。问题解决了!

ContentControl 的技巧也适用于导航:

```
<Window x:Class="MyNamespace.ShellView"
        xmlns:s="https://github.com/canton7/Stylet" .....>
   <StackPanel>
      <ContentControl s:View.Model="{Binding TopBar}"/>
      <ContentControl s:View.Model="{Binding CurrentPage}"/>
   </StackPanel>
</Window>
```

通过实例化该页面的 ViewModel 的新实例，并将其分配给属性 CurrentPage, ShellViewModel 将导航到一个新的页面。请注意，ShellViewModel 不再需要知道关于视图的任何信息。它不必实例化一个视图。这是一个非常重要、有用和强大的点。

对话框和窗口的处理方式与 [WindowManager]() 非常相似。它接受一个给定的ViewModel实例，并将其视图显示为一个对话框或窗口。

## 删除后台代码！（Delete the Code-Behind! ）

有了这种方法，您实际上不需要在代码背后做任何事情。您当然可以这样做，但是很少有 [响应]()(用于处理事件)、转换器、附加属性和(最重要的)附加行为 不能解决的问题。

Stylet 允许您完全删除代码(它将为您调用 InitializeComponent)，强烈建议您这样做。删除后台代码!

> 注意:如果您使用的是VB。有时，如果您删除了后面的代码，则 XAML 名称空间将停止工作。如果是这种情况，只需用匹配的文件名重新创建，给它正确的名称空间和类名，然后将其余部分留空。例如，MyView.xaml.vb:
>
> ```vb
> Namespace Views
>     Public Class MyView
> 
>     End Class
> End Namespace
> ```