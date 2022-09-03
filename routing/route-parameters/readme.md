> 原文链接：https://blazor-university.com/routing/route-parameters/

# 路由参数

[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/Routing/CapturingAParameterValue)

到目前为止，我们已经了解了如何将静态 URL 链接到 Blazor 组件。静态 URL 只对静态内容有用，如果我们希望同一个组件根据 URL 中的信息（例如客户 ID）呈现不同的视图，那么我们需要使用路由参数。

在添加组件的 `@page` 声明时，通过将其名称包装在一对 `{` 大括号 `}` 中来在 URL 中定义路由参数。

```
@page "/customer/{CustomerId}
```

## 捕获参数值
捕获参数的值就像添加具有相同名称的属性并用 `[Parameter]` 属性装饰它一样简单。

```
@page "/"
@page "/customer/{CustomerId}"

<h1>
  Customer:
  @if (string.IsNullOrEmpty(CustomerId))
  {
    @:None
  }
  else
  {
    @CustomerId
  }
</h1>
<h3>Select a customer</h3>
<ul>
  <li><a href="/customer/Microsoft">Microsoft</a></li>
  <li><a href="/customer/Google">Google</a></li>
  <li><a href="/customer/IBM">IBM</a></li>
</ul>

@code {
  [Parameter]
  public string CustomerId { get; set; }
}
```

请注意，当导航到解析为与当前页面相同类型的组件的新 URL 时，组件不会在导航之前被销毁，并且不会执行 `OnInitialized*` 生命周期方法。导航被简单地视为对组件参数的更改。

**[下一篇 - 路由参数约束](https://feiyun0112.github.io/blazor-university.zh-cn/routing/constraining-route-parameters)**