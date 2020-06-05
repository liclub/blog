---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）06-The WindowManager
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



在传统的 "视图优先" 的方式中，如果您想要显示一个新窗口或对话框，您需要创建一个视图的新实例，然后调用 `. show()` 或 `.showdialog() `。

<!--more-->

在 ViewModel-first 方式中，您不能直接与视图交互，所以您不能这样做。WindowManager 解决了这个问题——调用 `IWindowManager.ShowWindow(someViewModel)` 将获取那个 ViewModel，找到它的视图，实例化它，将它绑定到那个ViewModel，并显示它。

<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
class SomeViewModel
{
   private IWindowManager windowManager;
   public SomeViewModel(IWindowManager windowManager)
   {
      this.windowManager = windowManager;
   }
&nbsp;
   public void ShowAWindow()
   {
      var viewModel = new OtherViewModel();
      this.windowManager.ShowWindow(viewModel);
   }
&nbsp;
   public void ShowADialog()
   {
      var viewModel = new OtherViewModel();
      bool? result = this.windowManager.ShowDialog(viewModel);
      // result holds the return value of Window.ShowDialog()
      if (result.GetValueOrDefault(true))
      {
         // DialogResult was set to true
      }
   }
}</pre>
</td><td valign="top"><pre lang="vb.net">
Class SomeViewModel
&nbsp;
    Private windowManager As IWindowManager
&nbsp;
    Public Sub New(ByVal windowManager As IWindowManager)
        Me.windowManager = windowManager
    End Sub
&nbsp;
    Public Sub ShowAWindow()
        Dim viewModel = New OtherViewModel()
        Me.windowManager.ShowWindow(viewModel)
    End Sub
&nbsp;
    Public Sub ShowADialog()
        Dim viewModel = New OtherViewModel()
        Dim result As Boolean? = Me.windowManager.ShowDialog(viewModel)
        &#39; Result holds the return value of Window.ShowDialog()
        If result.GetValueOrDefault(True) Then
        &#39; DialogResult was set to true
        End If
    End Sub
End Class</pre></td></tr></table>

很简单! 此外，IWindowManager 的引入(而不是直接在 ViewModel 上调用方法)使测试变得更加容易。

要从视图模型中关闭窗口或对话框，请使用 `Screen.RequestClose`, 像这样:

<table><tr><td>C#</td><td>VB.NET</td>
<tr><td valign="top"><pre lang="csharp">
class ViewModelDisplayedAsWindow
{
   // Called by pressing the &#39;close&#39; button
   public void Close()
   {
      this.RequestClose();
   }
}
&nbsp;
class ViewModelDisplayedAsDialog
{
   // Called by pressing the &#39;OK&#39; button
   public void CloseWithSuccess()
   {
      this.RequestClose(true);
   }
}</pre>
</td><td valign="top"><pre lang="vb.net">
Class ViewModelDisplayedAsWindow
&nbsp;
    &#39; Called by pressing the  &#39;close&#39; button
    Public Sub Close()
        Me.RequestClose()
    End Sub
&nbsp;
  End Class
&nbsp;
Class ViewModelDisplayedAsDialog
&nbsp;
    &#39; Called by pressing the &#39;OK&#39; button
    Public Sub CloseWithSuccess()
        Me.RequestClose(True)
    End Sub
End Class</pre></td></tr></table>

