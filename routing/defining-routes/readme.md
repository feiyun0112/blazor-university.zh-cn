> 原文链接：https://blazor-university.com/routing/defining-routes/

# 定义路由
要定义路由，我们只需在任何组件的顶部添加 @page 声明。

```
@page "/"

<h1>Hello, world!</h1>

Welcome to your new app.
```

如果我们打开此视图的生成源代码（在 **obj\Debug\netcoreapp3.0\Razor\Pages\Index.razor.g.cs** 中），我们会看到编译为以下代码的 @page 指令。

```
[Microsoft.AspNetCore.Components.LayoutAttribute(typeof(MainLayout))]
[Microsoft.AspNetCore.Components.RouteAttribute("/")]
public class Index : Microsoft.AspNetCore.Components.ComponentBase
{
}
```

`@page` 指令在组件的类上生成一个 `RouteAttribute`。在启动期间，Blazor 会扫描使用 `RouteAttribute` 修饰的类并相应地构建其路由定义。

## 路由发现
路由发现由 Blazor 在其默认项目模板中自动执行。如果我们查看 `App.razor` 文件，我们将看到路由器组件的使用。

```
… other code …
    <Router AppAssembly="typeof(Startup).Assembly">
        … other code …
    </Router>
… other code … 
```

`Router` 组件扫描指定程序集中实现 `IComponent `的所有类，然后反射该类以查看它是否使用任何 `RouteAttribute` 属性进行修饰。对于它找到的每个 `RouteAttribute`，它都会解析其 URL 模板字符串，并将从 URL 到组件的关系添加到其内部路由表中。

这意味着单个组件可以用零个、一个或多个 `RouteAttribute` 属性（`@page` 声明）装饰。零个 `RouteAttribute` 属性的组件无法通过 URL 访问，而具有多个 `RouteAttribute` 属性的组件可以通过它指定的任何 URL 模板访问。


```
@page "/"
@page "/greeting"
@page "/HelloWorld"
@page "/hello-world"

<h1>Hello, world!</h1>
```

页面也可以在[组件库](https://feiyun0112.github.io/blazor-university.zh-cn/component-libraries)中定义。

**[下一篇 - 路由参数](https://feiyun0112.github.io/blazor-university.zh-cn/routing/route-parameters)**