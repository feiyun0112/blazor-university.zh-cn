> 原文链接：https://blazor-university.com/templating-components-with-renderfragements/using-typeparam-to-create-generic-components/

# 使用 @typeparam 创建通用组件
[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/TemplatedComponents/UsingTypeParamToCreateGenericComponents)

Blazor 在构建过程中将其 `.razor` 文件转换为 `.cs` 文件。由于我们的 `razor` 文件不需要我们声明类名（它是从文件名中推断出来的），我们需要一种方法来额外指定生成的类何时应该是通用类。 `@typeparam` 指令允许我们在类上定义一个或多个逗号分隔的泛型参数。这在与通用 `RenderFragment<T>` 类结合使用时特别有用。

## 通用 DataList 组件
首先，我们需要在 **/Shared** 文件夹中创建一个 **DataList.razor** 文件，并将其标识为具有一个名为 `TItem` 的单个泛型参数的泛型类。我们还将添加一个 `[Parameter]` 属性，需要一个 `IEnumerable<TItem>`。

```
@typeparam TItem

@code
{
  [Parameter]
  public IEnumerable<TItem> Data { get; set; }
}
```

## 使用通用组件
创建一个具有三个属性的 `Person` 类。

```
public class Person
{
  public string Salutation { get; set; }
  public string GivenName { get; set; }
  public string FamilyName { get; set; }
}
```

在 `Index` 页面中，创建一些我们希望向用户显示的 `Person` 实例，并将它们传递给我们的新 `DataList` 组件。

```
<DataList Data=@People/>
@code
{
  private IEnumerable<Person> People;
  protected override void OnInitialized()
  {
    base.OnInitialized();
    People = new Person[]
    {
      new Person { Salutation = "Mr", GivenName = "Bob", FamilyName = "Geldof" },
      new Person { Salutation = "Mrs", GivenName = "Angela", FamilyName = "Rippon" },
      new Person { Salutation = "Mr", GivenName = "Freddie", FamilyName = "Mercury" }
    };
  }
}
```

## 使用 `RenderFragment<TItem>` 渲染数据
最后，我们将添加一个 `RenderFragment<TItem>` 属性并将其标记为 `[Parameter]`，以便使用 **razor** 文件可以指定一个模板来渲染 `Data` 属性中的每个 `TItem`。

最终的 **DataList.razor** 组件标记将如下所示。

```
@typeparam TItem
<ul>
  @foreach(TItem item in Data ?? Array.Empty<TItem>())
  {
    <li>@ChildContent(item)</li>
  }
</ul>
@code
{
  [Parameter]
  public IEnumerable<TItem> Data { get; set; }

  [Parameter]
  public RenderFragment<TItem> ChildContent { get; set; }
}
```

- 第 1 行

  指定此组件是通用的，并且有一个名为 TItem 的通用参数。

- 第 10-11 行

  声明一个名为 `Data` 的 `[Parameter]` 属性，它是 `ITem` 类型的可枚举属性。

- 第 13-14 行

  声明一个名为 `ChildContent` 的 `[Parameter]` 属性，它是一个 `RenderFragment<TItem>`——因此我们可以将 `TItem` 的实例传递给它，并让它给我们一些渲染的 HTML 以输出。

- 第 3 行

  遍历 `Data` 属性，并为每个元素呈现名为 `ChildContent` 的 `RenderFragment<TItem>`，方法是将当前元素传递给它。

## 最终源代码

### Index.razor
注意：已添加第 5 行以指定我们希望为每个元素呈现的 `ChildContext`。元素本身是通过 `@context` 变量传递的，因此 `RenderFragment<TItem>` 实际上是一个 `RenderFragment<Person>` - 因此 `@context` 是一个 `Person`，因此我们将受益于类型安全编译和 IntelliSense。

```
@page "/"

<h1>A generic list of Person</h1>
<DataList Data=@People>
  @context.Salutation @context.FamilyName, @context.GivenName
</DataList>

@code
{
  private IEnumerable<Person> People;
  protected override void OnInitialized()
  {
    base.OnInitialized();
    People = new Person[]
    {
      new Person { Salutation = "Mr", GivenName = "Bob", FamilyName = "Geldof" },
      new Person { Salutation = "Mrs", GivenName = "Angela", FamilyName = "Rippon" },
      new Person { Salutation = "Mr", GivenName = "Freddie", FamilyName = "Mercury" }
    };
  }
}
```

### DataList.razor
```
@typeparam TItem
<ul>
  @foreach(TItem item in Data ?? Array.Empty<TItem>())
  {
    <li>@ChildContent(item)</li>
  }
</ul>
@code
{
  [Parameter]
  public IEnumerable<TItem> Data { get; set; }

  [Parameter]
  public RenderFragment<TItem> ChildContent { get; set; }
}
```

### 生成的输出
```
<h1>A generic list of Person</h1>
<ul>
  <li>Mr Geldof, Bob</li>
  <li>Mrs Rippon, Angela</li>
  <li>Mr Mercury, Freddie</li>
</ul>
```


## 显式指定泛型参数类型
因为 `razor` 文件转换为 `C#` 类，所以我们不需要指定 `DataList` 所期望的泛型参数的类型，因为它是由我们设置 `Data =（IEnumerable<TItem> 的某些实例`）的编译器推断出来的。如果我们确实需要显式指定泛型参数类型，我们可以编写以下代码。

```
<SomeGenericComponent TParam1=Person TParam2=Supplier TItem=etc/>
```

**[下一篇 - 将占位符传递给 RenderFragments](/templating-components-with-renderfragements/passing-placeholders-to-renderfragments)**