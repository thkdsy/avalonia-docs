---
id: threading
title: 线程模型
description: 了解 Avalonia 的单线程 UI 模型如何与 Dispatcher、异步模式和后台任务协同工作。
doc-type: explanation
---

Avalonia 采用单线程 UI 模型。所有与 UI 的交互，包括读取或写入控件属性，都必须发生在 UI 线程上。这与 WPF、WinForms 以及大多数桌面 UI 框架使用的线程模型是一致的。

## UI 线程

当应用启动时，Avalonia 会创建一个 Dispatcher 来管理 UI 线程上的工作项。所有控件创建、布局、渲染和输入处理都发生在这个线程上。

如果你尝试从后台线程访问控件，Avalonia 会抛出 `InvalidOperationException`，并附带消息 `Call from invalid thread.`。

## 访问 Dispatcher

### Dispatcher.UIThread

`Dispatcher.UIThread` 属性可以让你在代码中的任意位置访问 UI 线程的 Dispatcher。你可以用它把工作从后台线程切换回 UI 线程。

### Post（发出即返回）

`Post` 会在 UI 线程上安排一个回调，并立即返回。当你不需要等待结果时，可以使用它：

```csharp
Dispatcher.UIThread.Post(() =>
{
    StatusText.Text = "Processing complete";
});
```

### InvokeAsync（等待结果）

`InvokeAsync` 会安排一个回调，并返回一个 `Task`；当回调执行完成时，该任务才会结束。当你需要等待结果，或确保操作已经完成时，可以使用它：

```csharp
var text = await Dispatcher.UIThread.InvokeAsync(() =>
{
    return SearchBox.Text;
});
```

`InvokeAsync` 会捕获调用线程的 `ExecutionContext`，并在回调执行时恢复它。这意味着 `AsyncLocal<T>` 的值、模拟身份以及区域性设置等上下文信息都会从调用方传递到被调度的回调中，这与 `Task.Run` 和 WPF 的 Dispatcher 行为一致。

### CheckAccess and VerifyAccess

在调度之前，你可以先检查自己是否已经位于 UI 线程：

```csharp
if (Dispatcher.UIThread.CheckAccess())
{
    // Already on the UI thread, update directly
    StatusText.Text = "Ready";
}
else
{
    // On a background thread, marshal to UI thread
    Dispatcher.UIThread.Post(() => StatusText.Text = "Ready");
}
```

如果从非 UI 线程调用，`VerifyAccess()` 会抛出 `InvalidOperationException`：

```csharp
Dispatcher.UIThread.VerifyAccess(); // Throws if not on UI thread
```

### AvaloniaObject.Dispatcher

每个 `AvaloniaObject` 都会捕获它创建时所在线程的 Dispatcher。当你在编写控件或类库，并希望它们在不同 Dispatcher 环境下都能正确工作时，可以使用这个属性：

```csharp
// Uses the object's own dispatcher rather than assuming UIThread
myControl.Dispatcher.Post(() => myControl.IsVisible = false);
```

对于大多数应用来说，`AvaloniaObject.Dispatcher` 和 `Dispatcher.UIThread` 返回的是同一个实例。只有当类库作者需要支持多个 Dispatcher 时，这个区别才会真正重要。

### Dispatcher.CurrentDispatcher

返回调用线程对应的 Dispatcher；如果该线程还没有 Dispatcher，则会创建一个：

```csharp
var dispatcher = Dispatcher.CurrentDispatcher;
```

### Dispatcher.FromThread

返回与指定线程关联的 Dispatcher；如果不存在，则返回 `null`。与 `CurrentDispatcher` 不同，这个方法不会创建新的 Dispatcher：

```csharp
Dispatcher? dispatcher = Dispatcher.FromThread(Thread.CurrentThread);
if (dispatcher is not null)
{
    dispatcher.Post(() => { /* work */ });
}
```

## Dispatcher 优先级

`Post` 和 `InvokeAsync` 都支持一个可选的 `DispatcherPriority` 参数，用于控制该工作项相对于其他已排队项目的执行时机：

```csharp
Dispatcher.UIThread.Post(
    () => StatusText.Text = "Updated",
    DispatcherPriority.Background);
```

常见优先级从高到低如下：

| 优先级 | 说明 |
|---|---|
| `Send` | 在其他异步操作之前处理。 |
| `Normal` | 以普通优先级处理。 |
| `Default` | 前台 Dispatcher 的最低优先级。 |
| `Render` | 以与渲染相同的优先级处理。 |
| `Loaded` | 在布局与渲染之后、输入之前处理。 |
| `Input` | 以与输入相同的优先级处理。 |
| `Background` | 在其他非空闲操作完成后处理。 |
| `ContextIdle` | 在后台操作完成后处理。 |
| `ApplicationIdle` | 当应用处于空闲状态时处理。 |
| `SystemIdle` | 当系统处于空闲状态时处理。 |

## 异步模式

### 带 UI 更新的后台工作

最常见的模式是在后台线程执行较重的计算任务，并在完成后更新 UI：

```csharp
private async void OnLoadClick(object? sender, RoutedEventArgs e)
{
    LoadButton.IsEnabled = false;
    StatusText.Text = "Loading...";

    // Heavy work runs on a thread pool thread
    var data = await Task.Run(() =>
    {
        return LoadLargeDataSet();
    });

    // Back on the UI thread automatically (thanks to SynchronizationContext)
    Items = new ObservableCollection<Item>(data);
    StatusText.Text = $"Loaded {data.Count} items";
    LoadButton.IsEnabled = true;
}
```

:::info
如果一个 `async` 方法最初是在 UI 线程上启动的，那么在其中 `await` 一个 `Task` 后，后续执行会自动回到 UI 线程。Avalonia 会设置一个 `SynchronizationContext` 来捕获 UI 线程上下文。
:::

### 进度汇报

对于长时间运行的操作，可以把进度回传到 UI：

```csharp
private async void OnProcessClick(object? sender, RoutedEventArgs e)
{
    var progress = new Progress<int>(percent =>
    {
        // This callback runs on the UI thread
        ProgressBar.Value = percent;
    });

    await Task.Run(() => ProcessData(progress));

    StatusText.Text = "Done";
}

private void ProcessData(IProgress<int> progress)
{
    for (int i = 0; i <= 100; i++)
    {
        Thread.Sleep(50); // Simulate work
        progress.Report(i);
    }
}
```

### 基于定时器的更新

使用 `DispatcherTimer` 来执行周期性的 UI 更新。定时器回调会运行在 UI 线程上：

```csharp
var timer = new DispatcherTimer
{
    Interval = TimeSpan.FromSeconds(1)
};

timer.Tick += (sender, e) =>
{
    // Runs on the UI thread
    ClockText.Text = DateTime.Now.ToString("HH:mm:ss");
};

timer.Start();
```

### 向 Dispatcher 让出执行权

`Dispatcher.Yield()` 会暂停当前异步方法，并将其后续部分重新排入 Dispatcher 队列，从而让尚未处理的输入、布局和渲染任务先执行，再恢复当前流程：

```csharp
private async Task ProcessItemsAsync(IList<Item> items)
{
    foreach (var item in items)
    {
        ProcessItem(item);

        // Let the UI thread handle pending events before continuing
        await Dispatcher.Yield();
    }
}
```

`Yield` 是一个作用于 `Dispatcher.UIThread` 的静态方法。你可以指定优先级来控制何时恢复执行：

```csharp
// Resume only when the dispatcher is idle
await Dispatcher.Yield(DispatcherPriority.ApplicationIdle);
```

如果你不想使用静态方式，或者正在操作某个特定的 Dispatcher 实例，可以使用 `Resume`：

```csharp
await myControl.Dispatcher.Resume(DispatcherPriority.Background);
```

## 完整示例

下面的示例展示了如何从工作线程访问 UI 线程，以更新或读取 `TextBlock` 的文本：

```xml title='MainView.axaml'
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:vm="clr-namespace:AvaloniaApplication1.ViewModels"
             x:Class="AvaloniaApplication1.Views.MainView"
             x:DataType="vm:MainViewModel">
    <StackPanel Margin="20">
        <TextBlock Name="TextBlock1" />
    </StackPanel>
</UserControl>
```

```csharp title='MainView.axaml.cs'
using Avalonia.Controls;
using Avalonia.Threading;
using System.Threading.Tasks;

namespace AvaloniaApplication1.Views;

public partial class MainView : UserControl
{
    public MainView()
    {
        InitializeComponent();
        _ = Task.Run(() => OnTextFromAnotherThread("test"));
    }

    private void SetText(string text) => TextBlock1.Text = text;
    private string GetText() => TextBlock1.Text ?? "";

    private async void OnTextFromAnotherThread(string text)
    {
        // Start the job on the UI thread and return immediately.
        Dispatcher.UIThread.Post(() => SetText(text));

        // Start the job on the UI thread and wait for the result.
        var result = await Dispatcher.UIThread.InvokeAsync(GetText);

        // This would throw because we are on a worker thread:
        // SetText(text); // InvalidOperationException: 'Call from invalid thread'
    }
}
```

## 常见错误

### 在 Task.Run 中访问控件

```csharp
// WRONG: Accessing UI from background thread
await Task.Run(() =>
{
    StatusText.Text = "Done"; // Throws InvalidOperationException
});

// CORRECT: Update UI after awaiting the background work
var result = await Task.Run(() => ComputeResult());
StatusText.Text = result; // Runs on UI thread after await
```

### 从后台线程修改集合

从后台线程修改一个已绑定的 `ObservableCollection`，并不一定总会抛出异常。更麻烦的是，项目可能会被静默丢弃，或者只添加了一部分，导致问题很难排查：

```csharp
// WRONG: Collection changes may be silently lost
await Task.Run(async () =>
{
    foreach (var item in loadedItems)
    {
        Items.Add(item); // May only add the first item
    }
});

// CORRECT: Load data on background thread, update collection on UI thread
var data = await Task.Run(() => LoadItems());
Items = new ObservableCollection<Item>(data);

// ALSO CORRECT: Dispatch each addition if you need incremental updates
foreach (var item in loadedItems)
{
    await Dispatcher.UIThread.InvokeAsync(() => Items.Add(item));
}
```

如果你的 `async` 方法最初是在 UI 线程上启动的，那么 `await` 之后的代码会自动恢复到 UI 线程（参见 [SynchronizationContext](#异步模式)）。在这种 continuation 中修改集合时，不需要再显式调度。真正的问题通常发生在：整个方法本身一直运行在后台线程上，或者更早的调用链中使用了 `ConfigureAwait(false)`。

### 阻塞 UI 线程

```csharp
// WRONG: Blocks the UI thread, making the app unresponsive
var data = LoadDataFromNetwork().Result; // Deadlock risk!

// CORRECT: Use async/await
var data = await LoadDataFromNetworkAsync();
```

### 不必要的调度

```csharp
// UNNECESSARY: Already on UI thread in event handlers
private void OnButtonClick(object? sender, RoutedEventArgs e)
{
    // No need to dispatch - event handlers run on the UI thread
    StatusText.Text = "Clicked";
}
```

## 另请参阅

- [`Dispatcher` API reference](/api/avalonia/threading/dispatcher)
- [Application Lifetimes](/docs/fundamentals/application-lifetimes): How the application lifecycle interacts with threading.
