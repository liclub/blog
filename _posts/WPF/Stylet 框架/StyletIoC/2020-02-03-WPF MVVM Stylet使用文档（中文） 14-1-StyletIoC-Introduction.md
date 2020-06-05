---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）14-1-Introduction of StyletIoC
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



StyletIoC 是一个非常轻量和非常快的 IoC 容器。它被设计来只做很少的事情，但做得很好，且直观。

<!--more-->

它使用一个流畅的接口进行配置——没有任何XML垃圾，也没有依赖关系。

现在，我假设您对 IoC 容器的概念有一定的了解——如果没有，请阅读一些资料，然后再回来。我以后可能会写一篇更深入的介绍。



## 服务和实现（Services and Implementations）

StyletIoC 是围绕服务的概念构建的。服务是具体类型、抽象类型或接口，由具体类型实现(或可以实现)，例如:

``` C#
interface IVehicle { ... }
class HotHatchback : IVehicle { ... }
```

这里，`IVehicle` 是服务，`HotHatchback` 是实现服务的具体类型。注意 `HotHatchback` 也是一个服务，它是由 `HotHatchback` 类本身实现的。

配置 StyletIoC 时，要定义一组关系。每个关系都是服务与实现它的类型(或多个类型)之间的关系。所以这里，我们可以告诉 StyletIoC: "在服务 `IVehicle` 和类型 `HotHatchback` 之间创建一个关系”。

稍后，当你想要一个 `IVehicle` 的实现时，你可以请求 StyletIoC: “给我一个实现服务 `IVehicle` 的实例"，然后StyletIoC 会构造一个 `HotHatchback` 并将它传递给你。



## 处理类型——服务定位符和注入（Resolving Types - The Service Locator and Injection）

有3种方法让 StyletIoC 为我们构建一个类型:

1. 通过直接调用 IContainer.Get

2. 构造函数注入

3. 属性注入

直接调用 `IContainer.Get ` 是最容易解释的，它看起来像这样:

``` csharp
var ioc = ... // Covered in lots of detail elsewhere
var vehicle = ioc.Get<IVehicle>();
```

尽管这看起来很诱人，但这只应该在应用程序的根目录中完成——在其他地方使用构造函数注入和参数注入。

当 StyletIoC 为您构造一个类型时，它将寻找一个构造函数，该构造函数具有它知道如何解析的类型的参数。然后，它将解析这些类型，并将它们注入构造函数。例如:

``` csharp
class Engine { ... }
class Car
{
   public Car(Engine engine)
   {
      // 'engine' contains a new instance of Engine
   }
}
```

这是 StyletIoC 之外的创建新实例最常见的方式——StyletIoC 构造的每一个类型都会在它的构造函数中列出它的依赖项，而 StyletIoC 会构造每一个，注入它的依赖项。

如果你愿意，你也可以做参数注入，如果要注入的参数有属性 `[Inject]`，例如:

``` csharp
class Engine { ... }
class Car
{
   [Inject]
   public Engine Engine { get; set; }
}
```

每种方法的各种优点，我们将在其他地方讨论。

