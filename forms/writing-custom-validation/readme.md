> 原文链接：https://blazor-university.com/forms/writing-custom-validation/

# 编写自定义验证
[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/Forms/CustomValidation)

请注意，与有关 [EditContext、FieldIdentifiers 和 FieldState](https://feiyun0112.github.io/blazor-university.zh-cn/forms/editcontext-fieldidentifiers-and-fieldstate/) 的部分一样，这是一个高级主题。

如前所述，`FieldState` 类保存表单数据的元状态。除了指示值是否已被手动编辑外，Blazor 还存储验证错误消息的集合。为了了解它的工作原理，本节将说明如何创建我们自己的自定义验证机制，该机制可与 Blazor 一起使用来验证用户输入。

下面的 UML 图显示了 `EditForm` 和存储此元状态的各种类（在图中分组）之间的关系。请记住，每当 `EditForm.Model` 发生更改时，`EditForm` 都会创建一个新的 `EditContext` 实例。然后可以对以前的 `EditContext`（不再需要它，因为它包含有关以前模型的信息）进行垃圾收集，并且可以对图表中分组的所有类实例进行垃圾收集。

![](ValidationMessageStoresUML2.png)


我们的自定义验证将基于 [FluentValidation](https://github.com/JeremySkinner/FluentValidation)。完成本节后（或者如果您只是想要一些可以立即使用的东西），请查看 [blazor-validation](https://github.com/mrpmorris/blazor-validation)。


## 创建验证器组件
我们的验证器组件不必为了提供验证而从任何特定类继承。唯一的要求是它来自 Blazor `ComponentBase` 类，以便我们可以将其添加到视图中的 `<EditForm>` 标记中。嵌入 `<EditForm>` 标记的目的是为了让我们可以定义一个[级联参数](https://feiyun0112.github.io/blazor-university.zh-cn/components/cascading-values/)，以便在 `EditForm` 的 `Model` 参数更改时获取当前由 `EditForm` 创建的 `EditContext`。

首先，创建一个新的 Blazor 应用并添加对 [FluentValidation NuGet 包](https://www.nuget.org/packages/FluentValidation/)的引用。然后创建一个名为 `FluentValidationValidator` 的类。

```
public class FluentValidationValidator : ComponentBase
{
  [CascadingParameter]
  private EditContext EditContext { get; set; }

  [Parameter]
  public Type ValidatorType { get; set; }

  private IValidator Validator;
  private ValidationMessageStore ValidationMessageStore;
  [Inject]
  private IServiceProvider ServiceProvider { get; set; }
}
```

- EditContext

  从其父 `<EditForm>` 组件传递给我们的组件的级联参数。每次 `EditForm.Model` 更改时，这都会更改。

- ValidatorType

  这将指定用于执行实际验证的类类型。我们将检查这是一个 `IValidator`（一个 FluentValidation 接口）。

- Validator

  这将保存对指定 `ValidatorType` 实例的引用，以执行实际的对象验证。

- ValidationMessageStore

  每次我们的 `EditContext` 更改（因为 `EditForm.Model` 已更改）时，我们都会创建一个新的。

- ServiceProvider

  对 `IServiceProvider` 的注入依赖项，我们可以使用它来创建 `ValidatorType` 的实例。

```
public override async Task SetParametersAsync(ParameterView parameters)
{
  // Keep a reference to the original values so we can check if they have changed
  EditContext previousEditContext = EditContext;
  Type previousValidatorType = ValidatorType;

  await base.SetParametersAsync(parameters);

  if (EditContext == null)
    throw new NullReferenceException($"{nameof(FluentValidationValidator)} must be placed within an {nameof(EditForm)}");

  if (ValidatorType == null)
    throw new NullReferenceException($"{nameof(ValidatorType)} must be specified.");

  if (!typeof(IValidator).IsAssignableFrom(ValidatorType))
    throw new ArgumentException($"{ValidatorType.Name} must implement {typeof(IValidator).FullName}");

  if (ValidatorType != previousValidatorType)
    ValidatorTypeChanged();

  // If the EditForm.Model changes then we get a new EditContext
  // and need to hook it up
  if (EditContext != previousEditContext)
    EditContextChanged();
}
```

- 第 4-5 行

  每当我们的参数之一更改（包括我们的 `EditContext` 级联参数）时，都会执行 `SetParametersAsync`。我们需要做的第一件事是保留对一些原始值的引用，以便我们可以查看它们是否已更改并做出相应的反应。

- 第 7 行

  调用 `base.SetParametersAsync` 会将我们对象的属性更新为新值。

- 第 9-16 行

  确保我们有一个 `EditContext` 和一个作为 `IValidator` 的 `ValidatorType`。

- 第 18-19 行

  如果 `ValidatorType` 已更改，那么我们需要创建该类型的新实例并将其分配给我们的私有 `Validator` 字段以验证我们的 `EditContext.Model`。

- 第 23-24 行

  如果 `EditContext` 发生了变化，那么我们需要连接一些事件以便我们可以验证用户输入，并且我们需要一个新的 `ValidationMessageStore` 来存储任何验证错误。

创建一个新的 `ValidatorType` 实例就像指示我们的 `ServiceProvider` 检索一个实例一样简单。

```
private void ValidatorTypeChanged()
{
  Validator = (IValidator)ServiceProvider.GetService(ValidatorType);
}
```

为此，我们必须在应用程序的 `Startup.ConfigureServices` 方法中注册验证器——一旦我们有了验证器和要验证的东西，我们就会这样做。

每当 `EditContext` 发生变化时，我们都需要一个新的 `ValidationMessagesStore` 来存储我们的验证错误消息。

```
void EditContextChanged()
{
  ValidationMessageStore = new ValidationMessageStore(EditContext);
  HookUpEditContextEvents();
}
```


我们还需要连接一些事件，以便我们可以验证用户输入并将错误添加到我们的 `ValidationMessageStore`。

```
private void HookUpEditContextEvents()
{
  EditContext.OnValidationRequested += ValidationRequested;
  EditContext.OnFieldChanged += FieldChanged;
}
```

- OnValidationRequested

  当需要验证 `EditContext.Model` 的所有属性时会触发此事件。当用户尝试发布 `EditForm` 以便 Blazor 可以确定输入是否有效时，会发生这种情况。

- OnFieldChanged

  每当用户通过在 Blazor 的 `InputBase<T>` 派生组件之一中编辑 `EditContext.Model` 的属性值来更改它时，都会触发此事件。

```
async void ValidationRequested(object sender, ValidationRequestedEventArgs args)
{
  ValidationMessageStore.Clear();
  var validationContext =
    new ValidationContext<object>(EditContext.Model);
  ValidationResult result =
    await Validator.ValidateAsync(validationContext);
  AddValidationResult(EditContext.Model, result);
}
```

- 第 3 行

  首先，我们从任何先前的验证中清除所有错误消息。


- 第 4 行

  接下来，我们指示 `FluentValidation.IValidator` 验证在 `EditForm`（我们通过 `EditContext.Model` 访问）中正在编辑的模型。

- 第 5 行

  最后，我们将任何验证错误添加到我们的 `ValidationMessageStore`，这是在一个单独的方法中完成的，因为我们将在验证整个对象时使用它，并且在通过 `EditContext.OnFieldChanged` 通知时验证单个更改的属性时也会使用它。

将错误消息添加到 `ValidationMessageStore` 只是创建一个 `FieldIdentifier` 以准确识别哪个对象/属性有错误并使用该标识符添加任何错误消息，然后让 `EditContext` 知道验证状态已更改的情况。

请注意，当验证涉及长时间运行的异步调用（例如，对 WebApi 以检查 `UserName` 可用性）时，我们可以更新验证错误并多次调用 `EditContext.NotifyValidationStateChanged` 以在用户界面中提供验证状态的增量显示。


```
void AddValidationResult(object model, ValidationResult validationResult)
{
  foreach (ValidationFailure error in validationResult.Errors)
  {
    var fieldIdentifier = new FieldIdentifier(model, error.PropertyName);
    ValidationMessageStore.Add(fieldIdentifier, error.ErrorMessage);
  }
  EditContext.NotifyValidationStateChanged();
}
```

最后，当用户在表单输入控件中编辑值时，我们需要验证单个对象/属性。当发生这种情况时，我们会通过 `EditContext.OnFieldChanged` 事件得到通知。除了前两行和最后一行，以下代码是 FluentValidator 特定的。

```
async void FieldChanged(object sender, FieldChangedEventArgs args)
{
  FieldIdentifier fieldIdentifier = args.FieldIdentifier;
  ValidationMessageStore.Clear(fieldIdentifier);

  var propertiesToValidate = new string[] { fieldIdentifier.FieldName };
  var fluentValidationContext =
    new ValidationContext<object>(
      instanceToValidate: fieldIdentifier.Model,
      propertyChain: new FluentValidation.Internal.PropertyChain(),
      validatorSelector: new FluentValidation.Internal.MemberNameValidatorSelector(propertiesToValidate)
    );

  ValidationResult result = await Validator.ValidateAsync(fluentValidationContext);

  AddValidationResult(fieldIdentifier.Model, result);
}
```

- 第 3-4 行

  从事件 `args` 中获取 `FieldIdentifier`（ObjectInstance/PropertyName 对），并仅清除该属性的所有先前错误消息。

- 第 16 行

  使用 `ValidationRequested` 使用的相同方法将来自 FluentValidation 的错误添加到我们的 `ValidationMessageStore`。

## 使用组件
首先创建一个模型供我们的用户编辑。

```
namespace CustomValidation.Models
{
  public class Person
  {
    public string Name { get; set; }
    public int Age { get; set; }
  }
}
```

接下来，使用 FluentValidation 为 `Person` 创建一个验证器。

```
using CustomValidation.Models;
using FluentValidation;

namespace CustomValidation.Validators
{
  public class PersonValidator : AbstractValidator<Person>
  {
    public PersonValidator()
    {
      RuleFor(x => x.Name).NotEmpty();
      RuleFor(x => x.Age).InclusiveBetween(18, 80);
    }
  }
}
```

因为我们的验证组件使用 `IServiceProvider` 来创建验证器的实例，所以我们需要在 `Startup.ConfigureServices` 中注册它。

```
public void ConfigureServices(IServiceCollection services)
{
  services.AddScoped<Validators.PersonValidator>();
}
```

最后，我们需要设置我们的用户界面来编辑我们的 `Person` 类的一个实例。

```
@page "/"
@using Models

<EditForm Model=@Person OnValidSubmit=@ValidFormSubmitted>
  <FluentValidationValidator ValidatorType=typeof(Validators.PersonValidator)/>
  <p>Validation summary</p>
  <ValidationSummary />
  <p>Edit object</p>
  <div class="form-group">
    <label for="Name">Name</label>
    <InputText @bind-Value=Person.Name class="form-control" id="Name" />
    <ValidationMessage For="() => Person.Name" />
  </div>
  <div class="form-group">
    <label for="Age">Age</label>
    <InputNumber @bind-Value=Person.Age class="form-control" id="Age" />
    <ValidationMessage For=@(() => Person.Age) />
  </div>
  <input type="submit" class="btn btn-primary" value="Save" />
</EditForm>

@code {
  Person Person = new Person();

  void ValidFormSubmitted()
  {
    Person = new Person();
  }
}
```

## 执行流程
### 页面显示

1. 我们的 `EditForm` 组件是从 `<EditForm Model=@Person>` 标记创建的。
2. `EditForm.OnParametersSet` 被执行，因为 `EditForm.Model` 已经从 null 变成了我们的 `Person`，它创建了一个新的 `EditContext` 实例。
3. 新的 `EditContext` 实例通过[级联值](https://feiyun0112.github.io/blazor-university.zh-cn/cascading-values/cascading-values-by-type/)级联到所有子组件。
4. 对于这种级联值的变化，`InputBase<T>` 的每个派生类都会执行其 `SetParametersAsync`，并通过创建 `FieldIdentifier` 的新实例来做出反应。

### 我们的验证组件已初始化

1. 我们的验证组件的 `SetParametersAsync` 方法是通过引用新的 `EditContext` 来执行的。
2. 我们的组件创建了一个新的 `ValidationMessageStore`。
3. 我们的组件侦听 `EditContext` 上的事件以获取验证请求和输入更改通知。

### 用户更改数据
1. 用户在 `InputBase<T>` 派生类中编辑数据。
2. 组件通过 `EditContext.NotifyFieldChanged` 通知此状态更改（从未修改到已修改），并传递其 `FieldIdentifier`。
3. `EditContext` 触发其 `OnFieldChanged`，传递 `FieldIdentifier`。
4. 我们组件的事件订阅告诉我们的 `ValidationMessageStore` 清除由 `FieldIdentifier` 的 `Model` 和 `FieldName` 属性标识的状态的所有错误消息。
5. 我们的组件对单个属性执行其自定义验证。
6. 验证错误被添加到我们组件的 `ValidationMessageStore` 中，由 `FieldIdentifier` 作为键。
7. `ValidationMessageStore` 执行 `EditContext.GetFieldState` 以检索当前 `FieldIdentifier` 的 `FieldState`。
8. `ValidationMessageStore` 被添加到 `FieldState`，以便 `FieldState.GetValidationMessages` 能够从所有 `ValidationMessageStore` 实例中检索所有错误消息。

第 8 步特别重要，因为 Blazor 需要能够检索特定输入的所有验证错误消息，无论它们被添加到哪个 `ValidationMessageStore`。

### 用户提交表单
1. `<EditForm>` 执行 `EditContext.Validate`。
2. `EditContext` 触发其 `OnValidationRequested` 事件。
3. 我们组件的事件订阅告诉我们的 ·ValidationMessageStore· 清除所有字段的所有先前验证错误消息。
4. 我们的组件对整个 `EditContext.Model` 对象执行其自定义验证。
5. 与对单个更改的验证一样，错误被添加到 ·ValidationMessageStore·，它使用 ·EditContext· 中的所有相关 ·FieldState· 实例注册自身。
6. `<EditForm>` 根据是否有错误消息触发相关的有效/无效事件。

### EditForm.Model 已更改
如果这是一个用于创建 `people` 的用户界面，那么在成功将我们的新 `People` 提交到服务器后，我们的应用程序可能会创建一个新 `People` 供我们的表单进行编辑。这将丢弃与前一个  `People` 实例相关的所有状态（在虚线框中表示），并从新实例重新开始。

- EditContext
- FieldState
- ValidationMessageStore

![](ValidationMessageStoresUML2.png)

在演示代码中添加了一些日志记录，我们会看到以下输出。

```
WASM：编辑上下文已更改
WASM：创建了新的 ValidationMessageStore
WASM：连接 EditContext 事件（OnValidationRequested 和 OnFieldChanged）
WASM：触发 OnFieldChanged：验证类 Person 上名为 Name 的单个属性
WASM：触发 OnFieldChanged：验证 Person 类上名为 Age 的单个属性
WASM：OnValidationRequested 触发：验证整个对象
WASM：EditContext 已更改
WASM：创建了新的 ValidationMessageStore
WASM：连接 EditContext 事件（OnValidationRequested 和 OnFieldChanged）
```

- 第 1-3 行

  创建相关状态实例以支持编辑 `Person` 实例。

- 第 4 行

  输入了一个名字

- 第 5 行

  输入年龄

- 第 6 行

  用户提交表单

- 第 7 行

  Index.razor 中的 `Person` 实例被更改，导致元状态实例被丢弃并为新的 `EditForm.Model` 创建新实例。

**[下一篇 - 组件库](https://feiyun0112.github.io/blazor-university.zh-cn/component-libraries/)**