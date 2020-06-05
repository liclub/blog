---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）19-3 BoolToVisibilityConverter
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



在几乎每个项目中，我都需要根据 ViewModel 中的一些 bool 值隐藏/显示一个元素。您可以使用 DataTriggers 或使用转换器来实现这一点。

<!--more-->

转换器实现非常简单: 当绑定到一个 bool 属性时，如果它读取一个 true 值，它将返回一个(预配置的)可见性，如果它读取一个 false 值，它将返回另一个。

如果你绑定到一个类型的属性而不是 bool，它将使用以下规则:

1. 如果该值为null，则视为false

2. 如果值为0 (比如 int、float、double等)，则将其视为false

3. 如果值是空集合、字典等，则将其视为 false

4. 否则，它就被当作是真的

这与许多语言中的 “真实/虚假” 规则相匹配。比如说，当且仅当绑定到的集合不是空的时候，才显示一个 `ListView`，这个很方便。

基本示例用法:

``` XML
<!-- In any .Resources section - doesn't have to be Window.Resources -->
<Window.Resources>
   <s:BoolToVisibilityConverter x:Key="boolToVisConverter" TrueVisibility="Visible" FalseVisibility="Hidden"/>
</Window.Resources>

<!-- Later in code -->
<TextBlock Visibility="{Binding SomeBoolProperty, Converter={StaticResource boolToVisConverter}}"/>
```

如果你想要(通常的)的转换器是这种情况时：true 的情况是`Visiblity.Visible` ，false  的情况是 `Visibility.Collapsed` 。这是简化方式：

``` XML
<TextBlock Visibility="{Binding SomeBoolProperty, Converter={x:Static s:BoolToVisibilityConverter.Instance}}"/>
```

同样地，如果你想要(稍微不太常见的)是：true 时是 `Visibility.Collapsed`，而 false 是 `Visibility.Visible` 。这也有一个简化方式：

``` XML
<TextBlock Visibility="{Binding SomeBoolProperty, Converter={x:Static s:BoolToVisibilityConverter.InverseInstance}}"/>
```

