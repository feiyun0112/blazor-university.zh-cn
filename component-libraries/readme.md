> 原文链接：https://blazor-university.com/component-libraries/

# 组件库
组件库使我们能够将组件和页面以及任何支持文件（例如 CSS 文件、JavaScript 和图像）打包到一个可重用的项目中。

创建一个名为 `ClassLibraryConsumer` 的新 Blazor 解决方案。右键单击解决方案并选择 **Add->New Project**，然后选择 **Razor Class Library** – 将其命名为 **BlazorUniversity.ClassLibrary**。

这将在名为 **BlazorUniversity.ClassLibrary** 的新文件夹中创建一个新的 Razor 类库，并创建一个具有相同名称的新 `csproj` 文件。将新库添加到当前解决方案中，然后从 ClassLibraryConsumer 项目中引用新库。

我们的新类库现在可以通过将其包含在解决方案中并引用它来从任意数量的项目中使用，或者可以将其推送到 NuGet.org 并作为 NuGet 包使用。

## 添加支持文件
为我们创建的默认项目有一个名为 `wwwroot` 的文件夹。这是我们希望放置我们库的使用者需要的任何支持文件的地方，例如 JavaScript 等。

## 访问使用的组件库中的资源
使用的组件库的 `wwwroot` 文件夹中的资源将自动与您的项目一起发布。要从使用的库中访问资源，我们需要使用以下 URL 格式。

`/_content/PackageId/MyImage.png`

- `_content` 是所有使用的组件库资源最终到达的路径的一部分。
- `PackageId` 是包含资源的二进制文件的包 ID。这是您在右键单击类库、选择属性并选择包选项卡时在包 ID 输入中看到的名称。如果您通过 NuGet 安装库，则它是您安装的包的名称。
- `MyImage.png` 是组件库的 `wwwroot` 文件夹中任何资源的名称。资源可以直接位于该文件夹中，或者路径可以标识任何级别的子文件夹中的资源，例如 `/_content/BlazorUniversity.ConsumedLibrary/scripts/HelloWorld.js`

请注意，我们组件库中的任何组件也应该使用相同格式引用资源。

## 使用组件库
使用组件库非常简单

- 将项目引用添加到库

或者，

- 添加对库的 NuGet 引用。

确保阅读库作者的任何注释，因为您可能需要将 CSS 和/或 JavaScript 引用添加到 HTML。

## 在客户端 Blazor 中引用使用的脚本
在客户端 Blazor 应用程序中，这通常涉及向我们项目的 `wwwroot/index.html` 文件添加 `<script> `引用。

## 在服务器端 Blazor 中引用使用的脚本
对于服务器端 Blazor 应用程序，它被添加到文件 `/Pages/_Host.cshtml` 中，并且通常在引用 `_framework/blazor.server.js` 或 `_framework/blazor.webassembly.js` 的现有 `<script>` 标记之前添加



**[下一篇 - JavaScript 互操作](/javascript-interop/)**