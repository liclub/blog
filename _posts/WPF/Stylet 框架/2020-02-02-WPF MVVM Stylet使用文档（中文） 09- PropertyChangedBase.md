---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）09-PropertyChangedBase
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



## 简介

PropertyChangedBase 是实现 INotifyPropertyChanged 类型的基类，它提供了触发 PropertyChanged 通知的方法。

<!--more-->



## 触发通知（Raising Notifications）

有许多方法可以触发 PropertyChanged 通知，具体取决于您想要做什么。

最常见的情况是，每次将值分配给一个属性时，它都会引发一个通知。PropertyChangedBase 提供了一个很好的实用程序方法来帮助: SetAndNotify，它的参数需为要分配值的字段的引用和分配给该字段的值。如果字段的值不等于原来的值，则会赋值，并引发 PropertyChanged 通知。如下：

```c#
class MyClass : PropertyChangedBase
{
   private string _name;
   public string Name
   {
      get { return this._name; }
      set { SetAndNotify(ref this._name, value); }
   }
```

如果您想为当前属性以外的属性发出 PropertyChanged 通知，也有几种方法可以实现(取决于您是使用c# 6还是更低)。

首选的 C# 6方法是使用 `nameof() `，因为它开销特别小，并且提供了编译时安全性。

如果您使用的是 C# 5或更低的版本，您可以使用表达式: 这比较慢，但是也提供了编译时的安全性。

如果你真的想要，你也可以使用原始字符串。

见下文:

```c#
class MyClass : PropertyChangedBase
{
   public string Name { get; private set; }

   public void RaiseNameChangedExpression()
   {
      // Preferred if you're using C# 6, as it provides compile-time safety
      this.NotifyOfPropertyChange(nameof(this.Name));

      // Preferred for C# 5 and below, as it provides compile-time safety
      this.NotifyOfPropertyChange(() => this.Name);

      // Not preferred, but is very occasionally necessary
      this.NotifyOfPropertyChange("Name");
   }
}
```



## 调试事件（Dispatching Events）

默认情况下，PropertyChanged 事件在当前线程上引发 (WPF负责将它们分派到UI线程)。但是，如果您确实想改变这一点，您可以! PropertyChangedBase 有一个名为 PropertyChangedDispatcher 的属性，它有一个 `Action<Action>` 并且默认执行。DefaultPropertyChangedDispatcher(您可以分配它)(它的值为 `a => a()`)。

如果希望将其更改为在UI线程上执行，可以执行以下操作。

改变 PropertyChangedBase 的所有实例:

``` C#
class Bootstrapper : Bootstrapper<MyRootViewModel>
{
   public override void Configure()
   {
      base.Configure();
      Execute.DefaultPropertyChangedDispatcher = Execute.OnUIThread;
   }
}
```

改变 PropertyChangedBase 的一个实例:

``` C#
class MyClass : PropertyChangedBase
{
   public MyClass()
   {
      this.PropertyChangedDispatcher = Execute.OnUIThread;
   }
}
```



## 使用 PropertyChanged.Fody (Use with PropertyChanged.Fody)

[PropertyChanged.Fody](https://github.com/Fody/PropertyChanged) 是一个非常棒的包，它在编译时注入代码，为您的属性自动触发 PropertyChanged 通知，允许您编写非常简洁的代码。它还会找出你的属性之间的依赖关系，并提出适当的通知，例如:

``` C#
class MyClass : PropertyChangedBase
{
   public string FirstName { get; private set; }
   public string LastName { get; private set; }
   public string FullName { get { return String.Format("{0} {1}", this.FirstName, this.LastName); } }

   public void SomeMethod()
   {
      // PropertyChanged notifications are automatically raised for both FirstName and FullName
      this.FirstName = "Fred";
   }
```

PropertyChangedBase 还负责与 `Fody.PropertyChanged` 集成。`Fody.PropertyChanged` 发出的通知使用 `PropertyChangedDispatcher` 触发。

因此，在任何`Screen`、`ValidatingModelBase `或者 `PropertyChangedBase` 的子类中，你不需要做任何特殊的事情来使用 `Fody.PropertyChanged` 。



## 名词解释

1. 属性依赖：一个属性需要依靠其它属性的值才能得出结果。