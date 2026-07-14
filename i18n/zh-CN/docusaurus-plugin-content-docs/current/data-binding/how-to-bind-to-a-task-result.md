---
id: how-to-bind-to-a-task-result
title: 如何绑定到任务结果
description: 将控件属性绑定到异步 Task 的结果，以便在任务完成时显示数据。
doc-type: how-to
---

Avalonia 支持通过 `^`（流绑定）运算符，直接绑定到 `Task<T>` 属性上。任务完成后，绑定会自动显示结果，从而让你能够异步加载数据，而无需手动更新属性。

## 基础任务绑定

如果某个属性值需要通过耗时操作来加载，那么你可以直接绑定到 `async Task<TResult>` 的结果上。

先把任务定义为视图模型上的一个属性：

```csharp
public Task<string> MyAsyncText => GetTextAsync();

private async Task<string> GetTextAsync()
{
    await Task.Delay(1000); // 模拟一个耗时操作
    return "Hello from async operation";
}
```

然后使用 `^` 运算符绑定到其结果：

```xml
<TextBlock Text="{Binding MyAsyncText^, FallbackValue='Loading...'}" />
```

在任务仍在执行时，会显示 `FallbackValue`。一旦任务完成，结果就会替换掉它。

:::tip
在任务绑定上，建议始终设置 `FallbackValue`。否则，在任务完成前，绑定属性会一直保持默认值（通常是 `null` 或空值），这可能导致布局抖动或控件空白。
:::

## 从 API 加载数据

一种常见场景是在创建视图模型时就开始加载数据：

```csharp
public class UserProfileViewModel
{
    public Task<UserProfile> Profile { get; }

    public UserProfileViewModel(IUserService userService)
    {
        Profile = userService.GetCurrentUserAsync();
    }
}
```

你还可以在 `^` 运算符之后使用点号语法，绑定任务结果上的嵌套属性：

```xml
<StackPanel>
    <TextBlock Text="{Binding Profile^.Name, FallbackValue='Loading profile...'}" />
    <TextBlock Text="{Binding Profile^.Email}" />
</StackPanel>
```

## 显示加载指示器

你可以使用 `FallbackValue`，或者绑定可见性来在加载期间显示一个加载指示器：

```xml
<Panel>
    <ProgressBar IsIndeterminate="True"
                 IsVisible="{Binding !Profile^, FallbackValue=True}" />
    <TextBlock Text="{Binding Profile^.Name}"
               IsVisible="{Binding !!Profile^, FallbackValue=False}" />
</Panel>
```

`!` 前缀会对绑定值取反。在任务尚未完成时，`Profile^` 的结果为 `null`，所以 `!Profile^` 为 `True`，因此进度条可见。`!!` 双重取反则会把结果重新转换为布尔值，因此只有在任务完成并返回非 null 结果后，`!!Profile^` 才会变为 `True`。

## 刷新任务数据

由于一个 `Task<T>` 只能完成一次，所以每当你想刷新数据时，都需要创建一个新的任务实例。然后触发 `PropertyChanged`，让绑定系统重新接管这个新任务：

```csharp
public class RefreshableViewModel : INotifyPropertyChanged
{
    private readonly IUserService _userService;
    private Task<UserProfile>? _profile;

    public event PropertyChangedEventHandler? PropertyChanged;

    public Task<UserProfile>? Profile
    {
        get => _profile;
        private set
        {
            _profile = value;
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(Profile)));
        }
    }

    public RefreshableViewModel(IUserService userService)
    {
        _userService = userService;
        Refresh();
    }

    public void Refresh()
    {
        Profile = _userService.GetCurrentUserAsync();
    }
}
```

每次调用 `Refresh()`，都会分配一个新的 `Task<UserProfile>`，而绑定系统也会自动订阅这个新任务。

## 边界情况与限制

使用任务绑定时，请注意以下几点：

- **任务失败：** 如果任务抛出异常，`FallbackValue` 会继续显示。Avalonia 不会自动把异常暴露到 UI 上。如果你需要展示错误状态，建议在视图模型中处理异常，并额外暴露一个错误消息属性。
- **任务取消：** 被取消的任务行为与失败任务类似。`FallbackValue` 会保持显示，并且不会向绑定推送结果。
- **已完成任务：** 如果绑定到的是一个已经完成的任务，那么结果会立即显示，不会有等待过程。
- **null 结果：** 如果任务完成后返回 `null`，那么绑定值也会变成 `null`。如果你想在这种情况下显示特定内容，请为绑定设置 `TargetNullValue`。
- **线程安全：** `^` 运算符会使用 `SynchronizationContext` 把结果切回 UI 线程，因此你通常不需要自己处理线程调度。
- **编译绑定：** 带 `^` 运算符的任务绑定同样可以与编译绑定一起使用。请通过在视图上设置 `x:DataType`，确保任务属性类型在编译期可见。

:::warning
避免把任务属性定义成一个 getter 中的方法调用，并且每次访问都创建新的 `Task`（例如 `public Task<string> Data => LoadAsync();`）。因为绑定系统每次读取该属性时，都会创建一个新的任务，这可能导致重复的网络请求或其他意外副作用。更好的做法是把任务保存到一个后备字段中，或者只在构造函数里赋值一次。
:::

## 另请参阅

- [How to bind to an observable](/docs/data-binding/how-to-bind-to-an-observable)
- [Data binding syntax](/docs/data-binding/data-binding-syntax)
- [Compiled bindings](/docs/data-binding/compiled-bindings)
