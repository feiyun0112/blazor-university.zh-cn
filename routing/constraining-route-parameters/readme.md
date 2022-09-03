> 原文链接：https://blazor-university.com/routing/constraining-route-parameters/

# 路由参数约束
[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/Routing/ConstrainingRouteParameters)

除了能够指定包含参数的 URL 模板外，还可以确保 Blazor 仅在参数值满足特定条件时才将 URL 与组件匹配。

例如，在采购订单编号始终为整数的应用程序中，我们希望 URL 中的参数与我们的组件相匹配，以便仅当 `OrderNumber` 的 URL 值实际上是一个数字时才显示采购订单。

要为参数定义约束，它以冒号后缀，然后是约束类型。例如 `:int` 只会匹配在正确的位置包含一个有效的整数值的 URL。

```
@page "/"
@page "/purchase-order/{OrderNumber:int}"

<h1>
  Order number:
  @if (!OrderNumber.HasValue)
  {
    @:None
  }
  else
  {
    @OrderNumber
  }
</h1>
<h3>Select a purchase order</h3>
<ul>
  <li><a href="/purchase-order/1/">Order 1</a></li>
  <li><a href="/purchase-order/2/">Order 2</a></li>
  <li><a href="/purchase-order/42/">Order 42</a></li>
</ul>

@code {
  [Parameter]
  public int? OrderNumber { get; set; }
}
```

## 约束类型
约束 | .NET 类型 | 有效值 | 无效值
--- | --- | --- | ---
:bool |	System.Boolean	| true<br> false| 1<br> Hello
:datetime |	System.DateTime |	2001-01-01 <br> 02-29-2000 | 29-02-2000
:decimal |	System.Decimal	| 2.34 <br> 0.234 | 2,34 <br> ૦.૨૩૪
:double |	System.Double	 | 2.34 <br>  0.234 | 2,34 <br> ૦.૨૩૪
:float |	System.Single	| 2.34 <br> 0.234 | 2,34 <br> ૦.૨૩૪
:guid |	System.Guid	| 99303dc9-8c76-42d9-9430-de3ee1ac25d0 | {99303dc9-8c76-42d9-9430-de3ee1ac25d0}
:int	| System.Int32	| -1 <br> 42 <br> 299792458 | 12.34 <br> ૨૩
:long	| System.Int64	| -1 <br> 42 <br> 299792458 | 12.34 <br> ૨૩


## 本地化
Blazor 约束当前不支持本地化。

- 数字只有在 `0..9` 的形式下才被认为是有效的，而不是来自非英语语言，例如 `૦..૯`（古吉拉特语）。

- 日期仅以 `MM-dd-yyyy`、`MM-dd-yy` 或 ISO 格式 `yyyy-MM-dd` 的形式有效。

- 布尔值必须为 `true` 或 `false`。

## 不支持的约束类型
Blazor 约束不支持以下约束类型，但希望将来会支持：

- **贪心参数**

    在 ASP.NET MVC 中，可以提供一个以星号开头的参数名称，并捕获包括正斜杠在内的 URL 块。

    `/articles/{Subject}/{*TheRestOfTheURL}`

- **正则表达式**

    Blazor 当前不支持基于正则表达式约束参数的能力。

 - **枚举**

    目前不可能约束参数以匹配枚举的值。

- **自定义约束**

    无法定义自定义类来确定传递给参数的值是否有效。



**[下一篇 - 可选路由参数](/routing/optional-route-parameters)**