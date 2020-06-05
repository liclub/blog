---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）07-MessageBox
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



## 简介

我们都知道，WPF有自己的消息框实现——“System.Windows.MessageBox”。这很好，除了你不能从你的 ViewModel中调用它(好吧，你*可以*，但它使你的 ViewModel 不可测试)。网上常见的解决办法是“实现你自己的消息框”。

好吧，Stylet 自带了自己的 MessageBox 克隆，它的外观和行为几乎与 WPF 的一样(包括外观、按钮、图标、自动调整大小、声音、对齐等)。

<!--more-->



## 使用方法（Usage）

从 IWindowManager 调用 ShowMessageBox 即可，像这样：

<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
public MyViewModel
{
   private readonly IWindowManager windowManager;
&nbsp;
   public MyViewModel(IWindowManager windowManager)
   {
      this.windowManager = windowManager;
   }
&nbsp;
   public void ShowMessagebox()
   {
      var result = this.windowManager.ShowMessageBox(&quot;Hello&quot;);
   }
}</pre>
</td><td valign="top"><pre lang="vb.net">
Public Class MyViewModel
&nbsp;
    Private ReadOnly windowManager As IWindowManager
&nbsp;
    Public Sub New(ByVal windowManager As IWindowManager)
        Me.windowManager = windowManager
    End Sub
&nbsp;
    Public Sub ShowMessagebox()
        Dim result = Me.windowManager.ShowMessageBox(&quot;Hello&quot;)
    End Sub
End Class</pre></td></tr></table>

MessageBox接受与WPF消息框相同的所有选项，还有增加了一些其它选项。



## 定制消息框（Customising the MessageBox)

Stylet 的 MessageBox 实现为一个 ViewModel，`MessageBoxViewModel `，以及它相应的视图 `MessageBoxView`。ViewModel 实现了一个接口 `IMessageBoxViewModel`，而 `ShowMessageBox()` 方法使用这个接口来检索 ViewModel 的实例。

因此，您可以通过编写一个实现 `IMessageBoxViewModel` 的 ViewModel，并将其注册到 IoC 容器中，从而提供自己的 `MessageBoxViewModel` 和 `MessageBoxView` 的自定义实现。然后 `ShowMessageBox()`将使用它。

如果你只是想调整现有的 `MessageBoxViewModel` 的行为，你可以有以下选择:



### 定制按钮文本（Custom Button Text）

通过修改 MessageBoxViewModel.ButtonLabels，您可以在每个应用程序的基础上编辑任何按钮的按钮文本。它是一个字典，保存每个按钮要显示的文本。如果你只是想编辑文本的一个特定的消息框，`ShowMessageBox` 将接受一个字典，你可以这样做：

<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
MessageBoxViewModel.ButtonLabels[MessageBoxResult.No] = &quot;No, thanks&quot;;
&nbsp;
this.windowManager.ShowMessageBox(&quot;Do you want breakfast?&quot;, 
                                   buttons: MessageBoxButton.YesNo, 
                                   buttonLabels: new Dictionary&lt;MessageBoxResult, string&gt;()
        {
            { MessageBoxResult.Yes, &quot;Yes please!&quot; },
        });
&nbsp;
// Will display a MessageBox with the buttons &quot;Yes please!&quot; and &quot;No, thanks&quot;</pre>
</td><td valign="top"><pre lang="vb.net">
    MessageBoxViewModel.ButtonLabels(MessageBoxResult.No) = &quot;No, thanks&quot;
    Me.windowManager.ShowMessageBox(&quot;Do you want breakfast?&quot;,
                                    buttons:=MessageBoxButton.YesNo,
                                    buttonLabels:=New Dictionary(Of MessageBoxResult, String)() _
                                    From {{MessageBoxResult.Yes, &quot;Yes please!&quot;}})
&nbsp;
&#39; Will display a MessageBox with the buttons &quot;Yes Please!&quot; and &quot;No, thanks&quot;</pre></td></tr></table>



### 定制按钮显示枚举值（Custom Button Set）

`MessageBoxViewModel.ButtonToResults`  字典为每个 `MessageBoxButton` 指定显示哪些按钮的枚举值。想要同时显示 “OK“，“Yes” 和 “No” 按钮吗? 修改这个字典就可以了。



### 定制图标（Custom Icons）

`MessageBoxViewModel.IconMapping` 字典指定不同的 `MessageBoxImage` 的值显示对应的图标。这个字典必须包含`MessageBoxImage` 所有可能的 Key 值 (注意不同的 Key 值在这里允许有相同的 Value 值)，一个值也可以是null，在这种情况下不会显示任何图标。



### 定制音效（Custom Sounds）

`MessageBoxViewModel.SoundMapping` 是一个字典，它为每个 `MessageBoxImage` 指定应该播放哪个声音。与 `IconMapping` 一样，`MessageBoxImage` 的每个值都必须有一个键值对，null也是一个有效值(在这种情况下不播放声音)。



### 定制文本流向和文本对齐方式（Custom FlowDirection and TextAlignment）

`IWindowManager.ShowMessageBox()` 的参数允许您指定 `FlowDirection` 和 `TextAlignment` 。

如果不指定这些参数，则使用默认的 `MessageBoxViewModel.FlowDirection` 和 `MessageBoxViewModel.TextAlignment` 使用。

如果愿意，您也可以更改这些默认值。

