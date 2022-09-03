> 原文链接：https://blazor-university.com/forms/

# 表单
[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/Forms/BasicForm)

`EditForm` 组件是 Blazor 管理用户输入的方法，这种方法可以轻松地对用户输入执行验证。它还提供了检查是否满足所有验证规则的能力，如果没有满足，则向用户显示验证错误。

虽然可以使用标准的 `<form>` HTML 元素创建表单，但我建议使用 `EditForm` 组件，因为它为我们提供了额外的功能。

**注意：** 如果您还没有这样做，我建议您阅读[双向绑定指令](https://feiyun0112.github.io/blazor-university.zh-cn/components/two-way-binding/binding-directives/)部分。

## 表单模型
`EditForm` 的关键特性是它的模型参数。该参数为组件提供了一个上下文，它可以使用它来启用用户界面绑定并确定用户的输入是否有效。

让我们从创建一个可以与我们的 `EditForm` 一起使用的类开始。此时，一个简单的空类就足够了。

```
public class Person
{
}
```
编辑标准 `index.razor` 页面如下：

```
@page "/"

<EditForm Model=@Person>
  <input type="submit" value="Submit" class="btn btn-primary"/>
</EditForm>

@code
{
  Person Person = new Person();
}
```

第 9 行创建了一个 `Person` 实例，供我们的表单绑定。第 3 行创建一个 `EditForm` 并将其 `Model` 参数设置为我们的实例。前面的 rzaor 标记会生成以下 HTML。

```
<form>
  <input class="btn btn-primary" type="submit" value="Submit">
</form>
```

## 检测表单提交
当用户单击上例中的提交按钮时，`EditForm` 将触发其 `OnSubmit` 事件。我们可以在代码中使用这个事件来处理任何业务逻辑。

```
@page "/"

<h1>Status: @Status</h1>
<EditForm Model=@Person OnSubmit=@FormSubmitted>
  <input type="submit" value="Submit" class="btn btn-primary"/>
</EditForm>

@code
{
  string Status = "Not submitted";
  Person Person = new Person();

  void FormSubmitted()
  {
    Status = "Form submitted";
    // Post data to the server, etc
  }
}
```
表单提交将在[处理表单提交](https://feiyun0112.github.io/blazor-university.zh-cn/forms/handling-form-submission/)部分更深入地介绍。


**[下一篇 - 编辑表单数据](https://feiyun0112.github.io/blazor-university.zh-cn/forms/editing-form-data/)**