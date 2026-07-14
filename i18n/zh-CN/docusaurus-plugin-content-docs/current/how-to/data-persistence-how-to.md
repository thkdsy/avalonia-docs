---
id: data-persistence-how-to
title: "如何：保存和加载应用程序设置"
description: 使用 JSON 文件或数据库，在多个会话之间持久化用户偏好和应用程序状态。
doc-type: how-to
---

本指南介绍如何在 Avalonia 应用程序中跨会话持久化用户偏好和应用程序状态。

## JSON 文件设置

最简单的方式，是把设置保存为用户应用数据目录中的一个 JSON 文件：

```csharp
using System.Text.Json;

public class AppSettings
{
    public string Theme { get; set; } = "Default";
    public double WindowWidth { get; set; } = 800;
    public double WindowHeight { get; set; } = 600;
    public string LastOpenedFile { get; set; } = "";
    public bool ShowSidebar { get; set; } = true;
}

public class SettingsService
{
    private static readonly string SettingsPath = Path.Combine(
        Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
        "MyApp",
        "settings.json");

    public AppSettings Load()
    {
        if (!File.Exists(SettingsPath))
            return new AppSettings();

        var json = File.ReadAllText(SettingsPath);
        return JsonSerializer.Deserialize<AppSettings>(json) ?? new AppSettings();
    }

    public void Save(AppSettings settings)
    {
        var directory = Path.GetDirectoryName(SettingsPath)!;
        Directory.CreateDirectory(directory);

        var json = JsonSerializer.Serialize(settings, new JsonSerializerOptions
        {
            WriteIndented = true
        });
        File.WriteAllText(SettingsPath, json);
    }
}
```

:::tip
建议把 `Load` 方法包在 `try` / `catch` 中。如果 JSON 文件损坏，或者应用版本更新后其结构发生变化，`JsonSerializer.Deserialize` 可能会抛出异常。在 `catch` 中返回一个默认的 `AppSettings` 实例，可以避免应用在启动时直接崩溃。
:::

### 在视图模型中使用

```csharp
public partial class SettingsViewModel : ObservableObject
{
    private readonly SettingsService _settingsService;
    private readonly AppSettings _settings;

    public SettingsViewModel(SettingsService settingsService)
    {
        _settingsService = settingsService;
        _settings = settingsService.Load();
        _theme = _settings.Theme;
        _showSidebar = _settings.ShowSidebar;
    }

    [ObservableProperty]
    private string _theme;

    [ObservableProperty]
    private bool _showSidebar;

    partial void OnThemeChanged(string value)
    {
        _settings.Theme = value;
        _settingsService.Save(_settings);
    }

    partial void OnShowSidebarChanged(bool value)
    {
        _settings.ShowSidebar = value;
        _settingsService.Save(_settings);
    }
}
```

:::note
每次属性变化都立即保存虽然方便，但如果有很多高频变化的设置，就可能导致频繁磁盘 I/O。对于这种场景，建议使用去抖保存，或者通过定时器批量写入。
:::

## 保存窗口位置和大小

你可以持久化窗口几何信息，以便下次启动时恢复：

```csharp
public partial class MainWindow : Window
{
    private readonly SettingsService _settings;

    public MainWindow()
    {
        InitializeComponent();
        _settings = new SettingsService();
        RestoreWindowState();
    }

    private void RestoreWindowState()
    {
        var s = _settings.Load();
        if (s.WindowWidth > 0 && s.WindowHeight > 0)
        {
            Width = s.WindowWidth;
            Height = s.WindowHeight;
        }
    }

    protected override void OnClosing(WindowClosingEventArgs e)
    {
        var s = _settings.Load();
        s.WindowWidth = Width;
        s.WindowHeight = Height;
        _settings.Save(s);
        base.OnClosing(e);
    }
}
```

:::warning
在恢复窗口位置时，请务必确认保存下来的坐标仍然处于当前屏幕布局的有效范围内。因为用户可能在上次会话后断开了外接显示器，从而导致窗口恢复到屏幕之外。你可以通过 `TopLevel` 上的 `Screens.All` 检查可用屏幕边界，再决定是否应用这些保存值。
:::

如果你还希望持久化窗口位置（而不仅仅是大小），可以在 `AppSettings` 类中增加 `WindowX` 和 `WindowY` 属性，并在 `OnClosing` 中写回：

```csharp
// 在 AppSettings 中
public int WindowX { get; set; } = -1;
public int WindowY { get; set; } = -1;

// 在 RestoreWindowState 中
if (s.WindowX >= 0 && s.WindowY >= 0)
{
    Position = new PixelPoint(s.WindowX, s.WindowY);
}

// 在 OnClosing 中
s.WindowX = Position.X;
s.WindowY = Position.Y;
```

## 最近文件列表

你可以跟踪最近打开过的文件，并把它们显示在菜单或欢迎页上：

```csharp
public class RecentFilesService
{
    private const int MaxRecent = 10;
    private readonly string _path;
    private List<string> _recentFiles = new();

    public RecentFilesService()
    {
        _path = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
            "MyApp", "recent.json");
        Load();
    }

    public IReadOnlyList<string> RecentFiles => _recentFiles;

    public void AddFile(string filePath)
    {
        _recentFiles.Remove(filePath);
        _recentFiles.Insert(0, filePath);
        if (_recentFiles.Count > MaxRecent)
            _recentFiles.RemoveAt(_recentFiles.Count - 1);
        Save();
    }

    public void RemoveFile(string filePath)
    {
        _recentFiles.Remove(filePath);
        Save();
    }

    private void Load()
    {
        try
        {
            if (File.Exists(_path))
            {
                var json = File.ReadAllText(_path);
                _recentFiles = JsonSerializer.Deserialize<List<string>>(json) ?? new();
            }
        }
        catch (JsonException)
        {
            _recentFiles = new();
        }
    }

    private void Save()
    {
        var dir = Path.GetDirectoryName(_path)!;
        Directory.CreateDirectory(dir);
        File.WriteAllText(_path, JsonSerializer.Serialize(_recentFiles));
    }
}
```

:::tip
在显示最近文件列表时，建议先检查每个文件是否仍然存在。因为这些文件在上次打开后，可能已经被移动、重命名或删除。提供一个“从列表中移除”的选项（如上面 `RemoveFile` 方法所示），可以显著改善处理失效条目的用户体验。
:::

## 平台特定的存储路径

不同平台应使用各自合适的存储目录。下表列出了常见约定位置：

| 平台 | 常见路径 |
|---|---|
| Windows | `%APPDATA%\MyApp\` |
| macOS | `~/Library/Application Support/MyApp/` |
| Linux | `~/.config/MyApp/` |

```csharp
public static string GetAppDataPath()
{
    if (OperatingSystem.IsMacOS())
        return Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.UserProfile),
            "Library", "Application Support", "MyApp");

    if (OperatingSystem.IsLinux())
        return Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.UserProfile),
            ".config", "MyApp");

    // Windows and others
    return Path.Combine(
        Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
        "MyApp");
}
```

:::note
在 Linux 上，如果设置了 `XDG_CONFIG_HOME` 环境变量，建议遵循该约定。当它存在时，你的应用应将配置文件存储到 `$XDG_CONFIG_HOME/MyApp/`，而不是 `~/.config/MyApp/`。你可以通过 `Environment.GetEnvironmentVariable("XDG_CONFIG_HOME")` 检查这一点。
:::

## 书签存储（移动端与沙箱环境）

在沙箱平台（iOS、Android、WebAssembly）上，标准文件系统 API 可能无法在多个会话之间持续访问用户选择的文件。此时可以使用 Avalonia `IStorageProvider` 的书签功能来持久化文件访问权限：

```csharp
var storage = TopLevel.GetTopLevel(this)?.StorageProvider;
if (storage is null) return;

// Save a bookmark
var file = (await storage.OpenFilePickerAsync(new FilePickerOpenOptions())).FirstOrDefault();
if (file is not null)
{
    var bookmark = await file.SaveBookmarkAsync();
    // Store the bookmark string in your settings
    settings.LastFileBookmark = bookmark;
}

// Restore from bookmark
if (!string.IsNullOrEmpty(settings.LastFileBookmark))
{
    var restored = await storage.OpenFileBookmarkAsync(settings.LastFileBookmark);
    if (restored is not null)
    {
        await using var stream = await restored.OpenReadAsync();
        // Read file contents
    }
}
```

:::warning
如果用户移动或删除了文件，或者操作系统撤销了授权，书签就可能失效。恢复后务必检查得到的 `IStorageFile` 是否不为 `null`，并妥善处理失败情况。在某些平台上，书签也可能会在系统重启后失效。
:::

有关书签 API 的完整说明，请参阅 [Bookmarks](/docs/services/storage/bookmarks)。

## 另请参阅

- [Storage Provider](/docs/services/storage/storage-provider)：跨平台的文件与文件夹访问。
- [Bookmarks](/docs/services/storage/bookmarks)：在沙箱平台上跨会话保留文件访问权限。
- [Storage Item](/docs/services/storage/storage-item)：使用 `IStorageFile` 和 `IStorageFolder` 实例。
- [Window Management](/docs/app-development/window-management)：窗口生命周期、位置与状态管理。
- [Dependency Injection](/docs/app-development/dependency-injection)：在应用中注册 `SettingsService` 之类的服务。
