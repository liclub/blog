---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）19-2 Debug Converter
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



在我曾经需要调试绑定的每个项目上，最简单的方法是在绑定上放置一个转换器，它只记录它看到的值。`DebugConverter 就是这样一个转换器的实现，它将记录对 Visual Studio 输出窗口的每次调用，前提是你正在运行一个调试版本。

<!--more-->



基本用法很简单:

``` XML
<TextBox Text="{Binding MyProperty, Converter={x:Static s:DebugConverter.Instance}}"/>
```

如果您希望同时激活多个实例，并希望为每个实例指定一个名称(包含在其输出中)，您可以这样做:

``` XML
<!-- In any .Resources section - doesn't have to be Window.Resources -->
<Window.Resources>
   <s:DebugConverter x:key="debugConverter" Name="MySpecialName"/>
</Window.Resources>
<!-- Later in code -->
<TextBlock Text="{Binding MyProperty, Converter={StaticResource debugConverter}}"/>
```

