---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）05-Actions
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



你有一个按钮，你想点击它并在 ViewModel 上执行一个方法?  Actiions 完全可以解决这种问题。

<!--more-->

## 响应和方法（Actions and Methods）

在“传统的” WPF 中，你需要在 ViewModel 上创建一个属性来实现 ICommand 接口，并将按钮的命令属性绑定到它。这工作得相当好 (ViewModel 对 View 一无所知，也不需要代码隐藏)，但它有点混乱——您确实需要在ViewModel 上调用一个方法，而不是在某个属性上执行一个方法。

Stylet 通过引入响应（Action） 解决了这个问题。看看这个:



<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
class ViewModel
{
   public void DoSomething()
   {
      Debug.WriteLine(&quot;DoSomething called&quot;);
   }
}</pre>
</td><td valign="top"><pre lang="vb.net">
Class ViewModel
&nbsp;
  Public Sub DoSomething()
&nbsp;
  Console.WriteLine(&quot;DoSomething called&quot;)
&nbsp;
  End Sub
End Class</pre></td></tr></table>

``` html
<UserControl x:Class="MyNamespace.View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet">
   <Button Command="{s:Action DoSomething}">Click me</Button>
</UserControl>
```

您可能已经猜到了，单击按钮的时候，会调用 ViewModel 上的 DoSomething 方法。

就这么简单。

如果您的方法接受单个参数，可以传递按钮的 CommandParameter 属性的值。例如:

<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
class ViewModel
{
   public void DoSomething(string argument)
   {
      Debug.WriteLine(String.Format(&quot;Argument is {0}&quot;, argument));
   }
}</pre>
</td><td valign="top"><pre lang="vb.net">
Class ViewModel
&nbsp;
  Public Sub DoSomething(argument As String)
&nbsp;
  Debug.WriteLine(String.Format(&quot;Argument is {0}&quot;, argument)
&nbsp;
  End Sub
End Class</pre></td></tr></table>

``` html
<UserControl x:Class="MyNamespace.View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet">
   <Button Command="{s:Action DoSomething}" CommandParameter="Hello">Click me</Button>
</UserControl>
```

> 注意：Actions 对所有的 ICommand 属性都起作用 (比如 [KeyBinding](http://msdn.microsoft.com/en-us/library/system.windows.input.keybinding%28v=vs.110%29.aspx))



## 保护属性（Guard Properties）

您还可以使用 *Guard Properties* 来控制是否启用按钮。给定方法的保护属性是一个布尔属性，其名称为 “Can\<方法名\>”，因此如果您的方法名为 “DoSomething”，则相应的保护属性为 “CanDoSomething”。

Stylet 将检查保护属性是否存在，如果存在，如果返回 false 禁用按钮，如果返回 true 则启用按钮。它还将监视该属性的 PropertyChanged 通知，因此您可以控制按钮是否启用。 

例如:

<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
class ViewModel
{
   public bool CanDoSomething
   {
      get { return this.someOtherConditionIsSatisfied(); }
   }
   public void DoSomething()
   {
      Debug.WriteLine(&quot;DoSomething called&quot;);
   }</pre>
</td><td valign="top"><pre lang="vb.net">
Class ViewModel
&nbsp;
  Public ReadOnly Property CanDoSomething As Boolean
        Get
            Return Me.someOtherConditionIsSatisfied()
        End Get
    End Property
&nbsp;
    Public Sub DoSomething()
        Debug.WriteLine(&quot;DoSomething called&quot;)
    End Sub
End Class</pre></td></tr></table>



## 事件（Events）

但是如果您想在事件发生时调用 ViewModel 方法呢? Actions 也包括这些。语法是完全相同的，但是没有保护属性的概念。

``` html
<UserControl x:Class="MyNamespace.View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet">
   <Button Click="{s:Action DoSomething}">Click me</Button>
</UserControl>
```

被调用的方法必须有 0个 或者 1个 或者 2个 参数。可能的情况如下：

<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
public void HasNoArguments() { }
&nbsp;
// This can accept EventArgs, or a subclass of EventArgs
public void HasOneSingleArgument(EventArgs e) { }
&nbsp;
// Again, a subclass of EventArgs is OK
public void HasTwoArguments(object sender, EventArgs e) { }</pre>
</td><td valign="top"><pre lang="vb.net">
Public Sub HasNoArguments()
End Sub
&nbsp;
&#39; This can accept EventArgs, or a subclass of EventArgs
Public Sub HasOneSingleArgument(ByVal e As EventArgs)
End Sub
&nbsp;
&#39;Again, a subclass of EventArgs is OK
Public Sub HasTwoArguments(ByVal sender As Object, ByVal e As EventArgs)
End Sub</pre></td></tr></table>



## 响应目标（The View.ActionTarget）

到目前为止，我一直在说一个善意的谎言。我一直在说 Actions 是在 ViewModel 上调用的，但这并不完全正确。让我们更详细一点。

Stylet 定义了一个可继承的附加属性 View.ActionTarget。当一个视图被绑定到它的视图模型，View 中根元素上的ActionTarget 绑定到 ViewModel，然后由视图中的每个元素继承。当您调用一个 响应 时，它将在View.ActionTarget中被调用。

这意味着，在默认情况下，不管当前的 DataContext 是什么，都会在 ViewModel 上调用操作，这可能就是您想要的。

这是非常重要的一点，值得强调。DataContext 可能会在整个可视树中的多个位置发生变化。但是, View.ActionTarget 将保持不变(除非您手动更改它)。这意味着 Actions 总是由您的 ViewModel 来处理，而不是由绑定到的对象来处理，这几乎总是您想要的。

当然，您可以更改单个元素的 View.ActionTarget，例如:

<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
class InnerViewModel
{
   public void DoSomething() { }
}
class ViewModel
{
   public InnerViewModel InnerViewModel { get; private set; }
   public ViewModel()
   {
      this.InnerViewModel = new InnerViewModel();
   }
}</pre>
</td><td valign="top"><pre lang="vb.net">
Class InnerViewModel
&nbsp;
    Public Sub DoSomething()
    End Sub
&nbsp;
End Class
&nbsp;
Class ViewModel
&nbsp;
    Public Property InnerVM As InnerViewModel
&nbsp;
    Public Sub New()
        Me.InnerVM = New InnerViewModel()
    End Sub
&nbsp;
End Class</pre></td></tr></table>

``` html
<UserControl x:Class="MyNamespace.View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet">
   <Button s:View.ActionTarget="{Binding InnerViewModel}" Command="{s:Action DoSomething}">Click me</Button>
</UserControl>
```

在这里, 单击按钮时将调用 InnerViewModel.DoSomething。还有，因为 View.ActionTarget 是继承的，按钮的任何子元素也会将它们的 View.ActionTarget 设置为 InnewViewModel。



## 响应和样式（Actions and Styles）

从样式设置器执行的 Actions 将不起作用。WPF 中需要的类都是内部的，这意味着没有办法修复这个问题。不幸的是，在这种(罕见的)情况下，您需要使用老式的命令。



## 快捷方式——快捷菜单和弹出菜单（Gotchas - ContextMenu and Popup）

当然，View.ActionTarget 是一个附加属性，它被配置为由设置它的任何元素的子元素继承，就像任何附加的属性一样，甚至是 DataContext，有一些边界是不能被继承的，比如:

- 使用快捷菜单

- 使用弹出窗口

- 使用框架

在这些情况下，Stylet 将尽其所能找到一个合适的 ActionTarget (例如，它可能会找到与当前 XAML 文件中的根元素相关联的 ActionTarget )，但这可能不是您所期望的(例如，它可能会忽略你的页面中间的 `s:View.ActionTarget = "{Binding…}` 行)，或者它可能(在罕见的情况下)根本找不到一个ActionTarget。

在这种情况下，请设置`s:View.ActionTarget` 一个合适的值。您可能很难从 ContextMenu 的内部获得对ContextMenu 之外任何内容的引用: 我建议使用[BindingProxy](http://www.thomaslevesque.com/2011/03/21/wp- how- binding -to-data-when-the-datacontext-is-not-inherited/)技术。



## 其它行为（Additional Behaviour）

有两种情况会阻止操作正常工作:

- 如果 View.ActionTarget 为空
- 或者如果指定的方法在 View.ActionTarget 根本不存在。

这些情况下的默认行为如下:

|          | View.ActionTarget == null | No method on View.ActionTarget                 |
| -------- | ------------------------- | ---------------------------------------------- |
| Commands | Disable the control       | Throw an exception when the control is clicked |
| Events   | Enable the control        | Throw an exception when the event is raised    |

这样做的理由是如果 View.ActionTarget 为空，你必须自己设置它，所以你可能知道你在做什么。但是，如果指定的方法在 View.ActionTarget 中不存在，这可能是个错误，你应该知道。

当然，这种行为是可配置的。

当在 View.ActionTarget 为null时，要控制它的行为，可以在 `Action` 标记扩展上设置 `NullTarget` 属性，使它的值为 `Enable`、` Disable` 或 `Throw`都可以。(请注意，当Action 在事件中使用时，“Disable”是无效的时候，因为我们没有权力禁用任何东西)。

例如:

``` html
<Button Command="{s:Action MyMethod, NullTarget=Enable}"/>
<Button Click="{s:Action MyMethod, NullTarget=Throw}"/>
```

更熟悉的方式，你也可以设置 `ActionNotFound` 的值为上面三个值之一

``` html
<Button Command="{s:Action MyMethod, ActionNotFound=Disable}"/>
<Button Click="{s:Action MyMethod, ActionNotFound=Enable}"/>
```

