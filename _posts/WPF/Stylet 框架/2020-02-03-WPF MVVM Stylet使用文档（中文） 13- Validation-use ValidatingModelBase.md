---
layout: post
category: MVVM
title: WPF MVVM Stylet使用文档（中文）13-Validation Use ValidatingModelBase
tagline: by 明不知昔
tags: 
  - WPF
  - MVVM
published: true
---



## 简介（Introduction）

想象这个场景……用户正在填写您精心编写的表单，他们在应该输入电子邮件地址的地方输入了自己的名字。您需要检测这个问题，并以清晰的方式显示问题。

<!--more-->



输入验证是一个很大的领域，有很多方法可以实现它。最简单和最吸引人的是在你的属性的 setter 中抛出一个异常，像这样:

``` C#
private string _name;
public string Name
{
   get { return this._name; }
   set
   {
      if (someConditionIsFalse)
         throw new ValidationException("Message");
      this._name = value;
   }

```

当绑定设置此属性时，它会通知是否抛出异常，并相应地更新控件的验证状态。

然而，这最终会变成一个**彻底的坏主意**。这意味着您的属性只有在设置时才能被验证(例如，当用户单击 “Submit” 时，您不能遍历并验证整个表单)，这会导致大量重复逻辑的属性设置器。好可怕。

c#还定义了两个接口，WPF知道这两个接口: [IDataErrorInfo ](http://msdn.microsoft.com/en-gb/library/system.componentmodel.idataerrorinfo.aspx)和 [INotifyDataErrorInfo](http://msdn.microsoft.com/en-us/library/vstudio/system.componentmodel.inotifydataerrorinfo)。这两种方法都为ViewModel 提供了一种方法，通过事件和 PropertyChanged 通知来告诉视图，输入中一个或多个属性有一个或多个验证错误。其中，INotifyDataErrorInfo 更新、更容易使用，并且允许异步验证。

但是，使用 INotifyDataErrorInfo 仍然有点不直观: 它允许您广播一个或多个属性有错误的问题，但是没有为您提供运行验证的简单方法，并且要求您记录哪些错误与哪些属性相关联。

ValidatingModelBase 的目标是解决这些问题，并提供一种直观、简单的方法来运行和报告验证。



## ValidatingModelBase

ValidatingModelBase 派生自`PropertyChangedBase`，并由 Screen 继承。它建立在 PropertyChangeBase 的功能上，当属性发生变化时，它可以通知并报告验证。



### IModelValidator

有许多方法可以运行验证，有许多好的库可以帮助您。Stylet 并不打算提供另一个验证库，因此 Stylet 允许您提供自己的验证库，以便 ValidatingModelBase 使用。

这体现在 ValidatingModelBase 的 `validator` 属性中，它是一个 `IModelValidator`。这样做的目的是编写自己的 `IModelValidator` 实现，它封装了首选的验证库(稍后我将介绍一些如何实现的示例)，以便ValidatingModelBase 可以使用它。

这个接口有两个重要的方法:

``` csharp
Task<IEnumerable<string>> ValidatePropertyAsync(string propertyName);
Task<Dictionary<string, IEnumerable<string>>> ValidateAllPropertiesAsync();
```

当需要通过名称验证单个属性时，ValidatingModelBase 将调用第一个方法，并返回一个验证错误的数组。第二个是 ValidatingModelBase 在您要求它执行完整验证时调用的，并返回一个'`property name => array of validation errors`。

这些方法是异步的，这允许您利用 INotifyDataErrorInfo 的异步验证功能，并可以在某些外部服务上运行验证。但是，这个接口的大多数实现都只返回一个完成的任务。

还有第三个方法:

``` C#
void Initialize(object subject);
```

当 ValidatingModelBase 第一次设置它的验证时，它会调用这个函数，并传递它自己的一个实例。这使得 `IModelValidator` 的实现可以专门用于验证 ValidatingModelBase 的特定实例。当我们将东西绑定到 StyletIoC中时，这更有意义。

还有这个接口的一个通用版本 `IModelValidator<T>`，它只是扩展了 `IModelValidator`，没有添加任何额外的内容。当 IoC 容器进入画面时，这也是很有用的—稍后会详细介绍。



### 运行验证（Running Validations）

首先，您必须记住将 `IModelValidator` 实现传递给 `ValidatingModelBase`。你可以通过设置 `validator` 属性，或者调用一个合适的构造函数来实现:

``` C#
public class MyViewModel : ValidatingModelBase
{
   public MyViewModel(IModelValidator validator) : base(validator)
   {
   }
}
```

默认情况下，每当一个属性发生变化时，ValidatingModelBase 都会运行该属性的验证(前提是您调用 `SetAndNotify`。使用 `NotifyOfPropertyChange` 或使用 `PropertyChanged.Fody` 来触发  PropertyChangedBase 中定义的机制来引发PropertyChanged通知）。然后，它将使用 `INotifyDataErrorInfo` 接口中定义的机制报告该属性的验证状态的任何更改。它还将更改 `HasErrors`属性的值。

如果您想要禁用这种自动验证行为，请将 `AutoValidate` 属性设置为 `false` 。

如果需要，可以通过调用 `ValidateProperty("PropertyName")`, 或 `ValidateProperty(() => this.PropertyName)` 来手动运行单个属性的验证。如果您的验证是异步的，那么还有异步的版本—稍后将详细介绍。如果你想验证一个属性，无论何时设置，你可以这样做:

``` C#
private string _name
public string Name
{
   get { return this._name; }
   set
   {
      SetAndNotify(ref this._name, value);
      ValidateProperty();
   }
}
```

另外，可以通过调用 `Validate()` 对所有属性运行验证。

如果您希望在验证状态发生变化(任何属性的验证错误发生变化)时运行一些自定义代码，请覆盖 `OnValidationStateChanged() `。



## 理解和使用 IModelValidator

在接下来的几节中，我将带您通过使用一个非常有用的 [FluentValidation](http://fluentvalid.codeplex.com/) 库实现验证的示例。

FluentValidation 是通过创建一个新类来工作的，它实现了 `IValidator<T>` (通常通过扩展 `AbstractValidator<T>` 来实现，它可以验证特定类型的模型 `T`)。您需要创建一个新的实例，并使用它来运行验证。例如，如果你有一个 `UserViewModel`，你将定义一个 `UserViewModelValidator`，它扩展了 `AbstractValidator<UserViewModel>`，因此实现了 `IValidator<UserViewModel>`，就像这样:

``` C#
public class UserViewModel : Screen
{
   private string _name;
   public string Name
   {
      get { return this._name; }
      set { SetAndNotify(ref this._name, value); }
   }
}

public class UserViewModelValidator : AbstractValidator<UserViewModel>
{
   public UserViewModelValidator()
   {
      RuleFor(x => x.Name).NotEmpty();
   }
}
```

如果我们直接使用 `UserViewModelValidator` (没有 ValidatingModelBase 的帮助)，我们会做以下事情:

``` C#
public UserViewModel(UserViewModelValidator validator)
{
   this.Validator = validator;
}
// ...
this.Validator.Validate(this);
```

但是，使用 ValidatingModelBase 的意义在于它将自动运行和自动报告验证。如前所述，我们需要以ValidatingModelBase 知道如何与之交互的方式包装我们的 `UserViewModelValidator`。

做到这一点的最简单的方法是编写一个适配器，它可以接受 `IValidator<T> ` 的任何实现(即您编写的任何自定义验证器)，并以 ValidatingModelBase 能够理解的方式公开它。如果它消失了，我可以再次运行它:

- ValidatingModelBase.Validator 是一个 IModelValidator

- UserViewModelValidator是一个 IValidator<UserViewModel>

- 我们将编写一个适配器，FluentValidationAdapter<T>，这是一个 IModelValidator

- FluentValidationAdapter<T> 将接受一个 IValidator<T>，并封装它，以便可以通过 IModelValidator 访问它

- 因此，FluentValidationAdapter<UserViewModel> 将接受一个 UserViewModelValidator，并将其公开为IModelValidator;

明白了吗?这可能听起来工作量很大，但是我们可以让 IoC 容器完成大部分繁重的工作，我们很快就会看到这一点。

现在，在实践中会是什么样子呢?首先，还记得我说过 `IModelValidator<T>` 被定义为一个只实现 `IModelValidator` 的接口吗?我现在还不会告诉你为什么，但是请记住它们基本上是同义词。

``` C#
// Define the adapter
public class FluentValidationAdapter<T> : IModelValidator<T>
{
   public FluentValidationAdapter(IValidator<T> validator)
   {
      // Store the validator
   }

   // Implement all IModelValidator methods, using the stored validator
}

// This implements IValidator<UserViewModel>
public class UserViewModelValidator : AbtractValidator<UserViewModel>
{
   public UserViewModelValidator()
   {
      // Set up validation rules
   }
}

public class UserViewModel
{
   public UserViewModel(IModelValidator<UserViewModel> validator) : base(validator)
   {
      // ...
   }
}
```

看这里！如果我们要手动实例化一个新的 `UserViewModel `，我们会这样做:

``` C#
var validator = new UserViewModelValidator();
var validatorAdapter = new FluentValidationAdapter<UserViewModel>(validator);
var viewModel = new UserViewModel(validatorAdapter);
```

但是，我们可以配置 IoC 容器来完成这一任务。这里假设您使用的是 StyletIoC，尽管其他容器也可以进行类似的配置。

在你的 bootstrapper 中 重写 `ConfigureIoC`，首先，每当你要求一个 `IModelValidator<T> ` 时，告诉 StyletIoC 返回一个 `FluentValidationAdapter<T>` ，这样做:

``` C# 
builder.Bind(typeof(IModelValidator<>)).To(typeof(FluentValidationAdapter<>));
```

因此，无论何时 StyletIoC 都创建了一个新的 `UserViewModel`，它都会意识到它需要一个 `IModelValidator<UserViewModel>`。它知道已经被告知如何创建一个`IModelValidator<T>`——通过实例化一个新的 `FluentValidationAdapter<T>`。因此，它将尝试创建一个新的 `FluentValidationAdapter<UserViewModel> `，但是发现创建它需要一个新的 `IValidator<UserViewModel> `,这会因为找不到而失败。

因此，我们需要告诉 StyletIoC 如何创建一个新的 `IValidator<UserViewModel>`。我们*可以* 用这种比较长的方式，像这样：

``` C# 
// The long way
builder.Bind<IValidator<UserViewModel>>().To<UserViewModelValidator>();
```

但是，如果有很多验证器，就需要很多行配置。最好告诉S tyletIoC 发现所有的 `IValidator<T> `实现，并绑定它们本身，通过这样做:

``` C#
// The short way
builder.Bind(typeof(IValidator<>)).ToAllImplementations();
```

漂亮! 当 StyletIoC 试图创建一个新的 `FluentValidationAdapter<UserViewModel>` 时，它将看到它需要一个 `IValidator<UserViewModel>`，并将实例化一个新的`UserViewModelValidator`。

现在您可以看到为什么我们在这里使用 `IModelValidator<T>` 而不是 `IModelValidator`。如果 `UserViewModel` 需要一个`IModelValidator`，StyletIoC 就不能计算出它应该创建一个 `FluentValidationAdapter<UserViewModel>`，而不是一个 `FluentValidationAdapter<LogInViewModel>`。通过向 `IModelValidator` 添加类型信息，我们为 IoC 容器提供了足够的信息。



## 使用预制的 IModelValidator（Using a pre-made IModelValidator）

我写了以下 IModelValidator 实现，欢迎大家使用:

1. FluentValidationAdapter

如果你写了一个，你很乐意分享它，请让我知道，我会添加它。



## 实现同步验证适配器（Implementing IModelValidator (Synchronously)）

编写一个 `IModelValidator` 实现在概念上很简单，但是有一些问题。与前面一样，本节将假设我们正在为FluentValidation 库实现一个适配器，尽管您可以应用在这里学到的知识来为几乎所有的库编写一个适配器。

现在，我们假设所有的验证都是同步的。对于返回 Task 的方法，我们只返回一个已完成的 Task。一件容易的事。

首先，我们将实现 `IModelValidator<T>`，原因在前一节中讨论过。它还需要接受一个 `IValidator<T>` 作为构造函数参数，就像这样:

``` C#
public class FluentValidationAdapter : IModelValidator<T>
{
   private readonly IValidator<T> validator;
   public FluentValidationAdapter(IValidator<T> validator)
   {
      this.validator = validator;
   }
}
```

记住 `ValidatingModelBase` 需要一个 `IModelValidator`，它专门用于验证特定的 ViewModel 实例，因为它增加了更多的灵活性。这意味着 `ValidationModelBase` 可以调用 `ValidateAllPropertiesAsync()`，正确的 ViewModel 实例将被验证。然而，这里我们有一个鸡生蛋还是蛋生鸡的情况—为了使适配器专门化，ViewModel 必须存在。但是，只有在适配器被验证之后才能实例化 ViewModel，因为 ViewModel需 要适配器作为构造函数参数。

解决方案是 ` Initialize(object subject)  `方法。它被 `ValidatingModelBase` 调用，当它传递一个新的适配器时，它将把自己作为参数传递。然后适配器将存储这个实例，并在运行验证时使用它。是这样的:

``` c#
public class FluentValidationAdapter : IModelValidator<T>
{
   private readonly IValidator<T> validator;
   private T subject;

   public FluentValidationAdapter(IValidator<T> validator)
   {
      this.validator = validator;
   }

   public void Initialize(object subject)
   {
      this.subject = (T)subject;
   }
}
```

现在,实现 `ValidatePropertyAsync`。这应该验证单个属性，并返回验证错误列表，如果没有验证错误，则返回 null/emptyarray。使用 FluentValidation 执行同步验证，它可能是这样的:

``` C#
public Task<IEnumerable<string>> ValidatePropertyAsync(string propertyName)
{
   var errors = this.validator.Validate(this.subject, propertyName).Errors.Select(x => x.ErrorMessage);
   return Task.FromResult(errors);
}
```

类似地，`ValidateAllPropertiesAsync` 方法验证所有属性，并返回`{ propertyName => array of validation errors }`的字典。如果属性没有任何验证错误，您可以从字典中完全删除它，或者将其值设置为 null/emptyarray。

``` C#
public Task<Dictionary<string, IEnumerable<string>>> ValidateAllPropertiesAsync()
{
   var errors = this.validator.Validate(this.subject).Errors.GroupBy(x => x.PropertyName).ToDictionary(x => x.Key, x => x.Select(failure => failure.ErrorMessage));
   return Task.FromResult(errors);
}
```

把这些都放在一起，您就有了适配器!



## 实现异步验证适配器（Implementing IModelValidator (Asynchronously)）

实现异步验证(对于支持异步验证的库来说比较复杂)。

首先，请记住 `ValidatingModelBase` 有一组同步方法(`Validate`、`ValidateProperty`)和异步方法(`ValidateAsync`、`ValidatePropertyAsync`)。在底层，同步版本调用异步版本，但是阻塞线程，直到异步操作完成 (使用 `Task.Wait()`)。

现在，如果你经常使用任务，这应该会敲响警钟。你看，当你 `await DoSomethingAsync(); DoSomethingElse();` 时，你会说 “捕获当前线程 [\*] ”。当 `DoSomethingAsync()` 异步操作完成后，我想让你发布一条消息到那个捕获的线程，告诉它运行 `DoSomethingElse() `。但是，如果该线程正在等待异步操作完成，那么它将永远不会接收到该消息，操作将永远不会完成，并且会出现死锁。

[\*]不完全正确——它捕获当前的 `SynchronizationContext`。但是在UI线程上，这是相同的。

换句话说，这意味着下面的代码，从 UI 线程运行，将死锁:

``` c#
public async Task DoSomethingAsync()
{
   await Task.Delay(100);
}

// ... 
DoSomethingAsync().Wait();
DoSomethingElse();
```

当 `Task . Delay(100)` 任务完成时，它会向 UI 线程返回一条消息，说 “好的，运行 `DoSomethingElse()`”。但是，UI 线程被 `Wait()` 卡住了，永远不会处理消息，你就陷入了僵局。

为什么和这相关? 好。如果你写一个 `IModelValidator<T> ` 的方法，它看起来像这样:

``` C#
public async Task<IEnumerable<string>> ValidatePropertyAsync(string propertyName)
{
   var result = await this.Validator.ValidateAsync(this.subject, propertyName);
   return result.Errors.Select(x => x.ErrorMessage);
}
```

然后调用 `ValidateProperty `， **你会死锁**。

窍门是告诉 `await `不要捕获当前线程，使用 ` ConfigureAwait(false) `，即

``` C#
public async Task<IEnumerable<string>> ValidatePropertyAsync(string propertyName)
{
   var result = await this.Validator.ValidateAsync(this.subject, propertyName).ConfigureAwait(false);
   return result.Errors.Select(x => x.ErrorMessage);
}
```

现在，返回 `result.Errors…` 行将运行在另一个线程(而不是发布到 UI 线程)，没有死锁发生。