---
id: how-to-bind-to-an-observable
title: 如何绑定到 observable
description: 将控件属性绑定到 IObservable 流，以在 UI 中实现响应式数据更新。
doc-type: how-to
---

Avalonia 支持通过 `^`（流绑定）运算符，直接绑定到 `IObservable<T>` 属性上。每当 observable 产生新值时，绑定都会自动更新。

## 何时使用 observable 绑定

当你的数据会随着时间持续不断地产生一串值时，就适合使用 `IObservable<T>` 绑定。常见场景包括：

- **实时数据流**，例如时钟、传感器读数或股票价格。
- **响应式搜索**，即在查询服务前，对用户输入进行节流、去抖或转换。
- **事件驱动状态**，也就是值是被主动推送给你的，而不是按需拉取的。

对于大多数会因用户操作而变化的视图模型属性来说，使用能够触发 `INotifyPropertyChanged` 的普通属性（或者 Avalonia 的 `StyledProperty` / `DirectProperty`）通常更简单，也已经足够。只有当你确实需要 `System.Reactive` 提供的组合操作符，例如 `Throttle`、`DistinctUntilChanged`、`CombineLatest` 和 `Switch` 时，才更适合选择 observable。

## 基础 observable 绑定

如果 `DataContext.Name` 是一个 `IObservable<string>`，你可以绑定到它当前发出的值：

```xml
<TextBlock Text="{Binding Name^}" />
```

`^` 运算符会订阅这个 observable，并在每次发出新值时更新控件。

## 绑定到发出值的某个属性

你可以在 `^` 运算符之后继续链式访问属性。例如，要绑定到每次发出字符串的 `Length` 属性：

```xml
<TextBlock Text="{Binding Name^.Length}" />
```

## 示例：使用 observable 实现时钟

```csharp
public class ClockViewModel
{
    public IObservable<string> CurrentTime { get; } =
        Observable.Interval(TimeSpan.FromSeconds(1))
            .Select(_ => DateTime.Now.ToString("HH:mm:ss"));
}
```

```xml
<TextBlock Text="{Binding CurrentTime^}" FontSize="24" />
```

## 示例：搜索结果流

```csharp
public class SearchViewModel
{
    private readonly Subject<string> _searchText = new();

    public IObservable<IReadOnlyList<string>> Results { get; }

    public SearchViewModel()
    {
        Results = _searchText
            .Throttle(TimeSpan.FromMilliseconds(300))
            .DistinctUntilChanged()
            .SelectMany(query => SearchAsync(query));
    }

    public void OnSearchTextChanged(string text) => _searchText.OnNext(text);

    private async Task<IReadOnlyList<string>> SearchAsync(string query)
    {
        // 执行搜索
        return new[] { $"Result for '{query}'" };
    }
}
```

```xml
<ListBox ItemsSource="{Binding Results^}" />
```

## 为初始状态设置 FallbackValue

由于 observable 在一开始可能还没有发出任何值，因此你可以使用 `FallbackValue` 来显示占位内容：

```xml
<TextBlock Text="{Binding CurrentTime^, FallbackValue='Loading...'}" />
```

## 与任务绑定结合使用

`^` 运算符同样适用于 `Task<T>` 属性。详情请参阅 [How to bind to a task result](/docs/data-binding/how-to-bind-to-a-task-result)。

## 清理与释放

当绑定激活时，Avalonia 会自动订阅你的 `IObservable<T>`；当绑定的控件从视觉树移除时，也会自动取消订阅。在大多数情况下，你不需要自己管理订阅。

不过仍需注意以下几点：

- **Hot observable**（例如 `Subject<T>`）只要还有对象引用它，就会一直存活。如果这个 observable 由生命周期很长的服务持有，请确保在视图消失后，视图模型不会继续让它保持存活。
- **Cold observable**（例如 `Observable.Interval`）每次都会创建一个新的订阅。由于 Avalonia 会在控件分离时自动释放订阅，因此通常不需要手动清理。
- 如果你的视图模型实现了 `IDisposable`，并且你在 XAML 绑定之外自己创建了订阅（例如在构造函数中为派生属性建立订阅），请在 `Dispose` 方法中释放这些订阅，以避免内存泄漏。

## 另请参阅

- [How to bind to a task result](/docs/data-binding/how-to-bind-to-a-task-result)
- [Data binding syntax](/docs/data-binding/data-binding-syntax)
