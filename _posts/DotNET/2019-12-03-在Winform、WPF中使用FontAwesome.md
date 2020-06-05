---
layout: post
category: .NET
title: 在 Winform、WPF 中使用 Font Awesome
tagline: by 明不知昔
tags: 
  - .NET
  - C#
published: true
---

## 序

今天在 github 上发现了一个可以在 Winform、WPF 中使用 Font Awesome 的项目，本项目不需要自己安装 Font Awesome 字体，用起来很方便。

<!--more-->

项目地址：[https://github.com/awesome-inc/FontAwesome.Sharp](https://github.com/awesome-inc/FontAwesome.Sharp)

## 安装

在包管理器中添加 nuget 安装包。

> Install-Package FontAwesome.Sharp 

## 功能

将 FontAwesome 图标生成图片和 Icon 图标

## Winform 上使用

1. 在 Winform 上可以使用下列类
    * IconButton,
    * IconToolStripButton,
    * IconDropDownButton,
    * IconMenuItem,
    * IconPictureBox 或者
    * IconSplitButton
2. 当然，如果你只想为 icon 生成 bitmap，可以使用`ToBitmap()/ToImageSource`的扩展。如下：

```
var bitmap = IconChar.BatteryEmpty.ToBitmap(16, Color.Black); // Windows Forms
var image = IconChar.BatteryEmpty.ToImageSource(Brushes.Black, 16); // WPF

var customFontBitmap = MyCustomFont.ToBitmap(MyEnum.SomeIcon, 16, Color.Black); // Windows Forms, custom font
var customFontImage = MyCustomFont.ToImageSource(MyEnum.SomeIcon, Brushes.Black, 16); // WPF, custom font
```

## WPF 上使用

此处因为不需要，便暂时未做翻译，请直接参考：[https://github.com/awesome-inc/FontAwesome.Sharp](https://github.com/awesome-inc/FontAwesome.Sharp)

## 致谢

1. 本文来源于:https://github.com/awesome-inc/FontAwesome.Sharp
2. 图片来源于网络