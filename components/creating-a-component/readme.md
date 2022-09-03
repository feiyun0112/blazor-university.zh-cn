> 原文链接：https://blazor-university.com/components/creating-a-component/

# 创建组件
[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/Components/CreatingAComponent)

在客户端应用程序中创建一个名为 **Components** 的新文件夹。 这不是一个特殊的名称，我们可以选择任何我们想要的名称。

创建新的 **Components** 文件夹后，在其中创建一个名为 **MyFirstComponent.razor** 的文件并输入以下标记。

```
<div>
    <h2>This is my first component</h2>
</div>
```
现在编辑 **Index.razor** 文件。 此时，我们可以使用完全限定名称引用组件：

```
<CreatingAComponent.Client.Components.MyFirstComponent/>
```

或者编辑 **/_Imports.razor** 并添加 `@using CreatingAComponent.Client.Components`。 这里的 using 语句级联到所有 Razor 视图中——这意味着使用 **/Pages/Index.razor** 中的新组件的标记不再需要命名空间。

```
@page "/"

<h1>Hello, world!</h1>
<MyFirstComponent/>

Welcome to your new app.

<SurveyPrompt Title="How is Blazor working for you?" />
```
现在运行应用程序，我们将看到以下内容。
![](ThisIsMyFirstComponent.jpg)

 **[下一篇 - 单向绑定](https://feiyun0112.github.io/blazor-university.zh-cn/components/one-way-binding)**