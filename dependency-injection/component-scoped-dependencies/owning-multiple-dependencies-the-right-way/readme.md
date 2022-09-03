> 原文链接：https://blazor-university.com/dependency-injection/component-scoped-dependencies/owning-multiple-dependencies-the-right-way/

# 拥有多个依赖项：正确的方式

在[上一节](https://feiyun0112.github.io/blazor-university.zh-cn/dependency-injection/component-scoped-dependencies/owning-multiple-dependencies-the-wrong-way/)中，我们看到了将多个拥有的依赖项注入组件的错误方法。本节将演示解决问题的正确方法。

如前所述，`OwningComponentBase<T>` 类组件将创建自己的依赖容器并在该容器中解析 `T` 的实例，因此 `T` 的实例对于我们的组件是私有的。

如果我们需要我们的组件私有地拥有多种依赖类型的实例，那么我们必须做更多的工作。为此，我们需要使用非泛型 `OwningComponentBase` 类。与通用版本一样，此组件将创建自己的依赖容器，该容器将在组件的生命周期内存在。但是，它不会为我们实际解析任何依赖项，而是让我们访问其依赖项容器，以便我们可以解析我们需要的任何类型的实例。

## 示例
[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/DependencyInjection/OwningMultipleDependenciesTheRightWay)


首先，创建一个新的 Blazor 应用程序。然后，就像我们之前所做的那样，我们将创建一些可以注入的类，这些类将使用状态成员来跟踪已创建的类实例的数量。

创建以下接口

```
public interface IOwnedDependency1
{
  public int InstanceNumber { get; }
}

public interface IOwnedDependency2
{
  public int InstanceNumber { get; }
}
```
然后创建实现这些接口的类。我将只显示第一类的代码，第二类将是相同的。

```
public class OwnedDependency1 : IOwnedDependency1
{
  private static volatile int PreviousInstanceNumber;

  public int InstanceNumber { get; }
  public OwnedDependency1()
  {
    InstanceNumber =
      System.Threading.Interlocked.Increment(ref PreviousInstanceNumber);
  }
}
```

将接口 + 它们的实现类注册为 `Scoped`（如果需要提醒您，请参阅[比较依赖范围](https://feiyun0112.github.io/blazor-university.zh-cn/dependency-injection/dependency-lifetimes-and-scopes/comparing-dependency-scopes/)）。

接下来，编辑 **Index.razor** 页面，以便我们应用程序的用户可以通过单击复选框来切换组件。

```
@page "/"

<input id="show-component" type=checkbox @bind=ShowComponent />
<label for="show-component">Show component</label>

@if (ShowComponent)
{
  <MyOwningComponent />
}

@code
{
  bool ShowComponent = false;
}
```
当 `ShowComponent` 为 `true` 时，我们的标记将创建 `MyOwningComponent` 的一个实例并渲染它。接下来，我们将创建 `MyOwningComponent`。

## OwningComponentBase

在 **Shared** 文件夹中，创建一个名为 `MyOwningComponent` 的新 Razor 组件。我们将从 `OwningComponentBase` 中派生此组件。

```
@inherits OwningComponentBase
```


然后创建一些类字段来保存依赖项。

```
@code
{
  private IOwnedDependency1 OwnedDependency1;
  private IOwnedDependency2 OwnedDependency2;
}
```

## 解决拥有的依赖关系

`OwningComponentBase` `创建的私有依赖性容器通过其ScopedServices` 属性提供给我们。

```
protected IServiceProvider ScopedServices { get; }
```

我们可以使用这个 `IServiceProvider` 来解析组件所拥有的私有依赖容器中的依赖实例。


```
@inherits OwningComponentBase
@using Microsoft.Extensions.DependencyInjection


<div>
  OwnedDependency1.InstanceNumber = @OwnedDependency1.InstanceNumber
</div>
<div>
  OwnedDependency2.InstanceNumber = @OwnedDependency2.InstanceNumber
</div>

@code
{
  private IOwnedDependency1 OwnedDependency1;
  private IOwnedDependency2 OwnedDependency2;

  protected override void OnInitialized()
  {
    OwnedDependency1 =
      ScopedServices.GetService<IOwnedDependency1>();
    OwnedDependency2 =
      ScopedServices.GetService<IOwnedDependency2>();
  }
}
```

- 第 1 行

  从 `OwningComponentBase` 继承来给我们自己的私有依赖容器。

- 第 2 行

  使用 `DependencyInjection` 命名空间，因此我们可以在 `IServiceProvider` 上使用 `GetService<T>` 扩展方法。

- 第 19 & 21 行

  使用 `OwningComponentBase.ScopedServices` 属性来解析组件所需的依赖项实例。

- 第 6 & 9 行

  显示为我们创建的依赖项的实例号。

## 运行示例

如果我们运行示例应用并勾选复选框，我们将看到以下输出。

- OwnedDependency1.InstanceNumber = 1
- OwnedDependency2.InstanceNumber = 1

取消勾选该复选框以允许删除我们的组件，然后再次勾选该复选框以让 Blazor 创建 `MyOwningComponent` 的新实例。渲染输出现在应该如下所示。

- OwnedDependency1.InstanceNumber = 2
- OwnedDependency2.InstanceNumber = 2

 
这表明，每次创建组件时，我们在组件的 `OnInitialized` 方法中解析的两个依赖项都是新的实例。

## 依赖生命周期

`OwningComponentBase` 类实现 `IDisposable` `接口。当从OwningComponentBase` 派生的任何组件不再呈现时，Blazor 将在 `OwningComponentBase` 上执行 `Dispose` 方法。



组件上的 `Dispose` 方法将对其拥有的私有依赖项容器调用`Dispose`。反过来，该容器创建的任何实现 `IDisposable` 的对象实例也将执行其 `Dispose` 方法。



要演示这种行为，请对应用程序进行以下更改。



首先，在我们的组件上重写 `Dispose(bool isDisposing)`，并让它在被释放时输出日志。



```
public void Dispose()
{
  System.Diagnostics.Debug.WriteLine("Disposing " + GetType().Name);
}
```


然后，对于我们的每个依赖类（`OwnedDependency1` 和 `OwnedDependency2`），让它们实现 `IDisposable`，并且再次让它们在执行 `Dispose` 时输出日志。

```
  public class OwnedDependency1 : IOwnedDependency1, IDisposable
  {
    ... Other code omitted for brevity ...

    public void Dispose()
    {
      System.Diagnostics.Debug.WriteLine($"Created {GetType().Name} instance {InstanceNumber}");
    }
  }
```

我们还可以在类的构造函数中添加一些日志记录。



现在运行应用程序并切换复选框将输出类似于以下内容的日志文本。



- Created MyOwningComponent
- Created OwnedDependency1 instance 1
- Created OwnedDependency2 instance 1
- Disposing OwnedDependency2 instance 1
- Disposing OwnedDependency1 instance 1
- Disposing MyOwningComponent
- Created MyOwningComponent
- Created OwnedDependency1 instance 2
- Created OwnedDependency2 instance 2
- Disposing OwnedDependency2 instance 2
- isposing OwnedDependency1 instance 2
- Disposing MyOwningComponent

## 结论

当您的组件只需要拥有一个依赖项时，从 `OwningComponentBase<T>` 派生；当您的组件需要拥有多个依赖项时，从非泛型 `OwningComponentBase` 派生。



尽管解析组件依赖项实例的过程是一个手动过程，但不需要处理任何创建的依赖项，因为组件的依赖项容器将在 `OwningComponentBase.Dispose` 时处理它们。