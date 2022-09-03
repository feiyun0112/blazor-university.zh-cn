> 原文链接：https://blazor-university.com/components/cascading-values/cascading-values-by-type/

# 按类型级联值
[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/CascadingValues/CascadingValuesByType)

之前我们看到了如何通过名称级联一个值。设置**名称**很重要，因为它用于通过匹配名称来将 `CascadingValue` 中指定的值推送到使用组件的正确属性中。另一种选择是在不指定名称的情况下指定 `CascadingValue`，当 Blazor 遇到以这种方式指定的级联值时，如果属性满足以下条件，它会将其值注入到组件的属性中。

1. 该属性使用 `CascadingPropertyAttribute` 进行装饰。
2. [CascadingProperty] 没有指定名称。
3. 该属性与 [CascadingValue] 中设置的类型相同（例如布尔值）。
4. 该属性有一个设置器。
5. 该属性是 public 的。

例如，以下 `CascadingValue` 将匹配 `SomeComponent` 中的两个 `CascadingParameter` 属性。

```
<CascadingValue Value=@true>
  <SomeComponent/>
</CascadingValue>
```

```
Property1 = @Property1
Property2 = @Property2

@code
{
  [CascadingParameter]
  private bool Property1 { get; set; }

  [CascadingParameter]
  private bool Property2 { get; set; }
}
```

未命名的 `CascadingValue` 不如指定了 `Name` 的 `CascadingValue` 那样具体，因为每个具有正确类型且没有 `Name` 的 `CascadingParameter` 修饰属性都会使用该值。在我们定义一个简单的 .NET 类型（例如 `bool` 或 `int`）的情况下，建议我们使用命名参数，但是，有时值的类型足以识别其用途；指定名称将是多余的，因此排除它可以节省一点时间。

随着求职申请的增长，我们最终可能会得到多个级联参数，例如：

- `bool ViewAnonymizedData`

  指示是否应隐藏个人身份信息。
- `string DateFormat`

  使用组件可以使用它以统一的方式格式化日期。

- `string LanguageCode`

  组件可以使用它来显示翻译后的文本。


这里出现的明显模式是这些都与用户的偏好有关。而不是使用带有多个 `CascadingValue` 元素的 Razor 标记，如下所示：

```
<CascadingValue Name="ViewAnonymizedData" Value=@ViewAnonymizedData>
  <CascadingValue Name="DateFormat" Value=@DateFormat>
    <CascadingValue Name="LanguageCode" Value=@LanguageCode>
      (Body goes here)
    </CascadingValue>
  </CascadingValue>
</CascadingValue>
```

拥有一个自定义类会更有意义（并且需要更少的代码）：

```
public class UserPreferences
{
  public bool ViewAnonymizedData { get; set; }
  public string DateFormat { get; set; }
  public string LanguageCode { get; set; }
}
```

然后像这样创建你的 Razor 标记：

```
<CascadingValue Value=@UserPreferences>
</CascadingValue>
```

然后，使用组件只需要一个标记为 `[CascadingParameter]` 的属性，而不是三个。

```
@if (!UserPreferences.ViewAnonymizedData)
{
  <div>
    <span>Name</span> @Candidate.Name
  </div>
  <div>
    <span>Date of birth</span> @Candidate.DateOfBirth.ToString(UserPreferences.DateFormat)
  </div>
  <ViewAddress Address=@Candidate.Address/>
}
else
{
  <span>[Anonmymized view]</span>
}

@code
{
  [CascadingParameter]
  private UserPreferences UserPreferences { get; set; }
}
```

当然，这个例子忽略了如何根据 `UserPreferences.LanguageCode` 翻译静态文本。

**[下一篇 - 重写级联值](/components/cascading-values/overriding-cascaded-values)**