---
id: intent-filter
title: 注册应用以打开文件
description: 配置 Android intent filter，让你的 Avalonia 应用能够打开文件和自定义 URI scheme。
doc-type: how-to
---

Android 允许应用通过 intent filter 注册为特定文件类型或 URI scheme 的处理程序。当用户打开匹配的文件或链接时，Android 会启动你的应用，并通过激活事件传递相应数据。本指南将带你完成：让 Avalonia 应用能够处理纯文本文件，并使用 Avalonia 存储 API 读取它们的内容。

## 前置条件

- 一个面向 Android 的 Avalonia 项目
- 一个继承自 `AvaloniaMainActivity` 的 `MainActivity`

## 步骤 1：为 Activity 添加 `IntentFilter` 特性

为 `MainActivity` 添加 `[IntentFilter]` 特性。在构建时，该特性会自动把对应的 `<intent-filter>` 条目合并到 Android manifest 中，因此你不需要手动编辑 XML。

下面的示例会把应用注册为可处理通过 `file` 或 `content` URI scheme 打开的纯文本文件（`text/plain`）：

```csharp
[Activity(
    Label = "Demo.Android",
    Theme = "@style/MyTheme.NoActionBar",
    Icon = "@drawable/icon",
    MainLauncher = true,
    ConfigurationChanges = ConfigChanges.Orientation | ConfigChanges.ScreenSize | ConfigChanges.UiMode)]
[IntentFilter(["android.intent.action.VIEW"],
    Categories = [Intent.CategoryDefault, Intent.CategoryBrowsable],
    DataSchemes = ["file", "content"],
    DataMimeType = "text/plain",
    DataPathPattern = ".*\\.txt")]
public class MainActivity : AvaloniaMainActivity
{
    // See Steps 2 and 3 below.
}
```

### 选择 `DataMimeType` 和 `DataPathPattern`

| 属性 | 用途 | 示例 |
|---|---|---|
| `DataMimeType` | 匹配发送方应用报告的 MIME 类型。 | `"text/plain"`, `"application/pdf"` |
| `DataPathPattern` | 匹配文件路径（仅对 `file` scheme 生效）。 | `".*\\.txt"`, `".*\\.csv"` |

:::tip
如果你想处理多个 MIME 类型，请为每一种类型分别添加一个 `[IntentFilter]` 特性。Android 不支持在单个 filter 中声明 MIME 类型列表。
:::

## 步骤 2：监听 `Activated` 事件

在 `MainActivity` 构造函数中，为 `IAvaloniaActivity.Activated` 绑定处理程序。当 Android 为打开文件而激活你的应用时，Avalonia 会触发这个事件，并传入一个 `FileActivatedEventArgs` 实例，其中包含收到的存储项。

```csharp
public class MainActivity : AvaloniaMainActivity
{
    public MainActivity()
    {
        ((IAvaloniaActivity)this).Activated += HandleIntent;
    }

    private static void HandleIntent(object? sender, ActivatedEventArgs e)
    {
        if (e is FileActivatedEventArgs fileActivated && Avalonia.Application.Current is App app)
        {
            app.OpenFiles(fileActivated.Files);
        }
    }
}
```

:::note
无论 Android 是启动应用的新实例，还是把已存在的实例切换到前台，`Activated` 事件都会触发。请确保你的处理逻辑可以被安全地调用多次。
:::

## 步骤 3：把文件转发给 ViewModel

使用静态属性 `Avalonia.Application.Current`，你可以从 activity 获取 `App` 实例。一种比较方便的做法是：在 `OnFrameworkInitializationCompleted` 中创建 ViewModel 后把它保存到字段里，然后公开一个可供 activity 调用的辅助方法。

### `App.axaml.cs`

```csharp
public partial class App : Application
{
    private MainViewModel? mainViewModel;

    public override void OnFrameworkInitializationCompleted()
    {
        mainViewModel = new MainViewModel();

        if (ApplicationLifetime is ISingleViewApplicationLifetime singleView)
        {
            singleView.MainView = new MainView
            {
                DataContext = mainViewModel
            };
        }

        base.OnFrameworkInitializationCompleted();
    }

    public void OpenFiles(IReadOnlyList<IStorageItem> files)
    {
        mainViewModel?.OpenFiles(files);
    }
}
```

### `MainViewModel.cs`

```csharp
public class MainViewModel
{
    public async void OpenFiles(IReadOnlyList<IStorageItem> files)
    {
        foreach (IStorageItem item in files)
        {
            if (item is IStorageFile file)
            {
                using Stream stream = await file.OpenReadAsync();
                // Read the stream (use StreamReader, etc.)
            }
        }
    }
}
```

## 处理自定义 URI scheme

你可以通过调整 `DataSchemes` 数组来注册自定义 URI scheme（例如 `myapp://`）：

```csharp
[IntentFilter(["android.intent.action.VIEW"],
    Categories = [Intent.CategoryDefault, Intent.CategoryBrowsable],
    DataSchemes = ["myapp"])]
```

当应用通过自定义 scheme 被激活时，应检查 `ProtocolActivatedEventArgs`，而不是 `FileActivatedEventArgs`：

```csharp
private static void HandleIntent(object? sender, ActivatedEventArgs e)
{
    if (e is FileActivatedEventArgs fileActivated && Avalonia.Application.Current is App app)
    {
        app.OpenFiles(fileActivated.Files);
    }
    else if (e is ProtocolActivatedEventArgs protocolActivated)
    {
        // protocolActivated.Uri contains the full URI, e.g. myapp://path?query=value
    }
}
```

## 故障排查

| 现象 | 可能原因 |
|---|---|
| 你的应用没有出现在 Android 的分享/打开面板中。 | `[IntentFilter]` 中的 MIME 类型与发送方应用提供的不匹配。可使用 `adb shell am start -a android.intent.action.VIEW -t "text/plain" -d "content://..."` 检查。 |
| `FileActivatedEventArgs.Files` 为空。 | 发送方使用了你的 filter 未包含的 URI scheme。请确保 `DataSchemes` 中同时列出了 `"file"` 和 `"content"`。 |
| 启动时处理程序触发了两次。 | 你可能同时在构造函数和 `OnCreate` 重写中注册了事件。请只在一个位置注册。 |

## 另请参阅

- [Android 平台指南](/docs/platform-specific-guides/android)
- [部署到 Android](/docs/deployment/android)
- [Android intent filter 文档（developer.android.com）](https://developer.android.com/guide/components/intents-filters)
