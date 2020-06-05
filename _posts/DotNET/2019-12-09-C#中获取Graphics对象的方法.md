---
layout: post
category: .NET
title: C# 中获取 Graphics 对象的方法
tagline: by 明不知昔
tags: 
  - .NET
  - C#
published: true
---

在做自定义控件时或者GDI+的时候经常会遇到获取Graphics实例的问题。一般有 4 种获取方式。

<!--more-->

## 正文

1. 从Paint事件的参数中获取。
窗体和许多控件都有一个Paint事件，有一个PaintEventArgs类型的参数e
``` C#
private void Form1_Paint(object sender,System.Windows.Forms.PaintEventArgs e)
        {
           //获取Graphic对象
           Graphics g = e.Graphics;
         //书写绘图代码
            g.DrawLine(new Point(0,0),new Point(1,1));
            //释放Graphic对象占用的资源
            g.Dispose();
}
```

2. 用CreateGraphics方法创建
如果需要在Paint方法以外绘图，可以通过控件或窗体的CreateGraphics方法来获取Graphics对象
``` C#
using(Graphics g=new Control().CreateGraphics())
{
}
```

3. 对Image对象调用Graphics.FromImage获取
``` C#
//创建Image对象
Bitmap image1 = new Bitmap("football.jpg");
//窗体的绘图对象
Graphics formE = e.Graphics;
```
4. 通过Graphics的FromHwnd函数
``` C#
HandleRef NullHandleRef = new HandleRef(null, IntPtr.Zero);
using (Graphics g = Graphics.FromHwnd(NullHandleRef.Handle))
{
}
```

## 致谢
本文转载于 [https://www.bbsmax.com/A/Gkz1ony25R/](https://www.bbsmax.com/A/Gkz1ony25R/)

