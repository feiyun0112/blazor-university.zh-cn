> 原文链接：https://blazor-university.com/dependency-injection/dependency-lifetimes-and-scopes/singleton-dependencies/

# Singleton 依赖
Singleton 依赖是一个由依赖它的每个对象共享的单个对象实例。在 WebAssembly 应用程序中，这是在浏览器的当前选项卡中运行的当前应用程序的生命周期。当类没有状态或（在服务器端应用程序中）具有可以在连接到同一服务器的所有用户之间共享的状态时，将依赖项注册为 Singleton 是可以接受的；Singleton 依赖项必须是线程安全的。

为了说明这种共享状态，让我们创建一个非常简单（即不可扩展）的聊天应用程序。

## Singleton 聊天服务

[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/DependencyInjection/WebChat)

首先，创建一个新的 Blazor 服务器应用。然后新建一个名为 **Services** 的文件夹，添加如下界面。这是我们的 UI 将用来向其他用户发送消息的服务，每当用户发送消息时都会收到通知，并且当我们的用户首次连接时，他们将能够看到迄今为止有限的聊天历史记录。因为这是在 Blazor 服务器端应用程序上运行的 Singleton 依赖项，所以它将由同一服务器上的所有用户共享。

```
public interface IChatService
{
  bool SendMessage(string username, string message);
  string ChatWindowText { get; }
  event EventHandler TextAdded;
}
```

为了实现这项服务，我们将使用 `List<string>` 来存储聊天历史记录，并在队列中超过 50 条时从列表的开头删除消息。我们将使用 `lock()` 语句来确保线程安全。

```
public class ChatService : IChatService
{
  public event EventHandler TextAdded;
  public string ChatWindowText { get; private set; }

  private readonly object SyncRoot = new object();
  private List<string> ChatHistory = new List<string>();

  public bool SendMessage(string username, string message)
  {
    if (string.IsNullOrWhiteSpace(username) || string.IsNullOrWhiteSpace(message))
      return false;

    string line = $"<{username}> {message}";

    lock (SyncRoot)
    {
      ChatHistory.Add(line);
      while (ChatHistory.Count > 50)
        ChatHistory.RemoveAt(0);

      ChatWindowText = string.Join("\r\n", ChatHistory.Take(50));
    }

    TextAdded?.Invoke(this, EventArgs.Empty);
    return true;
  }
}
```

- 第 3 行

  每当有新消息发布到我们的聊天服务器时，我们的 UI 可以挂钩的事件。

- 第 4 行

  代表最多 50 行聊天记录的字符串。

- 第 16-23 行

  锁定 `SyncRoot` 以防止并发问题，将当前行添加到聊天历史记录，如果超过 50 行，则删除最旧的历史记录，然后重新创建 `ChatWindowText` 属性的内容。

- 第 25 行

  通知聊天服务的所有使用者 `ChatWindowText` 已更新。

要注册服务，请打开 `Startup.cs` 并在 `ConfigureServices` 中添加以下内容

```
services.AddSingleton<IChatService, ChatService>();
```

## 定义用户界面
为了将 C# 聊天代码与显示标记分开，我们将使用代码隐藏方法。在 **Pages** 文件夹中创建一个名为 **Index.razor.cs** 的新文件，Visual Studio 应自动将其嵌入到 **Index.Razor** 文件下方。然后我们需要将我们的新索引类标记为部分。

```
public partial class Index
{
}
```

那么需要我们的组件类来做以下事情

1. 初始化后，订阅 `ChatService.TextAdded`。
2. 为了避免我们的 Singleton 持有已处置对象的引用，当我们的组件被处置时，我们应该取消订阅 `ChatService.TextAdded`。
3. 当 `ChatService.TextAdded` 被触发时，我们应该更新用户界面以显示新的 `IChatService.ChatWindowText` 内容。
4. 我们应该允许用户输入他们的名字+一些文本发送给其他用户。

让我们从最简单的步骤开始，也就是第 4 步，然后按照列出的顺序实现其他要求。

为简单起见，我们将向当前类添加 `Name` 和 `Text` 属性，而不是创建视图模型，我们还将使用 `RequiredAttribute` 装饰它们，以便在用户尝试发布文本而不填写所需输入时向他们提供反馈.

```
public partial class Index
{
  [Required(ErrorMessage = "Enter name")]
  public string Name { get; set; }
  [Required(ErrorMessage = "Enter a message")]
  public string Text { get; set; }
}
```

## 初始标记和验证
我们将替换 **Index.razor** 的内容并将其替换为一个简单的 `EditForm`，该 `EditForm` 由一个 `DataAnnotationsValidator` 组件和一些用于输入用户名和文本的 Bootstrap CSS 修饰的 HTML 组成。

```
@page "/"
<h1>Blazor web chat</h1>

<EditForm Model=@this>
  <DataAnnotationsValidator/>
  <div class="row mt-1">
    <div class="col-3">
      <InputText class="form-control" placeholder="Name" @bind-Value=Name maxlength=20/>
      <ValidationMessage For=@( () => Name )/>
    </div>
    <div class="col-9">
      <div class="input-group">
        <InputText class="form-control" placeholder="..." @bind-Value=Text maxlength=100 />
        <div class="input-group-append">
          <button class="btn btn-primary" type=submit>Send</button>
        </div>
      </div>
      <ValidationMessage For=@( () => Text )/>
    </div>
  </div>
</EditForm>
```

- 第 4 行

  创建一个绑定到 this 的 EditForm。

- 第 5 行

  启用基于数据注释（如`RequiredAttribute`）的验证。

- 第 8 行

  将 Blazor `InputText` 组件绑定到 `Name` 属性。

- 第 9 行

  显示 `Name` 属性的任何验证错误。

- 第 13 行

  将 Blazor `InputText` 组件绑定到 `Text` 属性。

- 第 18 行

  显示 `Text` 属性的任何验证错误。

## 使用 IChatService
接下来，我们将注入 `IChatService` 并将其完全连接到我们的组件。为此，我们需要执行以下操作。

```
public partial class Index : IDisposable
{
  [Required(ErrorMessage = "Enter name")]
  public string Name { get; set; }
  [Required(ErrorMessage = "Enter a message")]
  public string Text { get; set; }

  [Inject]
  private IChatService ChatService { get; set; }

  private string ChatWindowText => ChatService.ChatWindowText;

  protected override void OnInitialized()
  {
    base.OnInitialized();
    ChatService.TextAdded += TextAdded;
  }

  private void SendMessage()
  {
    if (ChatService.SendMessage(Name, Text))
      Text = "";
  }

  private void TextAdded(object sender, EventArgs e)
  {
    InvokeAsync(StateHasChanged);
  }

  void IDisposable.Dispose()
  {
    ChatService.TextAdded -= TextAdded;
  }
}
```

- 第 8-9 行

  声明应自动注入的对 `IChatService` 的依赖项。

- 第 11 行

  声明一个使访问 `IChatService.ChatWindowText` 变得简单的属性。

- 第 16 行

  订阅 `IChatService.TextAdded` 事件。

- 第 21 行

  将当前用户的输入发送到聊天服务。

- 第 27 行

  每次调用 `IChatService.TextAdded` 时刷新用户界面。

- 第 32 行

  当组件被释放时，取消订阅 `IChatService.TextAdded` 以避免内存泄漏。

注意：我们必须将 `StateHasChanged` 调用包装在对 `InvokeAsync` 的调用中。这是因为 `IChatService.TextAdded` 事件将由添加文本的任何用户触发，因此将由各种线程触发。我们需要 Blazor 使用 `InvokeAsync` 编组这些调用，以确保我们组件上的所有线程调用都按顺序执行。

## 将聊天窗口添加到我们的用户界面
我们现在只需要在我们的标记中添加一个 HTML `<textarea>` 控件并将它绑定到我们的 `ChatWindowText` 属性，并确保在提交 `EditForm` 时没有验证错误，它会调用我们的 `SendMessage` 方法。

最终的用户界面标记如下所示。

```
@page "/"

<h1>Blazor web chat</h1>

<EditForm Model=@this OnValidSubmit=@SendMessage>
  <DataAnnotationsValidator/>
  <div class="row">
    <textarea class="form-control" rows=20 readonly>@ChatWindowText</textarea>
  </div>
  <div class="row mt-1">
    <div class="col-3">
      <InputText class="form-control" placeholder="Name" @bind-Value=Name maxlength=20/>
      <ValidationMessage For=@( () => Name )/>
    </div>
    <div class="col-9">
      <div class="input-group">
        <InputText class="form-control" placeholder="..." @bind-Value=Text maxlength=100 />
        <div class="input-group-append">
          <button class="btn btn-primary" type=submit>Send</button>
        </div>
      </div>
      <ValidationMessage For=@( () => Text )/>
    </div>
  </div>
</EditForm>
```

- 第 5 行

  当用户在 `InputText` 上按 `enter` 并且输入验证通过时调用 `SendMessage`。

- 第 7-9 行

  添加 HTML `<textarea>` 并将其绑定到 `WindowChatText`。

## WebAssembly 应用程序中的 Singleton 依赖项
如果 Blazor 应用程序是 Blazor 服务器端应用程序，则上述应用程序将仅允许用户相互聊天。

这是因为 Singleton 依赖项是每个应用程序进程共享的。 Blazor 服务器端应用程序实际上在服务器上运行，因此单例实例在同一服务器应用程序进程中运行的多个用户之间共享。

在 WebAssembly 应用程序中运行时，每个浏览器选项卡都是它自己独立的应用程序进程，因此如果用户在浏览器中运行单独的进程（WebAssembly 托管应用程序），用户将无法相互聊天，因为他们不共享任何公共状态。

使用多个服务器时也是如此。一旦我们的聊天服务流行到足以保证一个或多个额外的服务器，就不再有所有用户的全局共享状态，只有每个服务器的共享状态。

一旦我们需要扩展我们的服务器，或者我们希望将我们的聊天客户端实现为 WebAssembly 应用程序以从我们的服务器中移除一些工作负载，我们就需要设置一种更强大的共享状态的方法。这不在本节的范围内，因为本节的目的只是演示注册为单例的依赖项如何在单个应用程序进程中共享。

## 读者任务
浏览器不可能有足够的垂直空间同时显示 50 条聊天消息，因此用户必须手动滚动聊天区域才能看到最新消息。

为了改善用户体验，我们的组件应该在每次添加新文本时真正将 `<textarea>` 滚动条滚动到底部。如果您不想自己解决这个问题，那么只需看看本节随附的项目，工作已经为您完成。如果您确实想解决它，这里有一些线索。

1. 您将编写一些将控件作为参数并设置 `control.scrollTop = control.scrollHeight` 的 JavaScript。
2. 每次我们的组件呈现后，您都需要[调用此 JavaScript](/javascript-interop/calling-javascript-from-dotnet/)。
3. 您需要将 `<textarea>` 的 [ElementReference](/javascript-interop/calling-javascript-from-dotnet/passing-html-element-references/) 传递给 JavaScript。

**[下一篇 - Scoped 依赖](/dependency-injection/dependency-lifetimes-and-scopes/scoped-dependencies/)**