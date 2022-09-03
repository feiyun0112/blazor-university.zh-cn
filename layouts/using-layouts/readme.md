> 原文链接：https://blazor-university.com/layouts/using-layouts/

# 使用布局
## 为应用程序指定默认布局
指定布局的最通用方法是编辑 **/Pages/_Imports.razor** 文件并编辑单行代码以标识不同的布局。

```
@layout MainLayout
```

布局的名称是强类型的。如果存在指定名称的布局，Blazor 会高亮正确语法的代码，如果标识符不正确，编译会失败。

**注意：**当然，如果您只想更改现有布局的外观，您可以更改 **/Shared/MainLayout.razor** 文件。

## 为应用程序的某个区域指定默认模板
[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/Layouts/UsingALayout)

如果您的应用程序有单独的区域，例如“Admin”区域，则可以指定用于该区域内所有页面的默认布局，只需将它们分组到具有自己的 **_Imports.razor** 文件的子文件夹中。

创建一个新的 Blazor 客户端应用程序，然后更新导航菜单以包含指向我们将很快创建的新页面的链接。

- 打开 **/Shared/NavMenu.razor** 文件。
- 找到最后一个 `<li>` 元素，它应该包含一个 `<NavLink>` 元素。
- 复制 `<li>` 元素。
- 将 NavLink 的 `href` 属性更改为 `admin/users`。
- 将链接的文本更改为**管理员用户**。

接下来我们将创建一个非常基本的页面

1. 在解决方案资源管理器中展开 **/Pages** 节点。
2. 创建一个名为 **Admin** 的文件夹。
3. 在文件夹中创建一个名为 **AdminUsers.razor** 的新文件。

```
@page "/admin/users"
<h2>Users</h2
```

**注意：** 页面的 URL 不必反映文件夹结构。

现在运行应用程序，该应用程序将具有一个名为“管理员用户”的新菜单项。当您单击该菜单项时，它将显示一个非常基本的页面，其中仅显示“Users”。接下来，我们将为所有管理页面创建一个默认布局。

1. 在 **Admin** 文件夹中创建另一个名为 **_Imports.razor** 的新文件。
2. 输入以下代码。

```
@layout AdminLayout
```

此时，应用程序中没有名为 **AdminLayout** 的文件，因此您应该在 Visual Studio 中看到名称下方有一条红线，表示找不到该文件。您可以通过在 **/Shared** 文件夹中创建 **AdminLayout.razor** 来解决此问题。

```
@inherits LayoutComponentBase
<h1>Admin</h1>
@Body
```

如果您现在运行该应用程序并单击**管理员用户**链接，您将看到仅由 `<h1>` 和 `<h2>` 组成的简陋页面。我们将在[嵌套布局](/layouts/nested-layouts/)部分解决这个问题，但现在我们将使用它作为练习，了解如何从页面本身显式指定布局。

## 为单个页面显式指定布局
[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/Layouts/SpecifyingALayoutExplicitly)

到目前为止，我们已经看到可以在 **/Pages/_Imports.razor** 文件中指定默认布局。我们还看到，可以通过 **Blazor** 找到更接近其正在呈现的页面的更具体的 **_Imports.razor** 文件来覆盖此设置。指定要使用的模板的最后（也是最明确的）级别是使用 `@layout` 指令在页面本身中指定它。

```
@page "/admin/users"
@layout MainLayout
<h2>Users</h2>
```
再次运行应用程序并单击**管理员用户**链接现在将使用应用程序的标准布局显示基本页面。

**[下一篇 - 嵌套布局](/layouts/nested-layouts)**

 