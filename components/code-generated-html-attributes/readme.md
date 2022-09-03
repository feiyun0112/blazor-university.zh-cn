> 原文链接：https://blazor-university.com/components/code-generated-html-attributes/

# 代码生成 HTML 属性


Razor 在条件 HTML 输出或在 for 循环中输出 HTML 时非常棒，但在元素本身内的条件代码方面，事情就有点棘手了。例如，以下代码无法编译，因为您无法在元素的 `<` 和 `>` 内添加 C# 控制块。

```
<img
  @foreach(var nameAndValue in AdditionalAttributes)
  {
    @nameAndValue.Key = @nameAndValue.Value
  } 
  src="https://randomuser.me/api/portraits/lego/1.jpg" />

@code
{
  Dictionary<string, object> AdditionalAttributes;

  protected override void OnInitialized()
  {
    AdditionalAttributes = new Dictionary<string, object>
    {
      ["id"] = "EmmetImage",
      ["alt"] = "A photo of Emmet"
    };
    base.OnInitialized();
  }
}
```

我们可能尝试的下一个方法是编写一个返回字符串的方法，并在 `<` 和 `>` 字符内调用它。

```
<div @IfYouCanSeeThisTextThenTheCodeWasNotExecutedHere />
<span>@IfYouCanSeeThisTextThenTheCodeWasNotExecutedHere</span>

@code
{
  string IfYouCanSeeThisTextThenTheCodeWasNotExecutedHere = "The code here was executed";
}
```
但这也不起作用。前面的示例将输出以下 HTML。

```
<div @ifyoucanseethistextthenthecodewasnotexecutedhere=""></div>
<span>The code here was executed</span>`
```

Razor 只会在以下位置执行 C# 代码：

1. 在元素的内容区域内，例如 `<span>@GetSomeHtml()</span>`。
2. 在确定要分配给元素属性的值时，例如 `<img src=@GetTheImageForTheUrl() />`。
3. 在 `@code` 部分中。

我们需要用来为 HTML 元素生成一个或多个属性 + 值的技术称为“属性展开”。属性展开涉及将 `Dictionary<string, object>` 分配给具有特殊名称 `@attribute` 的属性。

```
<div @attributes=MyCodeGeneratedAttributes/>

@code
{
  Dictionary<string, object> MyCodeGeneratedAttributes;

  protected override void OnInitialized()
  {
    MyCodeGeneratedAttributes = new Dictionary<string, object>();
    for(int index = 1; index <= 5; index++)
    {
      MyCodeGeneratedAttributes["attribute_" + index] = index;
    }
  }
}
```
前面的代码将输出一个具有 5 个属性的 `<div>`。

```
<div attribute_1="1" attribute_2="2" attribute_3="3" attribute_4="4" attribute_5="5"></div>
```

## 特殊情况

一些 HTML 属性，例如 `readonly` 和 `disabled` 不需要任何值——它们仅存在于元素上就足以使它们生效。事实上，即使应用诸如 `false` 之类的值仍然会激活它们。以下 `<input>` 元素将是只读和禁用的。

```
<input readonly="false" disbabled="false"/>
```

在 razor 视图中，规则略有不同。如果我们输出 `readonly=@IsReadOnly` 或 `disabled=@IsDisabled` - 只要分配的值为 `false`，razor 根本不会输出该属性；当分配的值为 `true` 时，razor 将在不分配值的情况下输出元素。

`<input readonly=@true disabled=@false/>` 将导致生成的 HTML 完全不包含 disabled 属性。

**[下一篇 - 捕获意外参数](https://feiyun0112.github.io/blazor-university.zh-cn/components/capturing-unexpected-parameters)**