> 原文链接：https://blazor-university.com/routing/detecting-navigation-events/

# 检测导航事件
[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/Routing/NavigatingViaCode)

从 Blazor 访问浏览器导航是通过 `NavigationManager` 服务提供的。这可以使用 razor 文件中的 `@inject` 或 CS 文件中的 `[Inject]` 属性注入到 Blazor 组件中。

## LocationChanged 事件
`LocationChanged` 是在浏览器中的 URL 发生更改时触发的事件。它传递一个 `LocationChangedEventArgs` 实例，该实例提供以下信息：

```
public readonly struct LocationChangedEventArgs
{
  public string Location { get; }
  public bool IsNavigationIntercepted { get; }
}
```

`Location` 属性是显示在浏览器中的完整 URL，包括协议、路径和任何查询字符串。

`IsNavigationIntercepted` 指示导航是通过代码还是通过 HTML 导航启动的。

- `false`

  导航由从代码调用的 `NavigationManager.NavigateTo` 启动。

- `true`

  用户单击 HTML 导航元素（例如 `a href`），Blazor 拦截导航，而不是允许浏览器实际导航到新 URL，这将导致向服务器发出请求。在其他情况下也是如此，例如页面上的某些 JavaScript 导致导航（例如，在超时之后）。最终，任何不是通过 `NavigationManager.NavigateTo` 发起的导航事件都将被视为拦截导航，并且该值为 `true`。

请注意，目前无法拦截导航并阻止其继续进行。

## 观察 OnLocationChanged 事件
需要注意的是，`NavigationManager` 服务是一个长期存在的实例。因此，任何订阅其 `LocationChanged` 事件的组件都将在服务的生命周期内被强引用。因此，重要的是我们的组件在它们被销毁时也取消订阅此事件，否则它们将不会被垃圾收集。

目前，`ComponentBase` 类在销毁时没有生命周期事件，但可以实现 `IDisposable` 接口。

```
@implement IDisposable
@inject NavigationManager NavigationManager

protected override void OnInitialized()
{
  // Subscribe to the event
  NavigationManager.LocationChanged += LocationChanged;
  base.OnInitialized();
}

void LocationChanged(object sender, LocationChangedEventArgs e)
{
  string navigationMethod = e.IsNavigationIntercepted ? "HTML" : "code";
  System.Diagnostics.Debug.WriteLine($"Notified of navigation via {navigationMethod} to {e.Location}");
}

void IDisposable.Dispose()
{
  // Unsubscribe from the event when our component is disposed
  NavigationManager.LocationChanged -= LocationChanged;
}
```

**[下一篇 - 表单](https://feiyun0112.github.io/blazor-university.zh-cn/forms)**