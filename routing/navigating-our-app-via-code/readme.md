> 原文链接：https://blazor-university.com/routing/navigating-our-app-via-code/

# 通过代码导航
[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/Routing/NavigatingViaCode)

 

从 Blazor 访问浏览器导航是通过 `NavigationManager` 服务提供的。这可以使用 razor 文件中的 `@inject` 或 CS 文件中的 `[Inject]` 属性注入到 Blazor 组件中。

`NavigationManager` 服务有两个特别的成员； `NavigateTo` 和 `LocationChanged`。

`LocationChanged` 事件将在[检测导航事件](https://feiyun0112.github.io/blazor-university.zh-cn/routing/detecting-navigation-events/)中更详细地解释。

## NavigateTo 方法
`NavigationManager.NavigateTo` 方法使 C# 代码能够控制浏览器的 URL。与截获的导航一样，浏览器实际上并不导航到新的 URL。相反，浏览器中的 URL 被替换，之前的 URL 被插入到浏览器的导航历史中，但没有向服务器请求新页面的内容。通过 `NavigateTo` 进行的导航将触发 `LocationChanged` 事件，传递新 URL 并为 `IsNavigationIntercepted` 传递 `false`。

对于此示例，我们将再次更改标准 Blazor 模板。我们将使用我们之前在[路由参数](https://feiyun0112.github.io/blazor-university.zh-cn/routing/route-parameters/)和[可选路由参数](https://feiyun0112.github.io/blazor-university.zh-cn/routing/optional-route-parameters/)中学到的知识。

首先，删除 **Index.razor** 和 **FetchData.razor** 页面，并删除 **NavMenu.razor** 文件中指向这些页面的链接。同样在 `NavMenu` 中，将 `counter` 链接的 `href` 更改为 为 `href=""`，因为我们将使其成为默认页面。

编辑 `Counter.razor` 并给它两个路由，“/” 和 “/counter/{CurrentCount:int}”，同时确保它来自 `CounterBase` 类，这样我们就可以在浏览器的控制台窗口中看到导航日志 - `CounterBase.cs` 文件之前在 `OnLocationChanged` 部分中进行了概述。

```
@page "/"
@page "/counter/{CurrentCount:int}"
@inherits CounterBase
```

我们还需要更改 `currentCount` 字段，使其成为具有 `getter` 和 `setter` 的属性，并将其装饰为 `[Parameter]`。请注意，它也已从 `camelCase` 重命名为 `PascalCase`。

```
[Parameter]
public int CurrentCount { get; set; }
```


我们现在有一个 counter 页面，可以简单地访问应用程序的主页，也可以通过指定 `/counter/X` 来访问，其中 `X` 是一个整数值。

`NavigationManager` 被注入到我们的 `CounterBase` 类中，因此可以在 `Counter.razor` 文件中访问。

```
@code {
  [Parameter]
  public int CurrentCount { get; set; }

  bool forceLoad;

  void AlterBy(int adjustment)
  {
    int newCount = CurrentCount + adjustment;
    UriHelper.NavigateTo("/counter/" + newCount, forceLoad);
  }
}
```

我们将从两个按钮调用 `AlterBy` 方法，一个用于增加 `CurrentCount`，一个用于减少它。还有一个用户可以选择的选项 `forceLoad`，它将在调用 `NavigateTo` 时设置相关参数，以便我们看到差异。整个文件最终应如下所示：

```
@page "/"
@page "/counter/{CurrentCount:int}"
@implements IDisposable
@inject NavigationManager NavigationManager

<h1>Counter value = @CurrentCount</h1>

<div class="form-check">
  <input @bind=@forceLoad type="checkbox" class="form-check-input" id="ForceLoadCheckbox" />
  <label class="form-check-label" for="ForceLoadCheckbox">Force page reload on navigate</label>
</div>

<div class="btn-group" role="group">
  <button @onclick=@( () => AlterBy(-1) ) class="btn btn-primary">-</button>
  <input value=@CurrentCount readonly class="form-control" />
  <button @onclick=@( () => AlterBy(1) ) class="btn btn-primary">+</button>
</div>
<a class="btn btn-secondary" href="/Counter/0">Reset</a>
<p>
  <em>Page redirects to ibm.com when count hits 10!</em>
</p>

@code {
  [Parameter]
  public int CurrentCount { get; set; }

  bool forceLoad;

  void AlterBy(int adjustment)
  {
    int newCount = CurrentCount + adjustment;

    if (newCount >= 10)
      NavigationManager.NavigateTo("https://ibm.com");

    NavigationManager.NavigateTo("/counter/" + newCount, forceLoad);
  }

  protected override void OnInitialized()
  {
    // Subscribe to the event
    NavigationManager.LocationChanged += LocationChanged;
    base.OnInitialized();
  }

  private void LocationChanged(object sender, LocationChangedEventArgs e)
  {
    string navigationMethod = e.IsNavigationIntercepted ? "HTML" : "code";
    System.Diagnostics.Debug.WriteLine($"Notified of navigation via {navigationMethod} to {e.Location}");
  }

  void IDisposable.Dispose()
  {
    // Unsubscribe from the event when our component is disposed
    NavigationManager.LocationChanged -= LocationChanged;
  }
}
```
单击 `-` 或 `+` 按钮将调用 `AlterBy` 方法，该方法将指示 `NavigationManager` 服务导航到 `/counter/X`，其中 `X` 是调整后的 `CurrentCount` 的值——在浏览器的控制台中产生以下输出：

```
WASM：通过代码通知导航到 http://localhost:6812/counter/1
WASM：通过代码通知导航到 http://localhost:6812/counter/2
WASM：通过代码通知导航到 http://localhost:6812/counter/3
WASM：通过代码通知导航到 http://localhost:6812/counter/4
```

单击重置链接将导航到 `/counter/0`，重置 `CurrentCount` 的值。

```
WASM：通过代码通知导航到 http://localhost:6812/counter/1
WASM：通过代码通知导航到 http://localhost:6812/counter/2
WASM：通过代码通知导航到 http://localhost:6812/counter/3
WASM：通过代码通知导航到 http://localhost:6812/counter/4
WASM：通过 HTML 通知导航到 http://localhost:6812/Counter/0
```

## ForceLoad
`forceLoad` 参数指示 Blazor 绕过其自己的路由系统，而是让浏览器实际导航到新 URL。这将导致对服务器的 HTTP 请求以检索要显示的内容。

请注意，导航到站外 URL 不需要强制加载。调用 `NavigateTo` 到另一个域将调用完整的浏览器导航。

使用本节的 GitHub 示例。在浏览器的控制台窗口中查看 `IsNavigationIntercepted` 在通过按钮和重置链接导航时有何不同，并在浏览器的网络窗口中查看根据您是否：

 - 将 forceLoad 设置为 false 进行导航。
- 将 forceLoad 设置为 true 进行导航。
- 导航到站外 URL。

要观察最后一种情况，您可能希望更新 `AdjustBy` 方法以在 `CurrentValue` 传递特定值时进行站外导航。

```
void AlterBy(int adjustment)
{
  int newCount = CurrentCount + adjustment;

  if (newCount >= 10)
    NavigationManager.NavigateTo("https://ibm.com");

  NavigationManager.NavigateTo("/counter/" + newCount, forceLoad);
}
```


**[下一篇 - 检测导航事件](https://feiyun0112.github.io/blazor-university.zh-cn/routing/detecting-navigation-events)**