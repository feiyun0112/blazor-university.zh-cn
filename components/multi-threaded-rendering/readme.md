> 原文链接：https://blazor-university.com/components/multi-threaded-rendering/

# 多线程渲染
由于 Blazor Server 应用程序中可用的线程不止一个，因此完全有可能不同的组件可以让不同的线程在其上执行代码。

这在基于异步任务的操作中最常见。例如，向服务器发送 HTTP 请求的多个组件将收到单独的响应。每个单独的响应都将使用系统从可用线程池中为我们选择的任何线程来恢复调用方法。

我们观察这种行为的最简单方法是创建一些执行 `await` 的异步方法。对于此示例，我们将使用 OnInitializedAsync [生命周期](https://feiyun0112.github.io/blazor-university.zh-cn/components/component-lifecycles/)方法。

[源代码](https://github.com/mrpmorris/blazor-university/tree/master/src/Components/MultithreadedRendering)

为了演示这一点，我们首先需要创建一个新的 Blazor Server 应用程序。然后，在 **/Shared** 文件夹中，创建一个名为 `SynchronousInitComponent` 的组件。该组件会在执行 `OnInitialized` 时捕获当前线程的 `Thread.ManagedThreadId`。当我们的组件渲染时，该值将显示在页面上。

```
<p>Sync rendered by thread @IdOfRenderingThread</p>

@code
{
  int IdOfRenderingThread;

  protected override void OnInitialized()
  {
    base.OnInitialized();
    IdOfRenderingThread =
      System.Threading.Thread.CurrentThread.ManagedThreadId;
  }
}
```

- 第 5 行

  声明一个字段以保存对线程 ID 的引用。

- 第 7 行

  OnInitialized 生命周期方法被重写。

- 第 10 行

  当前线程的 ID 存储在 `IdOfRenderingThread` 中，因此可以渲染。

- 第 1 行

  呈现在第 10 行捕获的线程的 ID。

最后，编辑 **/Pages/Index.razor** 页面以显示我们新组件的 5 个实例。

```
@page "/"

<h1>Components with synchronous OnInitialized()</h1>
@for (int i = 0; i < 5; i++)
{
  <SynchronousInitComponent />
}
```

运行应用程序将为每个组件显示相同的线程 ID。显然，您的线程 ID 可能与我的不同。

```
Components with synchronous OnInitialized()
Sync rendered by thread 4
Sync rendered by thread 4
Sync rendered by thread 4
Sync rendered by thread 4
Sync rendered by thread 4
```

## 异步
接下来，我们将在名为 `AsynchronousInitComponent` 的 **/Shared** 文件夹中创建另一个新组件。此组件将与 `SynchronousInitComponent` 相同，但会在等待 1 秒后在 `OnInitializedAsync` 中额外重新分配 `IdOfRenderingThread` 的值。

```
<p>Async rendered by thread @IdOfRenderingThread</p>

@code
{
  int IdOfRenderingThread;

  protected override async Task OnInitializedAsync()
  {
    // Runs synchronously as there is no code in base.OnInitialized(),
    // so the same thread is used
    await base.OnInitializedAsync().ConfigureAwait(false);
    IdOfRenderingThread =
      System.Threading.Thread.CurrentThread.ManagedThreadId;

    // Awaiting will schedule a job for later, and we will be assigned
    // whichever worker thread is next available
    await Task.Delay(1000).ConfigureAwait(false);
    IdOfRenderingThread =
      System.Threading.Thread.CurrentThread.ManagedThreadId;
  }
}
```

- 第 7 行

  OnInitializedAsync 生命周期方法被重写。

- 第 12 行

  与同步组件一样，当前线程的 ·ManagedThreadId· 被分配给 ·IdOfRenderingThread·，因此它可以被组件渲染。 （见说明）

- 第 17 行

  在继续执行该方法之前，我们允许等待 1 秒。
  
- 第 18 行

  `IdOfRenderingThread` 再次更新，显示在第 17 行等待 1 秒后重新渲染组件的线程 ID。


**注意：** 第 11 行的 `await` 将异步运行似乎是有道理的。事实上，它是同步运行的。这是因为基本方法什么都不做。异步代码（例如 `Task.Delay`）没有等待，因此同一线程继续执行。

我们还需要另一个页面来呈现这个新组件。在 **/Pages** 中使用以下标记创建一个名为 **AsyncInitPage.razor** 的新页面。

```
@page "/async-init"

<h1>Components with asynchronous OnInitializedAsync()</h1>
@for (int i = 0; i < 5; i++)
{
  <AsynchronousInitComponent/>
}
```

运行应用程序并导航到第二个页面将产生与第一页非常相似的输出，其中每个组件都由单个线程渲染。

```
Components with asynchronous OnInitializedAsync()
Async rendered by thread 4
Async rendered by thread 4
Async rendered by thread 4
Async rendered by thread 4
Async rendered by thread 4
```

但是，1 秒后，每个组件的 `OnInitializedAsync` 方法中的 `await Task.Delay(1000)` 将在为浏览器渲染 HTML 之前完成并更新 `IdOfRenderingThread`。这一次，我们可以看到使用不同的线程来完成 `OnInitializedAsync` 方法。

```
Components with asynchronous OnInitializedAsync()
Async rendered by thread 7
Async rendered by thread 18
Async rendered by thread 10
Async rendered by thread 13
Async rendered by thread 11
```

## ConfigureAwait(true) 会怎么样？
在我们的 `await` 上指定 `ConfigureAwait(true)` 并不能保证我们将看到我们的所有组件都渲染在启动 `await` 的同一线程上。指定`ConfigureAwait(true)` 仍将导致用于回调的线程混合。

```
Components with asynchronous OnInitializedAsync()
Async rendered by thread 11
Async rendered by thread 11
Async rendered by thread 9
Async rendered by thread 13
Async rendered by thread 13
```

即使 `ConfigureAwait(true)` 确实保证我们可以在同一个线程上继续，这仍然不能确保我们的 UI 仅由单个线程渲染。可能由于多种原因导致组件重新渲染，包括（但不限于）。

- 来自 `System.Threading.Timer` 的回调
- 由多个用户共享的 `Singleton` 实例上的另一个线程触发的事件
- 来自我们通过 Web Socket 连接的另一台服务器的数据推送。

## 总结
在 Blazor Server 应用程序中，没有单个 UI 线程。当需要渲染工作时，可以使用任何可用的线程。

此外，如果任何方法在执行异步操作的代码上使用了 `await`，则分配用于继续处理该方法的线程很可能与启动它的线程不同。

在 Blazor WebAssembly 应用程序（只有一个线程）中没有线程问题，但在服务器端应用程序中，当跨多个组件使用非线程安全依赖项时，这可能会导致问题。

此问题将在有关 [OwningComponentBase<T>](https://feiyun0112.github.io/blazor-university.zh-cn/dependency-injection/component-scoped-dependencies/owningcomponentbase-generic/) 的部分中解决。

**[下一篇 - 线程安全的使用 InvokeAsync](https://feiyun0112.github.io/blazor-university.zh-cn/components/multi-threaded-rendering/invokeasync)**