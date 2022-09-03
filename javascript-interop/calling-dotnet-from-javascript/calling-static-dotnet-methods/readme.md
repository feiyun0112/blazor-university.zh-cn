> 原文链接：https://blazor-university.com/javascript-interop/calling-dotnet-from-javascript/calling-static-dotnet-methods/

# 调用静态 .NET 方法
除了在 .NET 对象实例上调用方法外，Blazor 还使我们能够调用静态方法。下一个示例将展示如何从 JavaScript 调用 .NET 并检索 API 调用可能需要的特定设置，例如 `Google Analytics`。

从服务器设置中读取 JavaScript 设置的好处是，在部署过程中可以根据环境（开发/QA/生产）覆盖这些值，而无需更改 JavaScript 文件。

[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/JavaScriptInterop/CallingStaticDotNetMethods)


- 创建新的 Blazor 服务器端应用程序
- 打开 **/appsettings.json** 文件并添加一个名为“JavaScript”的部分
```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "JavaScript": {
    "SomeApiKey":  "123456789"
  },
  "AllowedHosts": "*"
}
```

- 接下来我们需要一个类来保存这个设置，创建一个名为 `Configuration` 的文件夹
- 在该文件夹中，创建一个名为 `JavaScriptSettings.cs` 的文件
```
public class JavaScriptSettings
{
  public string SomeApiKey { get; set; }
}
```
- 编辑 **/Startup.cs** 文件
- 在该类的构造函数中，我们将使用注入的 `IConfiguration` 实例从 `/appsettings.json` 中获取“JavaScript”部分并将其存储在静态引用中。
```
public Startup(IConfiguration configuration)
{
  Configuration = configuration;
  var javaScriptSettings = configuration
    .GetSection("JavaScript")
    .Get<JavaScriptSettings>();
  JavaScriptConfiguration.SetSettings(javaScriptSettings);
}
```
- `JavaScriptConfiguration` 类还不存在，所以接下来我们将在 **Configuration** 文件夹中创建它。
```
public static class JavaScriptConfiguration
{
  private static JavaScriptSettings Settings;

  internal static void SetSettings(JavaScriptSettings settings)
  {
    Settings = settings;
  }

  public static JavaScriptSettings GetSettings() => Settings;
}
```

现在我们的配置文件中有一些新设置，一个在 .NET 中表示这些设置的类，我们正在读取这些值并将它们存储在静态引用中。接下来我们需要从 JavaScript 访问它。

- 编辑 /Pages/_Host.cshtml 文件并在现有 `<script>` 标记下添加以下内容
```
<script src="~/scripts/CallingStaticDotNetMethods.js"></script>
```

- 接下来，在 **/wwwroot** 文件夹下创建一个名为 **scripts** 的文件夹

- 在该文件夹中，创建一个名为 `CallingDotNetStaticMethods.js` 的新文件并添加以下脚本
```
setTimeout(async function () {
  const settings = await DotNet.invokeMethodAsync("CallingStaticDotNetMethods", "GetSettings");
  alert('API key: ' + settings.someApiKey);
}, 1000);
```

`DotNet.invokeMethodAsync `至少需要两个参数。可以传递两个以上，并且第二个之后的任何参数都被认为是作为参数传递给方法的值。

1. 方法所在的二进制文件的全名（不包括文件扩展名）
2. 要执行的方法的标识符

最后一块拼图是用 `[JSInvokable]` 属性装饰方法，传入标识符——在本示例中，标识符将是 `GetSettings`。

编辑 **/Configuration/JavaScriptConfiguration** 类，并更改 `GetSettings` 方法：

```
[JSInvokable("GetSettings")]
public static JavaScriptSettings GetSettings() => Settings;
```

传递给 `[JSInvokable]` 的标识符不必与方法名称相同。

## JavaScript 可调用方法的条件
要成为可通过这种方式调用的候选 .NET 方法，该方法必须满足以下条件：

- 拥有该方法的类必须是公共的
-方法必须是公开的
- 必须是静态方法
- 返回类型必须为 `void`，或可序列化为 JSON——或者它必须是 `Task` 或 `Task<T>`，其中 `T` 可序列化为 JSON
- 所有参数必须可序列化为 JSON
- 该方法必须用 `[JSInvokable]` 装饰
- `JSInvokable` 属性中使用的同一标识符不能在单个程序集中多次使用。

**注意：不要立即从 JavaScript 调用 .NET 静态方法** 

如果您回顾 [JavaScript 启动过程](/javascript-interop/javascript-boot-process/)部分，您会记得 JavaScript 在 Blazor 初始化之前已在浏览器中初始化。

![](JavaScriptBootProcessDiagram.png)

正是出于这个原因，我们只在初始超时后调用 .NET 静态方法——在这种情况下，我选择了一秒。

```
setTimeout(async function () {
  const settings = await DotNet.invokeMethodAsync("CallingStaticDotNetMethods", "GetSettings");
  alert('API key: ' + settings.someApiKey);
}, 1000);
```
在撰写本文时，无法从 JavaScript 检查 Blazor 是否已准备好被调用，而无需尝试调用它并失败。

```
window.someInitialization = async function () {
  try {
    const settings = await DotNet.invokeMethodAsync("CallingStaticDotNetMethods", "GetSettings");
    alert('API key: ' + settings.someApiKey);
  }
  catch {
    // Try again
    this.setTimeout(someInitialization, 10);
  }
}
window.someInitialization();
```

## 连接到 Blazor.start
可以在通过调用 `Blazor.start` 函数初始化 Blazor 时调用我们的 JavaScript。

首先，编辑 **/Pages/_Host.cshtml** 并更改引用 Blazor 脚本的 `<script>` 标记，并添加一个名为 `autostart` 且值为 `false` 的新属性。

```
<script src="_framework/blazor.server.js" autostart="false"></script>
```

接下来，我们需要更改我们的 JavaScript 以便它调用 `Blazor.start` - 这将返回一个 `Promise<void>`，一旦 Blazor 初始化，我们就可以使用它来执行我们自己的代码。

```
Blazor.start({})
  .then(async function () {
    const settings = await DotNet.invokeMethodAsync("CallingStaticDotNetMethods", "GetSettings");
    alert('API key: ' + settings.someApiKey);
  });
```

这种方法的问题是您只能使用一次。因此，如果我们在不同的脚本中有多个入口点，那么我们将不得不创建自己的挂钩点来缓存来自 `Blazor.start` 的结果并将其返回给任何调用脚本。

**[下一篇 - 依赖注入](/dependency-injection/)**