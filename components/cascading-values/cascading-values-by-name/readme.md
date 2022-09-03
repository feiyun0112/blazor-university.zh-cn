> 原文链接：https://blazor-university.com/components/cascading-values/cascading-values-by-name/

# 按名称级联值
[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/CascadingValues/CascadingValuesByName)

指定级联参数的值非常简单。在我们的 Razor HTML 标记中的任何时候，我们都可以创建一个 `CascadingValue` 元素。该元素内呈现的所有内容都可以访问指定的值。

```
@page "/"

<h1>Toggle the options</h1>
<input @bind-value=FirstOptionValue type="checkbox" /> First option
<br />
<input @bind-value=SecondOptionValue type="checkbox" /> Second option
<br />

<CascadingValue Name="FirstOption" Value=@FirstOptionValue>
  <CascadingValue Name="SecondOption" Value=@SecondOptionValue>
    <FirstLevelComponent />
  </CascadingValue>
</CascadingValue>

@code {
  bool FirstOptionValue;
  bool SecondOptionValue;
}
```

使用级联值同样简单。任何组件，无论它在 `CascadingValue` 元素中的嵌套程度如何，都可以通过使用 `CascadingParameter` 属性修饰的属性来访问该值。

```
<ul>
  <li>FirstOption = @FirstOption</li>
  <li>SecondOption = @SecondOption</li>
</ul>

@code {
  [CascadingParameter(Name="FirstOption")]
  private bool FirstOption { get; set; }

  [CascadingParameter(Name="SecondOption")]
  private bool SecondOption { get; set; }
}
```

请注意，我们使用该值的属性的名称是无关紧要的。 Blazor 不会查找在 `CascadingValue` 元素中指定的同名属性；我们可以随意命名我们的属性，它实际上用的是 `CascadingParameterAttribute` 上的 `Name` ，它标识应该注入哪个级联值。

将充当级联参数的属性的可见性设置为私有是一种很好的做法。允许他们通过代码对使用者进行设置是不合逻辑的，因为该值实际上由设置 `Cascading` 值的父级所有。

**[下一篇 - 按类型级联值](/components/cascading-values/cascading-values-by-type)**