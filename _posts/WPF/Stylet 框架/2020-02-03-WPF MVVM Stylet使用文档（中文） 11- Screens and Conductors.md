---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）11-Screens and Conductors
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



Screen 和 Conductor 是一个简单的主题，但它需要一些思维跳跃，并且要求您在它们变得有意义之前覆盖它们的所有部分。相信我，值得你花时间阅读这篇文章——它们非常强大，值得你花时间投资。

<!--more-->



## ViewModel 生命周期（ViewModel Lifecycles）

一个好的起点是查看 ViewModel 的生命周期。

想象一个选项卡式界面——类似 Visual Studio，它有(非常简单地)一个 shell (包含菜单、工具栏等)和一个包含编辑器选项卡的选项卡控件。在 Stylet 中，每个编辑器标签都由它自己的视图模型支持。

现在，其中一个 Viewmodel 将通过实例化开始它的生命周期。接下来，它将被显示。之后，根据当前处于活动状态的选项卡，可能会显示或隐藏它，最后关闭它。就在它关闭之前，它有机会阻止关闭来提示您保存文件。

简而言之，这就是ViewModel的生命周期：它被创建，然后被激活(显示给用户)。在此之后，它可以被停用(仍然是活动的，但没有显示)，并再次激活任意次数，直到最后被关闭(在被询问是否准备关闭之后)。



## IDisposable

值得注意的是，如果 ViewModel 实现了 `IDisposable`，那么它将在父类关闭后被处理(除非父类的 `DisposeChildren `属性为 false)。



## Conductors 介绍（Introduceing Conductors）

现在，ViewModel 并不能神奇地知道它何时被显示、隐藏或关闭。它必须被告知。这就是 Conductor 的角色。

简单地说，一个 Conductor 就是一个 ViewModel，它拥有另一个 ViewModel，并且知道如何管理它的生命周期。

在我们的 Visual Studio 示例中，Conducotr 是拥有 TabControl 的 ViewModel，并显示一个 TabControl 的 ViewModel——代码编辑界面，因此 Conductor 可能是 Shell ViewModel。每当用户选择一个新的编辑器选项卡时，Conductor 将停用旧选项卡，并激活新选项卡。当用户关闭一个选项卡时，指挥将告诉该选项卡它已经关闭，然后决定下一个要显示的选项卡，并激活它。

就是这样，真的。ViewModel 有一个生命周期，它是由拥有 ViewModel 的 Conductor 实现的。

到目前为止，这是相当抽象的-让我们进入细节。



## IScreen and Screen

正如我们在上面所看到的，一个 ViewModel 的生命周期是由一个在 ViewModel 上调用方法的 Conductor 来管理的。那些方法是在一组独立的接口中定义的——如果你实现了接口，并且那个 ViewModel 被管理为一个 Conductor，那个方法就会被调用。如果需要，您可以选择想要的接口。

有一个名为 `IScreen` 的总接口，它包含了所有这些功能，还有一个名为 `Screen` 的默认实现。这表现得非常好，您可能永远都不需要实现自己的 `IScreen`—但是如果您愿意，您可以这样做。

- `IScreenState `: 用于激活，停用，并关闭视图模型。有 `Activate`、`Deactivate` 和 `Close`方法，以及跟踪Screen 状态变化的事件和属性。

- `IGuardClose`: 用来询问 ViewModel 是否可以关闭。有一个 `CanCloseAsync` 方法。

- `IViewAware`: 有时视图模型需要知道它的视图(当它被附加时，它是什么，等等)。这个接口允许通过一个 `View`  属性和一个 `AttachView` 方法来实现。

- `IHaveDisplayName `: 有一个 `DisplayName` 属性。这个名称用作窗口和使用 `WindowManager` 显示的对话框的标题，对于选项卡控件之类的东西也很有用。
- ` IChild`: 对于 ViewModel 来说，知道管理它的是什么 Conductor 是有利的(例如，请求关闭它)。如果ViewModel 实现了 `IChild`，它将被告知这一点。

请注意，无法保证调用 Activate、Deactivate 和 Close 的顺序——一个 ViewModel 可以连续激活两次，然后在不停用的情况下关闭。由 ViewModel 来注意这些事情，并做出相应的反应。Stylet's `Screen` 做这个。

`Screen` 有一些虚方法，如果你需要，你可以重写它们：

- `OnInitialActivate`: 在第一次激活 Screen 时调用，以后再也不会调用。可以设置不想在构造函数中设置的内容。
- `OnActivate`: 当 Screen 激活时调用。仅在 Screen 尚未激活时调用。

- `OnDeactivate`: 当 Screen 被停用时调用。仅在 Screen 尚未停用时调用。
- `OnClose`: 当 Screen 关闭时调用。只会被调用一次。仅在 Screen 停用时调用。

- `OnViewLoaded`: 当视图的 `Loaded `事件被触发时调用。

- `CanCloseAsync`: 当 Conductor 想知道 Screen 是否可以关闭时，就会调用这个函数。默认情况下，retuns `Task.FromResult(this.CanClose) `，但是您可以在这里添加自己的异步逻辑。

- `CanClose`:  默认情况下由 `CanCloseAsync` 调用。这只是为了方便。如果您想决定是否可以同步关闭，请覆盖 `CanClose`。如果你想异步决定，覆盖 `CanCloseAsync`。

- `RequestClose(bool? dialogResult = null)`:  当您想从您拥有的 Conductor 请求一个关闭时，您可以调用它。如果你想显示关闭结果到对话框中，可以使用 DialogResult。

Screen 源自 PropertyChangedBase ，因此很容易引发 PropertyChanged 通知。

您可能会发现所有的 Viewmodel 都是 Screen 的子类。这并不是说它们必须如此—您可以创建自己的 `IScreen`实现，或者从上面选择您想要实现的接口—但是确实 `Screen` 非常方便和强大。



## Conductors 细节（Conductors in Detail）

Conductor 有各种各样的形式，每一种都有自己的使用案例。一个 Conductor 可以拥有一个单一的视图模型(比如一次显示一个页面的导航)，或者多个视图模型。那些具有多个 Viewmodel 的视图一次可能只有一个(想想上面 Visual Studio 例子中的 TabControl )，或者全部(想想带有许多独立元素的网格)激活。Conductor 还可以添加一些行为，比如记录显示的是哪一个视图模型(对于导航很有用)。

与 `Screen` 类一样，Stylet 定义了许多 Conductor 感兴趣的接口和实现(取决于所需的 Conductor 行为的种类)，当然您也可以实现自己的接口。

主要接口是 `IConductor<T>`，它表示一个 Conductor，你将与它进行交互。它有以下方法

- `ActivateItem(T item)`: 获取给定的项并激活它。是否使先前的项失效是由 Conductor 决定的。

- `DeactivateItem(T item)`: 获取给定的项，然后停用它。是否激活另一个项目是由 Conductor 决定的。

- ` CloseItem(T item)`:  获取给定的项并关闭它。Conductor 指定是否由另外的项激活并替换该关闭的项。

拥有单个活动项的 Conductor (不管它们可能有多少个非活动项) 还实现了 `IHaveActiveItem<T>`，它都有唯一属性`ActiveItem`。

如果项目实现了 `IChild`，那么所有的内置 Conductor 都将把项的 `Parent` 属性设置为自身。所有的内置 Conductor 都额外实现了 `IChildDelegate`，它允许子进程请求关闭它(通过调用 `CloseItem`)。在默认的 `Screen` 实现中，调用 `Screen.RequestClose` 将导致 Screen 在它的 "父" 上调用 `CloseItem` (提供它的父实现 `IChildDelegate`)，这反过来导致它的 “父”(如果它存在)关闭它。



## 内置 Conductor（Built-In Conductors）

Stylet 带有一些内置的 Conductor，它们以多种直观的方式执行。

所有这些 Conductor 都来自 `Screen`，允许 Conductor 轻松地拥有其他 Conductor。这意味着你可以以任何你想要的方式组合你的 Conductor 和 Screen。

### `Conductor<T>`

这个非常简单的 Conductor 只有一个 ViewModel (类型为 `T`)，它被公开为 `ActiveItem`。`ActivateItem` 方法用一个新的 ViewModel 实例替换当前的 `ActiveItem`，并激活新项且关闭旧项。每当 `Condcutor<T> `被激活时，它就会激活它的 `ActiveItem`; 同样地，当它被停止活动或关闭时，它也会让 `ActiveItem` 停止活动和关闭。

当被问及是否可以关闭(当调用 `CanCloseAsync`时)，它会返回 `ActiveItem` 返回的任何结果，如果没有 `ActiveItem`，则返回true。

也可以直接设置 `ActiveItem`，其效果与调用 `ActivateItem` 相同。

Conductor 的视图模型看起来是这样的——一个 ContentControl 绑定到 Conductor  的 `ActiveItem` 上:

``` html
<Window x:Class="MyNamespace.ConductorViewModel"
        xmlns:s="https://github.com/canton7/Stylet" ....>
   <ContentControl s:View.Model="{Binding ActiveItem}"/>
</Window>
```



### `Conductor<T>.Collection.OneActive`

这个 Conductor 拥有许多项，但一次只能激活一个。通过这种方式，它模拟了一个选项卡控件的行为——许多选项卡可以同时存在，但一次只能显示一个。

它拥有一个名为 `Items` 的 `T` 集合，其中一个是 `ActiveItem`。调用 `ActivateItem`将添加 项到 `Items` 集合，并将激活它且将其设置为`ActiveItem`; 如果先前设置了 `ActiveItem `，则其旧值将被停用，并保留在 `Items ` 集合中。

在一个项上调用 `DeactivateItem` 或 `CloseItem` 将分别导致该项被停用和关闭。因为它不再是活动的，它不能保持为 `ActiveItem`——相反，另一个项被选择为 `ActiveItem`时，它将被激活。默认情况下，新的 `ActiveItem` 是存在于 `Items` 集合中的，位于被停用/关闭的项之前。

如果需要，可以直接操作 `Items` 集合。也可以直接设置 `ActiveItem`，其效果与调用 `ActivateItem `并添加该项相同。

使用这个 Conductor 的，带有 TabControl 的视图模型可能是这样的(查看下面的简短版本):

``` HTML
<TabControl ItemsSource="{Binding Items}" SelectedItem="{Binding ActiveItem}" DisplayMemberPath="DisplayName">
   <TabControl.ContentTemplate>
      <DataTemplate>
         <ContentControl s:View.Model="{Binding}" VerticalContentAlignment="Stretch" HorizontalContentAlignment="Stretch" IsTabStop="False"/>
      </DataTemplate>
   </TabControl.ContentTemplate>
</TabControl>
```

但是，这有点拗口，所以 Stylet 为您提供了一个样式，它可以完成相同的工作。这意味着你可以这样做:

``` XML
<TabControl Style="{StaticResource StyletConductorTabControl}"/>
```



### `Conductor<T>.Collection.AllActive`

这个 Conductor 与 `Conductor<T>.Collection.OneActive`非常相似，除了它没有 `ActiveItem`。相反，它只有一个 `Item` 集合。当一个项被激活时(使用 `ActivateItem` )，它被添加到这个集合中，当它被关闭时，它被从这个集合中删除。

调用 `DeactivateItem` 将在不从 `Items` 集合中移除的情况下就地停用该项目。

还可以直接操作 `Items` 集合。任何添加的项将被激活，任何移除的项将被关闭。

一个典型的用例可能是使用一个 ItemsControl，其中所有的项都是同时可见的。以这种方式使用 ItemsControl 的ViewModel 可能看起来是这样的(同样，查看下面的简短版本):

``` XML
<ItemsControl ItemsSource="{Binding Items}">
   <ItemsControl.ItemTemplate>
      <DataTemplate>
         <ContentControl s:View.Model="{Binding}" VerticalContentAlignment="Stretch" HorizontalContentAlignment="Stretch" IsTabStop="False"/>
      </DataTemplate>
   </ItemsControl.ItemTemplate>
</ItemsControl>
```

由于这很冗长，Stylet 提供了一个样式来设置这些属性:

``` XML
<ItemsControl Style="{StaticResource StyletConductorItemsControl}"/>
```



### `Conductor<T>.StackNavigation`

这个 Conductor 是 `Conductor <T> `和 `Conductor<T>.Collection.OneActive` 的混合体。它提供了一些额外的功能:基于堆栈的导航。

它有一个单独的 `ActiveItem`，但也保留了一个(私有的)过去活动项目的历史记录。当您激活一个新项目时，先前的 `ActiveItem` 将被停用，并推入历史堆栈。调用 `GoBack() ` 将关闭当前 `ActiveItem`，并重新激活这个历史堆栈中的顶部项目，并将其设置为新的 `ActiveItem `。

如果您在当前的 `ActiveItem` 上调用 `CloseItem`，则会产生相同的效果。如果您在历史堆栈中存在的任何项目上调用 `CloseItem`，该项目将被关闭并从历史堆栈中删除。调用 `Clear()` 将关闭并从历史堆栈中删除所有项。

### `WindowConductor`

这个有点奇怪，因为它是内部的，你不需要直接与它交互，但我把它包含在这里是为了引起兴趣。当你使用 `WindowManager` 显示一个对话框或窗口时(这包括当你第一次启动你的应用程序时 Stylet 显示的窗口)，一个新的 `WindowConductor` 管理它的生命周期。当你的窗口或对话框最小化时，它就会失效。只要最大化，它就会被激活。如果您的 ViewModel 请求关闭它(参见上面的 `RequestClose`)，则 `WindowConductor` 将处理此操作。类似地，如果用户自己关闭窗口，`WindowConductor`将询问 ViewModel 是否准备关闭。