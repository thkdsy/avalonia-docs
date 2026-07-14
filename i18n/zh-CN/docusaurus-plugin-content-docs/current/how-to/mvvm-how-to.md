---
id: mvvm-how-to
title: "如何：实现常见 MVVM 模式"
description: 学习如何在 Avalonia 中使用 CommunityToolkit.Mvvm 实现常见 MVVM 模式，包括可观察属性、命令、消息通信、依赖注入和验证。
doc-type: how-to
---

本指南介绍如何在 Avalonia 中使用推荐的 MVVM 框架 `CommunityToolkit.Mvvm` 实现常见且实用的 MVVM 模式。每一节都会围绕一个具体模式展开，并附上可直接迁移到你自己项目中的代码示例。

## 配置 CommunityToolkit.Mvvm

先在项目中安装对应的 NuGet 包：

```bash
dotnet add package CommunityToolkit.Mvvm
```

安装完成后，你就可以使用工具包提供的源生成器和基类，减少视图模型中的样板代码。

## 可观察属性

使用 `[ObservableProperty]` 特性，可以自动生成支持 `INotifyPropertyChanged` 的属性。你只需声明一个私有后备字段，源生成器就会为你创建对应的公共属性：

```csharp
using CommunityToolkit.Mvvm.ComponentModel;

public partial class PersonViewModel : ObservableObject
{
    [ObservableProperty]
    private string _firstName = "";

    [ObservableProperty]
    private string _lastName = "";

    // 自动生成：带 INotifyPropertyChanged 支持的 FirstName 和 LastName 属性
}
```

你的类必须标记为 `partial`，这样源生成器才能把生成出的成员补进去。生成出的属性名遵循 .NET 命名约定，例如 `_firstName` 会变成 `FirstName`。

### 计算属性

当一个属性依赖另一个属性时，可以使用 `[NotifyPropertyChangedFor]`，自动为依赖属性触发变更通知：

```csharp
[ObservableProperty]
[NotifyPropertyChangedFor(nameof(FullName))]
private string _firstName = "";

[ObservableProperty]
[NotifyPropertyChangedFor(nameof(FullName))]
private string _lastName = "";

public string FullName => $"{FirstName} {LastName}";
```

每当 `FirstName` 或 `LastName` 变化时，工具包也会为 `FullName` 触发 `PropertyChanged`，从而让 UI 保持同步。

### 属性变化回调

你还可以定义 partial 方法，让源生成器在属性变化时自动调用它们，从而执行额外逻辑：

```csharp
[ObservableProperty]
private string _searchText = "";

partial void OnSearchTextChanged(string value)
{
    // 在 SearchText 变化后调用
    ApplyFilter(value);
}

partial void OnSearchTextChanging(string value)
{
    // 在 SearchText 变化前调用
}
```

`OnSearchTextChanging` 会在新值赋入前执行，因此你可以先检查即将进入的值；而 `OnSearchTextChanged` 会在赋值完成后执行，适合触发例如列表筛选之类的副作用逻辑。

## 命令

命令可以让你把 UI 操作（例如按钮点击）绑定到视图模型中的方法上。

### 基础命令

只要把 `[RelayCommand]` 标记在一个方法上，工具包就会自动为你生成对应的 `IRelayCommand` 属性：

```csharp
[RelayCommand]
private void Save()
{
    _repository.Save(CurrentItem);
}
```

这样会生成一个 `SaveCommand` 属性。命名规则是在原方法名后面追加 `Command`。

### 带参数的命令

你可以通过给方法增加参数，把数据从视图传递到命令中：

```csharp
[RelayCommand]
private void Delete(Item item)
{
    Items.Remove(item);
}
```

然后在 AXAML 中绑定命令及其参数：

```xml
<Button Content="Delete"
        Command="{Binding DeleteCommand}"
        CommandParameter="{Binding SelectedItem}" />
```

### 异步命令

对于耗时操作，建议使用 `async Task` 方法。工具包会在命令运行期间自动禁用它，并提供内置的取消支持：

```csharp
[RelayCommand]
private async Task LoadDataAsync(CancellationToken token)
{
    IsLoading = true;
    var data = await _api.FetchDataAsync(token);
    Items = new ObservableCollection<Item>(data);
    IsLoading = false;
}
```

自动生成的命令会：

- 在任务运行期间禁用关联按钮
- 传入一个 `CancellationToken`，便于你取消操作
- 暴露 `IsRunning` 属性，方便显示进度状态

```xml
<Button Content="Load" Command="{Binding LoadDataCommand}" />
<ProgressBar IsVisible="{Binding LoadDataCommand.IsRunning}" IsIndeterminate="True" />
```

### CanExecute

你可以根据视图模型当前状态，有条件地启用或禁用一个命令。对于影响条件判断的属性，可以使用 `[NotifyCanExecuteChangedFor]`，让命令在属性变化时自动重新评估：

```csharp
[ObservableProperty]
[NotifyCanExecuteChangedFor(nameof(SaveCommand))]
private string _name = "";

[RelayCommand(CanExecute = nameof(CanSave))]
private void Save()
{
    // Save logic
}

private bool CanSave() => !string.IsNullOrWhiteSpace(Name);
```

当 `CanSave()` 返回 `false` 时，绑定到 `SaveCommand` 的按钮会自动禁用。而当 `Name` 变化时，命令会重新判断自己是否可以执行。

## 视图模型之间的通信

### 使用消息器

`WeakReferenceMessenger` 允许你在视图模型之间发送消息，而无需彼此建立直接引用。这样可以让各个视图模型保持解耦：

```csharp
using CommunityToolkit.Mvvm.Messaging;

// 定义消息
public record ItemSelectedMessage(Item Item);

// 从一个视图模型发送
WeakReferenceMessenger.Default.Send(new ItemSelectedMessage(selectedItem));

// 在另一个视图模型中接收
public class DetailViewModel : ObservableRecipient, IRecipient<ItemSelectedMessage>
{
    public DetailViewModel()
    {
        IsActive = true; // 开始接收消息
    }

    public void Receive(ItemSelectedMessage message)
    {
        LoadItem(message.Item);
    }
}
```

设置 `IsActive = true` 后，视图模型就会注册为消息接收者。当你把它设为 `false`（或者对象被垃圾回收）时，注册也会自动移除。

### 请求/响应模式

如果你的场景需要拿到一个响应（例如确认对话框），可以使用请求消息模式：

```csharp
public record ConfirmDeleteRequest(Item Item);

// 发起请求
var confirmed = WeakReferenceMessenger.Default
    .Send(new ConfirmDeleteRequest(item));

// 响应处理器（放在视图或协调器中）
WeakReferenceMessenger.Default.Register<ConfirmDeleteRequest>(this, async (r, m) =>
{
    // 显示确认对话框
    m.Reply(await ShowConfirmDialogAsync());
});
```

## 依赖注入

把视图模型和服务注册到 DI 容器中，可以更清晰地管理它们的生命周期和依赖关系：

```csharp
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddViewModels(this IServiceCollection services)
    {
        services.AddTransient<MainViewModel>();
        services.AddTransient<SettingsViewModel>();
        services.AddSingleton<IDataService, DataService>();
        return services;
    }
}
```

然后在 `App.axaml.cs` 中把容器接起来：

```csharp
public override void OnFrameworkInitializationCompleted()
{
    var services = new ServiceCollection();
    services.AddViewModels();
    var provider = services.BuildServiceProvider();

    if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktop)
    {
        desktop.MainWindow = new MainWindow
        {
            DataContext = provider.GetRequiredService<MainViewModel>()
        };
    }

    base.OnFrameworkInitializationCompleted();
}
```

对于每次都应重新创建的视图模型，请使用 `AddTransient`；对于需要在整个应用中共享状态的服务，请使用 `AddSingleton`。

## 使用构造函数注入的视图模型

当你在 DI 容器中注册视图模型后，就可以通过构造函数注入服务。容器会自动解析所有依赖项：

```csharp
public partial class MainViewModel : ObservableObject
{
    private readonly IDataService _dataService;
    private readonly INavigationService _navigation;

    public MainViewModel(IDataService dataService, INavigationService navigation)
    {
        _dataService = dataService;
        _navigation = navigation;
    }

    [RelayCommand]
    private async Task LoadAsync()
    {
        var items = await _dataService.GetItemsAsync();
        Items = new ObservableCollection<Item>(items);
    }
}
```

这种模式让视图模型更易于测试，因为你可以在单元测试中用 `IDataService` 和 `INavigationService` 的模拟实现来替代真实服务。

## ObservableCollection patterns

### Replace vs. add

当需要更新大量项目时，直接替换整个集合通常比逐个添加项目要快得多。每次调用 `Add` 都会触发一次 UI 更新，而重新赋值一个新集合通常只会触发一次更新：

```csharp
// Slow: UI updates on each Add
foreach (var item in newItems)
    Items.Add(item);

// Fast: single notification
Items = new ObservableCollection<Item>(newItems);
```

### Filtered collection

你可以在筛选文本变化时，通过替换显示集合的方式来实现过滤：

```csharp
[ObservableProperty]
private string _filter = "";

[ObservableProperty]
private ObservableCollection<Item> _filteredItems = new();

partial void OnFilterChanged(string value)
{
    FilteredItems = new ObservableCollection<Item>(
        _allItems.Where(i => i.Name.Contains(value, StringComparison.OrdinalIgnoreCase)));
}
```

请将 `ItemsControl` 或 `ListBox` 绑定到 `FilteredItems`，而不是底层的 `_allItems` 集合。

## 验证

将 `ObservableValidator` 作为基类，就可以在视图模型属性上启用基于数据注解的验证：

```csharp
public partial class RegisterViewModel : ObservableValidator
{
    [ObservableProperty]
    [NotifyDataErrorInfo]
    [Required(ErrorMessage = "Name is required")]
    private string _name = "";

    [RelayCommand]
    private void Submit()
    {
        ValidateAllProperties();
        if (!HasErrors)
        {
            // Proceed with submission
        }
    }
}
```

`[NotifyDataErrorInfo]` 特性会通知源生成器在属性变化时自动触发验证。Avalonia 的数据绑定系统能够捕获这些验证错误，并通过 `DataValidationErrors` 在 UI 中显示出来。

有关如何在视图中显示验证错误的更多信息，请参阅 [Validation in data binding](/docs/data-binding/binding-validation)。

## 另请参阅

- [The MVVM pattern](/docs/fundamentals/the-mvvm-pattern)
- [Binding to commands](/docs/data-binding/binding-to-commands)
- [INotifyPropertyChanged](/docs/data-binding/inotifypropertychanged)
- [Dependency injection](/docs/app-development/dependency-injection)
- [Validation in data binding](/docs/data-binding/binding-validation)
