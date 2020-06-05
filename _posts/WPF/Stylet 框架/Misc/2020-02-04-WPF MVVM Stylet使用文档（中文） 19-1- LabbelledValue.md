---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）19-1 LabelledValue
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



有时您希望向用户显示一些对象，但您希望将自定义(字符串)标签与之关联，视图中将显示该标签。因此要创建易用类来包装对象并关联这个标签。

<!--more-->



然后你会想要重写 ToString 以及 Equals 和 GetHashCode，这样你的视图就只显示标签，这样它们就能在含有`SelectedItem` 的类中工作 (例如，`ComboBox`)。最后，您将希望实现 `INotifyPropertyChanged`，以便视图能够对其进行更改。

这就是`LabelledValue<T>` 的全部内容——一个具有字符串`Label`属性和 `T ` ` Value ` 属性的类。加上一个重写的 `ToString`， `GetHashCode `， `Equals`，并实现了 `INotifyPropertyChanged`。

例如:

``` C#
public enum MyEnum
{
   Foo,
   Bar,
   Baz
}

class MyViewModel
{
   // Implement INotifyPropertyChanged if you want
   public BindableCollection<LabelledValue<MyEnum>> EnumValues { get; private set; }
   public LabelledValue<MyEnum> SelectedEnumValue { get; set; }

   public MyViewModel()
   {
      this.EnumValues = new BindableCollection<LabelledValue<MyEnum>>()
      {
         LabelledValue.Create("Foo Value", MyEnum.Foo),
         LabelledValue.Create("Bar Value", MyEnum.Bar),
         LabelledValue.Create("Baz Value", MyEnum.Baz),
      };

      this.SelectedEnumValue = this.EnumValues[0];
   }
}
```

然后，在视图中：

``` XML
<ComboBox ItemsSource="{Binding EnumValues}" SelectedItem="{Binding SelectedEnumValue}"/>
```

