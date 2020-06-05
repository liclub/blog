---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）16-Listening to INotifyPropertyChanged
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



可能容易实现 INotifyPropertyChanged，但通常是有点痛苦，想象一个对象需要`propertychange` 通知：你需要注册一个事件处理程序,检查属性名是否是你所期望的,完成后又需要注销事件处理程序。

这是一个很常见的问题，Stylet提供了一些方法来简化工作。

<!--more-->



## INotifyPropertyChanged.Bind

这是订阅 PropertyChanged 事件的最简单方法，它使用对订阅服务器的强引用(与普通事件一样)来实现这一点。这意味着如果你打算在订阅的东西还存在的时候发布，你必须记得取消订阅。

用法很简单。假设有一个对象的形式:

```csharp
// Can be any implementation of INotifyPropertyChanged - I'm using PropertyChangedBase as it makes the example shorter
class Model : PropertyChangedBase
{
   private string _stringProperty;
   public string StringProperty
   {
      get { return this._stringProperty; }
      set { SetAndNotify(ref this._stringProperty, value); }
   }
}
```

您希望每次 `StringProperty` 属性更改时都得到通知。你这样做:

```csharp
var model = new Model();
// ... 
model.Bind(x => x.StringProperty, (sender, eventArgs) => Debug.WriteLine(String.Format("New value for property {0} on {1} is {2}", eventArgs.PropertyName, sender, eventArgs.NewValue)));
```

`x => x.StringProperty` 是指定你想观察那个属性，这种方式是类型安全的。`x` 可以任意命名，当你打出 `x=>x.` 时，智能感知会提示你一个属性列表。

`(propertyName, newValue) => Debug.WriteLine(String.Format("New value is {0}", newValue))`在每次属性改变时被调用。



`Bind` 方法实际上返回一个 `IEventBinding` 的实现，它只有一个 `Unbind`方法。要删除绑定，请调用该方法。例如:

```csharp
var model = new Model();
// ... 
var binding = model.Bind(x => x.StringProperty, (sender, eventArgs) => Debug.WriteLine(String.Format("New value for property {0} on {1} is {2}", eventArgs.PropertyName, sender, eventArgs.NewValue)));
// ...
binding.Unbind();
```



## INotifyPropertyChanged.BindWeak

通常，当您订阅一个事件时，接收事件通知的对象至少与发布事件的对象存活的时间一样长，因为发布事件的对象最终会引用接收事件通知的对象。

这通常是不受欢迎的。例如，如果您有一个 ViewModel，它想要监视它所依赖的某些服务上的 PropertyChanged事件。

Stylet 在 INotifyPropertyChanged 上提供了一个名为 `BindWeak` 的扩展方法，它与 `Bind` 非常相似，只是它创建了一个弱绑定。语法与 `Bind` 相同，所以这里我就不重复了。

> 请注意，以弱方式绑定每个委托是不可能的。捕获局部变量的委托通常会失败。下面将对此进行更详细的讨论。



## 技术:弱事件订阅（Technical: Weak Event Subscriptions）

我将忽略委托的一些细节，但在基本术语中，当您订阅一个事件时，您创建一个新的委托实例，并将其传递给拥有该事件的对象。委托(基本上)由两部分组成: 要调用的方法(`method`)属性和要调用该方法的实例(`Target`属性)。

如果你创建一个委托，它指向你的类的一个实例方法，一切都很好，很简单:

```csharp
class MyClass
{
   public MyClass(Model model)
   {
      model.PropertyChanged += new PropertyChangedEventHandler(this.PropertyChangedHandler);
      // or, more concisely
      model.PropertyChanged += this.PropertyChangedHandler;
   {

   public void PropertyChangedHandler(object sender, PropertyChangedEventArgs e)
   {
      // ...
   }
}
```

在这种情况下，一个新的委托被创建，它的 `Target` 被设置为 `MyClass` 实例，它的 `method` 被设置为 `MethodInfo`，代表你的 `PropertyChangedHandler` 方法。

`model` 实例然后成为该委托的所有者。这意味着 `model` 实例拥有一个拥有 `MyClass` 实例的委托，这意味着 `MyClass` 实例不能被释放，直到 `model` 实例被释放。

当匿名委托/ lambdas开始发挥作用时，事情开始变得有点复杂，例如:

```csharp
class MyClass
{
   public MyClass(Model model)
   {
      model.PropertyChanged += delegate { Debug.WriteLine("Hi"); };
      // Or, using lambas (preferred)
      model.PropertyChanged += (o, e) => Debug.WriteLine("Hi");
   }
}
```

在这里，c# 编译器必须在你的类上创建一个新的，特殊的方法，它看起来像这样:

```csharp
class MyClass
{
   public MyClass(Model model)
   {
      model.PropertyChanged += new PropertyChangedEventHandler(this.<.ctor>b__0);
   }

   [CompilerGenerated]
   private void <.ctor>b__0(object sender, PropertyChangedEventArgs e)
   {
      Debug.WriteLine("Hi");
   }
}
```

注意上面那个特殊方法名的使用——其中包含字符( “<” 和 “>”)，这些字符在 c# 中无效，但在 CLR 中有效)。

如果我们有一个捕获局部变量的匿名委托/lambda，这将变得更加复杂。在这里，c# 编译器需要生成一个全新的嵌入式类，它保留了对该变量的引用。例如:

```csharp
class MyClass
{
   public MyClass(Model model)
   {
      string test = "test";
      model.PropertyChanged += (o, e) => Debug.WriteLine(test);
   }
}
```

编译成看起来有点像:

```csharp
class MyClass
{
   public MyClass(Model model)
   {
      MyClass.<>c__DisplayClass1 c__DisplayClass1 = new MyClass.<>c__DisplayClass1();
      c__DisplayClass1.test = "test";
      model.PropertyChanged += new PropertyChangedEventHandler(c__DisplayClass1.<.ctor>b__0);
   }

   [CompilerGenerated]
   private sealed class <>c__DisplayClass1
   {
      public string test;
      public void <.ctor>b__0(object sender, PropertyChangedEventArgs e)
      {
         Debug.WriteLine(this.test);
      }
   }
```

这里，创建的 PropertyChangedEventHandler 委托将 `<>c__DisplayClass` 的实例作为它的 `Target` 属性的值。

这意味着，当`MyClass` 的构造函数返回时，`<>c__DisplayClass` 实例引用的*唯一*东西就是委托。`<>c__DisplayClass` 实例的生命周期现在完全独立于 `MyClass` 实例。

实现弱事件的方法是让委托的 “target” 属性以某种方式成为 “WeakReference” ——通常是让它指向一个中间类，而这个中间类又有一个指向“真正”目标的 “WeakReference”。这意味着委托不会保留目标。

如果这个目标是一个编译器生成的内部类，那么除了我们创建的 `WeakReference` 之外，没有其他任何东西会保存对它的引用。这意味着这个内部类将被直接收集，因此委托将永远不会被调用。

因此，如果委托给它有一个`target`，且有 `CompilerGenerated ` 属性，`BindWeak` 将抛出一个异常。

