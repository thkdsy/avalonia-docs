---
id: faq
title: 常见问题
doc-type: troubleshooting
---

## 常规

#### 使用 Avalonia 需要许可证吗？

不需要。**Avalonia 本身依然完全基于 MIT 许可证开源。** 你可以永久免费地使用 Avalonia 构建并发布商业应用程序。Community License 只适用于专业工具链（Visual Studio 扩展、Dev Tools、Parcel），并不适用于框架本身。

#### 如果我完全不想用这些怎么办？

这完全没问题。所有旧版工具仍保持开源，并可在 GitHub 上获取：
- 现有的 Visual Studio 扩展
- Dev Tools
- TreeDataGrid

你可以继续按当前方式使用它们，或者根据自己的需要 fork 后独立维护。

#### 我还能继续使用旧版工具吗？

可以。旧版 FOSS Visual Studio 扩展仍可在 [github.com/AvaloniaUI/AvaloniaVS](https://github.com/AvaloniaUI/AvaloniaVS) 上克隆和构建。旧版 Dev Tools 源码仍然可用，原始 TreeDataGrid 也仍然可用。它们全部采用 MIT 许可证，任何人都可以使用、fork 或维护。

#### 我的 Community License 什么时候过期？

只要你始终符合资格条件，Community License 就不会过期。不过，如果你的情况发生变化（例如你的组织规模超出了资格阈值），你就必须升级到付费许可证。

#### Visual Studio 宽限期结束后会怎样？

在 2026 年 4 月 13 日之后，如果你还没有注册 Community License 或购买付费许可证，你可以：
- 继续使用旧版 FOSS Visual Studio 扩展
- 切换到 Visual Studio Code 或 JetBrains Rider（这些扩展仍可免费使用）
- 如果符合条件，则注册 Community License
- 购买付费许可证

#### 我还有其他问题，可以去哪里提问？

欢迎在 [Community Hub](https://github.com/AvaloniaCommunity) 和 [Avalonia Support](https://support.avaloniaui.net/) 上提出你的问题或反馈。

## Developer tools

#### 是否可以让多个实例连接到 Developer Tools？

可以。当一个 Developer Tools 实例处于运行并已激活状态时，你可以让一个或多个应用连接到它。
每建立一个新连接，都会打开一个新的 Developer Tools 窗口，并且它们彼此独立工作。

#### 它支持 Browser/Android/iOS 吗？

支持移动端和浏览器应用程序。
更多信息请参阅 [附加到浏览器或移动应用](/tools/developer-tools/attaching-applications)。

#### 我可以在 NativeAOT 应用中使用 Developer Tools 和 DiagnosticsPackage 吗？

可以。DiagnosticsPackage 对 trimming 完全友好。虽然它确实使用了反射，但该工具已经在 AOT 环境下经过测试。

#### `AvaloniaUI.DiagnosticsSupport` 是否取代了 `Avalonia.Diagnostics` 包？还是两个都需要？

你只需要 `AvaloniaUI.DiagnosticsSupport`。
`Avalonia.Diagnostics` 是旧版开发者工具使用的老包，可以安全地从项目中移除。
如果出于某些特殊原因确实需要，两个包也可以同时引用，但你可能需要为每个工具设置不同的手势快捷键。

#### 即使没有许可证，任何人都可以构建引用 `AvaloniaUI.DiagnosticsSupport` 的项目吗？

可以。`AvaloniaUI.DiagnosticsSupport` 是一个集成包，用于在 `Developer Tools` 与用户应用之间搭建桥梁。它本身不需要许可证，也可以在公开项目中引用。

但如果你要真正打开 `Developer Tools`，则仍然需要许可证和 Avalonia 门户账号。

#### 是否有必要在 Release/Production 构建中排除 `AvaloniaUI.DiagnosticsSupport` 包？

该工具对于 Release 构建的内部测试依然很有用，因此并不是说它只能包含在 Debug 构建中。

与旧版 Avalonia DevTools 不同，这个包并不会携带那些可能导致 Release 编译出现问题的大型依赖项。

不过，出于安全性和最终包体积的考虑，仍然建议在生产构建中排除该包。

为此，可以使用 `Condition="'$(Configuration)' == 'Debug'"'`：
```xml
<PackageReference Include="AvaloniaUI.DiagnosticsSupport" Version="" Condition="'$(Configuration)' == 'Debug'" />
```

同时再配合对 `this.AttachDeveloperTools()` 或 `.WithDeveloperTools()` 使用 `#if DEBUG`。

#### 该工具是否已经提供或计划提供 arm64 和 x86 构建？

目前 Windows 和 Linux 仅提供 **x64** 构建。
macOS 构建则是同时包含 **x64** 与 **arm64** 架构的通用 bundle。

## TreeDataGrid

### Data Updates

#### TreeDataGrid doesn't update when I change model properties

**Problem**: You're modifying properties on your data objects, but the grid doesn't reflect the changes.

**Solution**: Your data model must implement `INotifyPropertyChanged`. The TreeDataGrid relies on property change notifications to update the UI.

```csharp
// ❌ Wrong - No property change notifications
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

// ✅ Correct - Implements INotifyPropertyChanged
public class Person : INotifyPropertyChanged
{
    private string _name;
    private int _age;

    public string Name
    {
        get => _name;
        set
        {
            if (_name != value)
            {
                _name = value;
                OnPropertyChanged();
            }
        }
    }

    public int Age
    {
        get => _age;
        set
        {
            if (_age != value)
            {
                _age = value;
                OnPropertyChanged();
            }
        }
    }

    public event PropertyChangedEventHandler? PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

#### New items don't appear when I add them to the collection

**Problem**: You're adding items to a `List<T>` or array, but the grid doesn't show new items.

**Solution**: Use `ObservableCollection<T>` instead, which automatically notifies the grid of collection changes.

```csharp
// ❌ Wrong - List doesn't notify of changes
private List<Person> _people = new List<Person>();

// ✅ Correct - ObservableCollection notifies of changes
private ObservableCollection<Person> _people = new ObservableCollection<Person>();
```

### Cell Editing

#### Cell editing doesn't work when I click on cells

**Problem**: Clicking cells doesn't begin editing.

**Solution**: Make sure you've provided both a getter and setter in the column definition:

```csharp
// ❌ Wrong - No setter, column is read-only
new TextColumn<Person, string>("Name", x => x.Name)

// ✅ Correct - Has both getter and setter
new TextColumn<Person, string>(
    "Name",
    x => x.Name,
    (row, value) => row.Name = value)
```

你可能还需要指定编辑手势：

```csharp
new TextColumn<Person, string>(
    "Name",
    x => x.Name,
    (row, value) => row.Name = value,
    options: new TextColumnOptions<Person>
    {
        BeginEditGestures = BeginEditGestures.Tap
    })
```

## WebView

#### 支持离屏渲染吗？可以避免 airspace 问题吗？

不支持，当前尚未支持离屏渲染。离屏渲染已被记录为未来可能支持的功能。

#### Linux 上支持 NativeWebView 吗？

支持。Linux 上的 `NativeWebView` 使用 [WPE WebKit](https://wpewebkit.org)，并通过 SHM（软件渲染）进行离屏渲染，因此它不依赖原生窗口嵌入，并且可在 X11 和 Wayland 会话中运行。请参阅[Linux 前置条件](/docs/app-development/embedding-web-content#linux)，了解需要安装的运行时库（`libwpewebkit-2.0`、`libwpe-1.0`、`libWPEBackend-fdo-1.0`）。

如果目标发行版上不可用 WPE，你有两个回退方案：

- 设置 [`LinuxWpeWebViewEnvironmentRequestedEventArgs.PreferWebKitGtkInstead`](/controls/web/webview-environment#linux-wpe-webkit) 以改用 WebKitGTK 适配器。
- 使用 [`NativeWebDialog`](/controls/web/nativewebdialog)，它会在专用窗口中渲染 WebKitGTK。

#### 我可以将 WebAuthenticationBroker 用于 Google Auth 或 Microsoft.Identity Auth 吗？

可以，这两种身份验证提供程序都受支持。你可以：
- 手动构建请求与重定向 `Uri`
- 集成 `Google.Apis.Auth` 和 `Microsoft.Identity.Client` NuGet 包

集成示例可在我们的[示例仓库](https://github.com/AvaloniaUI/Accelerate.Samples/tree/main/WebAuthenticationBrokerSample)中找到。

#### 为什么要使用 WebAuthenticationBroker，而不是其他方案？

虽然 `Microsoft.Identity.Client` 和 `Google.Apis.Auth` 都自带 Web UI 对话框，但这些实现通常受限于特定平台和特定提供程序。WebAuthenticationBroker 提供：
- 与提供程序无关的实现
- 无需特殊框架要求的桌面平台支持
- 不受 mac-catalyst 限制的完整 macOS 支持

#### NativeWebView 是否支持通过 getUserMedia() API 访问摄像头、麦克风和屏幕共享？

支持，`getUserMedia()` API 在各个平台上都受支持。用户会像在桌面浏览器中一样，收到摄像头、麦克风或屏幕共享权限提示。macOS 支持是在 `11.2.4` 版本中加入的。

某些平台还要求开发者在应用 bundle 上配置权限。如果主应用需要某项权限，那么 WebView 很可能也需要该权限。例如，在已打包应用中，macOS/iOS 需要配置 [NSCameraUsageDescription](https://developer.apple.com/documentation/bundleresources/information-property-list/nscamerausagedescription?language=objc)。

## See also

- [Developer tools installation](/tools/developer-tools/installation)
- [Avalonia Tools overview](/tools/)
