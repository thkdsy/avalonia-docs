---
id: macos
title: macOS
---

## Avalonia 如何在 macOS 上运行

Avalonia 不使用标准的 .NET macOS workload（`net10.0-macos`）。相反，它自带一套原生平台后端，通过编译后的动态库（`libAvaloniaNative.dylib`）与 macOS API 交互，完全绕过微软提供的托管 macOS 绑定。

这个原生后端由 Objective-C++（`.mm` 文件）编写，代码位于 Avalonia 仓库中的 [`native/Avalonia.Native/src/OSX`](https://github.com/AvaloniaUI/Avalonia/tree/master/native/Avalonia.Native/src/OSX)。它提供平台所需的核心能力：窗口、输入处理、Metal 和 OpenGL 渲染、剪贴板、菜单、拖放、系统托盘、文件对话框以及无障碍支持。.NET 侧通过 MicroCom 与这些原生代码通信；MicroCom 是一个轻量级的类 COM 互操作层。两侧之间的接口定义在一个 [IDL 文件](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Native/avn.idl) 中，而 MicroCom 会生成跨边界封送调用所需的托管包装器。

这种设计带来一个很重要的实际好处：你可以在 Windows 或 Linux 上为 macOS 构建和编译 Avalonia 桌面应用，而无需安装 macOS workload，也不必拥有一台 Mac。默认 target framework 只是 `net9.0`（或 `net10.0`），而不是某个特定平台版本。

代价在于，Avalonia 提供的原生绑定是精简版的。它们覆盖了框架构建 UI 所需的内容，但不会暴露完整的 macOS 平台 API（例如 MapKit、HealthKit、StoreKit 等）。

### 访问完整的 macOS API

如果你的应用需要 Avalonia 之外的 macOS API，请将 target framework 改为 macOS 专用版本：

```xml
<TargetFramework>net10.0-macos</TargetFramework>
```

这样可以获得 .NET macOS workload 提供的完整 API 集，但也会带来一个限制：**面向 macOS TFM 的构建必须在 macOS 上完成**。也就是说，你将失去从 Windows 或 Linux 交叉编译的能力。

## 应用名称与标识

macOS 会在多个位置显示你的应用名称：菜单栏、“About” 对话框、“Quit” 菜单项、Dock 工具提示以及窗口标题栏。要让这些地方显示正确，就需要在正确的位置设置名称。

### 名称来源

| 位置 | 来源 | 说明 |
|---|---|---|
| 菜单栏（加粗应用名） | `Info.plist` 中的 `CFBundleName`（已打包），或 `Application.Name`（未打包） | `CFBundleName` 最多 15 个字符。 |
| Dock 工具提示 | `Info.plist` 中的 `CFBundleDisplayName`，回退到 `CFBundleName` | 超过 15 个字符时应使用 `CFBundleDisplayName`。 |
| “About” 菜单项 | 你定义的 [`NativeMenuItem`](/api/avalonia/controls/nativemenuitem) 的 Header 文本 | 该文本完全由你控制。 |
| “Quit” 菜单项 | `CFBundleName` 或 `Application.Name` | Avalonia 会自动生成 “Quit App Name”。 |
| 窗口标题栏 | `Window.Title` 属性 | 与应用名称独立。 |

### 设置应用名称

**步骤 1：在 `App.axaml` 中设置 `Application.Name`**

这会控制开发阶段（也就是你还没有 `.app` 包时）的名称显示：

```xml
<Application Name="My Application" ...>
```

**步骤 2：在 `Info.plist` 中设置 `CFBundleName` 和 `CFBundleDisplayName`**

当应用以 `.app` 包形式运行时，macOS 会从 `Info.plist` 中读取名称，而不是使用 `Application.Name`。请确保这些值保持一致：

```xml
<key>CFBundleName</key>
<string>My App</string>

<key>CFBundleDisplayName</key>
<string>My Application</string>
```

`CFBundleName` 最多 15 个字符，用于菜单栏和 “Quit” 菜单项。`CFBundleDisplayName` 没有长度限制，主要用于 Finder 和 Dock。如果你的应用名称本来就在 15 个字符以内，那么只设置 `CFBundleName` 就够了。

## 原生菜单栏

macOS 应用拥有位于屏幕顶部、独立于应用窗口的菜单栏。Avalonia 通过 [`NativeMenu`](/api/avalonia/controls/nativemenu) 支持这一点，它会被渲染为原生 macOS 菜单栏。

### 应用菜单

菜单栏最左侧的菜单会显示你的应用名称，通常包含 “About”、“Preferences” 和 “Quit” 等菜单项。你可以通过在 `App.axaml` 中为 `Application` 附加一个 `NativeMenu` 来定义它：

```xml
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyApp.App"
             Name="My Application">

    <NativeMenu.Menu>
        <NativeMenu>
            <NativeMenuItem Header="About My Application..." Click="About_OnClick" />
            <NativeMenuItem Header="Preferences..." Click="Preferences_OnClick"
                            Gesture="Meta+Comma" />
        </NativeMenu>
    </NativeMenu.Menu>
</Application>
```

如果你没有定义 `NativeMenu`，Avalonia 会创建一个默认应用菜单，其中包含 “About Avalonia” 项。定义你自己的菜单即可替换它。

Avalonia 会在你的自定义菜单项后自动追加标准菜单项，包括一个分隔符以及带有 <kbd>⌘</kbd><kbd>Q</kbd> 快捷键的 “Quit App Name” 项。你不需要自己手动添加 Quit 菜单项。

### 替换 About 对话框

应用菜单中的 “About” 项是你自己定义的 `NativeMenuItem`。它没有任何内建特殊行为。你只需把它的 `Click` 事件连接到你希望显示的 UI：

```csharp
private void About_OnClick(object? sender, EventArgs e)
{
    var aboutWindow = new AboutWindow();
    aboutWindow.ShowDialog(this.GetMainWindow());
}
```

macOS 用户通常期望 About 菜单项是应用菜单中的第一项，并且标题文本类似 “About My Application...” 并带有省略号。这是一种平台惯例，而不是框架强制要求。

### 窗口菜单

如果要添加 File、Edit 这类标准菜单，可将 `NativeMenu` 附加到 `Window` 上：

```xml
<Window xmlns="https://github.com/avaloniaui">
    <NativeMenu.Menu>
        <NativeMenu>
            <NativeMenuItem Header="File">
                <NativeMenu>
                    <NativeMenuItem Header="Open..." Gesture="Meta+O"
                                    Click="FileOpen_OnClick" />
                    <NativeMenuItem Header="Save" Gesture="Meta+S"
                                    Command="{Binding SaveCommand}" />
                    <NativeMenuItemSeparator />
                    <NativeMenuItem Header="Close" Gesture="Meta+W"
                                    Click="FileClose_OnClick" />
                </NativeMenu>
            </NativeMenuItem>
            <NativeMenuItem Header="Edit">
                <NativeMenu>
                    <NativeMenuItem Header="Cut" Gesture="Meta+X"
                                    Command="{Binding CutCommand}" />
                    <NativeMenuItem Header="Copy" Gesture="Meta+C"
                                    Command="{Binding CopyCommand}" />
                    <NativeMenuItem Header="Paste" Gesture="Meta+V"
                                    Command="{Binding PasteCommand}" />
                </NativeMenu>
            </NativeMenuItem>
        </NativeMenu>
    </NativeMenu.Menu>
</Window>
```

在 macOS 上，名为 “Edit” 的菜单是特殊的。Avalonia 会自动为带有这个标题的菜单添加标准 macOS 文本编辑功能（例如自动补全和字符替换）。

每个 `NativeMenuItem` 都需要配置 `Click` 事件处理程序或 `Command` 绑定，否则它会显示为灰色不可用状态。

### Dock 菜单

当用户在 Dock 中右键（或 Control-点击）你的应用图标时，macOS 会显示一个上下文菜单。你可以通过在 `App.axaml` 中为 `Application` 附加 `NativeDock.Menu` 来自定义该菜单：

```xml
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyApp.App"
             Name="My Application">

    <NativeDock.Menu>
        <NativeMenu>
            <NativeMenuItem Header="New Window" Click="NewWindow_OnClick" />
            <NativeMenuItemSeparator />
            <NativeMenuItem Header="Show Main Window" Click="ShowMainWindow_OnClick" />
        </NativeMenu>
    </NativeDock.Menu>
</Application>
```

你定义的 dock 菜单项会显示在 macOS 自动添加的系统标准项（例如 “Options” 和 “Quit”）之上。

你也可以在运行时修改 dock 菜单：

```csharp
var dockMenu = NativeDock.GetMenu(this);
if (dockMenu is not null)
{
    dockMenu.Items.Insert(0, new NativeMenuItem("Dynamic Item"));
}
```

:::note
`NativeDock.Menu` 只在 macOS 上生效。在其他平台上，这个属性会被忽略。
:::

### 键盘快捷键

`Gesture` 属性用于为菜单项指定键盘快捷键。Avalonia 在手势字符串中使用平台中立的修饰键名称，而在 macOS 上，它们会映射到标准修饰键：

| Avalonia 修饰键 | macOS 键位 | 符号 |
|---|---|---|
| `Meta` | Command | <kbd>⌘</kbd> |
| `Control` | Control | <kbd>⌃</kbd> |
| `Shift` | Shift | <kbd>⇧</kbd> |
| `Alt` | Option | <kbd>⌥</kbd> |

一个手势字符串由一个或多个修饰键通过 `+` 拼接，最后再加上按键名称：

| `Gesture` 值 | macOS 快捷键 |
|---|---|
| `Meta+S` | <kbd>⌘</kbd> <kbd>S</kbd> |
| `Meta+Shift+S` | <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>S</kbd> |
| `Meta+Comma` | <kbd>⌘</kbd> <kbd>,</kbd> |
| `Meta+Alt+Q` | <kbd>⌘</kbd> <kbd>⌥</kbd> <kbd>Q</kbd> |

## macOS 平台惯例

macOS 用户会期待某些标准快捷键和行为。Avalonia 会自动处理其中一部分，但也有一些需要你显式配置。

### 标准快捷键

下列快捷键属于 macOS 用户普遍期待的行为。你可以通过 `NativeMenu` 的手势或 `KeyBinding` 来配置它们：

| 操作 | 快捷键 | 说明 |
|---|---|---|
| Preferences | <kbd>⌘</kbd> <kbd>,</kbd> | 应打开你的设置/偏好界面 |
| Quit | <kbd>⌘</kbd> <kbd>Q</kbd> | 由原生菜单自动处理 |
| Close Window | <kbd>⌘</kbd> <kbd>W</kbd> | 绑定到关闭当前活动窗口 |
| Minimize | <kbd>⌘</kbd> <kbd>M</kbd> | 自动处理 |
| Hide | <kbd>⌘</kbd> <kbd>H</kbd> | 自动处理 |
| Full Screen | <kbd>⌘</kbd> <kbd>⌃</kbd> <kbd>F</kbd> | 自动处理 |
| Select All | <kbd>⌘</kbd> <kbd>A</kbd> | 在文本控件中自动处理 |
| Find | <kbd>⌘</kbd> <kbd>F</kbd> | 绑定到你的查找功能 |

### PlatformHotkeyConfiguration

Avalonia 会自动把常见热键适配到当前平台。在 macOS 上，复制/粘贴/剪切使用的是 Cmd 而不是 Ctrl。你可以在运行时通过 `PlatformSettings.HotkeyConfiguration` 查询当前平台的热键映射：

```csharp
protected override void OnKeyDown(KeyEventArgs e)
{
    var hotkeys = this.GetPlatformSettings()?.HotkeyConfiguration;
    if (hotkeys?.Copy.Any(g => g.Matches(e)) == true)
    {
        // 处理复制
    }
}
```

当你在构建需要响应平台标准快捷键的自定义控件时，这会很有用，因为你不必把修饰键硬编码进去。

## 嵌入原生视图

你可以使用 `NativeControlHost` 在 Avalonia 控件内部托管原生 macOS 视图（NSView 子类）。这对于集成那些 Avalonia 没有对应实现的平台特定 UI 组件很有帮助，例如地图视图、相机预览或平台媒体播放器。

`NativeControlHost` 的工作方式是：在窗口中提供一块区域，让原生视图与 Avalonia 渲染表面一起合成显示。要使用它，你需要创建一个平台特定实现，并返回原生视图的句柄：

```csharp
public class NativeControlHostExample : NativeControlHost
{
    protected override IPlatformHandle CreateNativeControlCore(
        IPlatformHandle parent)
    {
        if (OperatingSystem.IsMacOS())
        {
            // 创建并返回 NSView 的句柄
            // 该视图会托管在当前控件的边界内
        }

        return base.CreateNativeControlCore(parent);
    }

    protected override void DestroyNativeControlCore(
        IPlatformHandle control)
    {
        // 清理原生资源
        base.DestroyNativeControlCore(control);
    }
}
```

以这种方式渲染的原生视图会处于独立于 Avalonia 渲染的合成层中，因此它们总是显示在 Avalonia 内容的上方或下方，而不会参与常规视觉树的 z 轴排序。

:::note
嵌入原生视图需要使用 `net10.0-macos` target framework，因为你必须访问 macOS API 才能创建这些原生视图。请参阅前文的[访问完整的 macOS API](#accessing-the-full-macos-api-surface)。
:::

### 在原生 macOS 应用中嵌入 Avalonia

反过来也是可行的。你可以把 Avalonia 的渲染表面作为 NSView 嵌入到原生 macOS（Cocoa 或 Mac Catalyst）应用的视图层级中，从而在原生应用内部托管 Avalonia UI。这对于将现有 macOS 应用逐步迁移到 Avalonia，或只想在原生应用的某些局部视图中使用 Avalonia 的场景很有帮助。

## URL 协议处理程序

你可以注册应用来处理自定义 URL scheme（例如 `myapp://open`），这样用户在浏览器或其他应用中点击相应链接时，就会启动你的应用。请在 `Info.plist` 中添加 `CFBundleURLTypes` 条目：

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLName</key>
        <string>MyApp</string>
        <key>CFBundleTypeRole</key>
        <string>Viewer</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>myapp</string>
        </array>
    </dict>
</array>
```

完成配置后，像 `myapp://some-action` 这样的 URL 就会打开你的应用。你也可以查看 `/Applications` 中其他应用的 `Info.plist`，了解它们是如何配置 URL scheme 的。

## 文件类型关联

你可以将应用注册为特定文件类型的处理程序，这样在 Finder 中双击这些文件时就会由你的应用打开。请在 `Info.plist` 中添加 `CFBundleDocumentTypes` 条目：

```xml
<key>CFBundleDocumentTypes</key>
<array>
    <dict>
        <key>CFBundleTypeName</key>
        <string>Sketch</string>
        <key>CFBundleTypeExtensions</key>
        <array>
            <string>sketch</string>
        </array>
        <key>CFBundleTypeIconFile</key>
        <string>icon.icns</string>
        <key>CFBundleTypeRole</key>
        <string>Viewer</string>
        <key>LSHandlerRank</key>
        <string>Default</string>
    </dict>
</array>
```

| 键 | 说明 |
|---|---|
| `CFBundleTypeName` | 面向用户显示的文件类型名称。 |
| `CFBundleTypeExtensions` | 需要关联的文件扩展名数组（不带点号）。 |
| `CFBundleTypeIconFile` | 这类文件显示的图标。 |
| `CFBundleTypeRole` | 应用扮演的角色：`Editor`（可读写）、`Viewer`（只读）或 `None`。 |
| `LSHandlerRank` | 优先级：`Owner`（该类型由你的应用创建）、`Default`、`Alternate` 或 `None`。 |

## 原生代码

Avalonia 的原生 macOS 代码位于 `native/Avalonia.Native/src/OSX`。如果你需要修改或调试原生层，可以在 Xcode 中打开 `Avalonia.Native.OSX.xcodeproj` 项目。

你可以在 Xcode 中按 <kbd>⌘</kbd> <kbd>B</kbd> 编译修改后的代码，然后让 Avalonia 应用指向这个修改后的 dylib。输出路径可以通过点击 Xcode 项目导航器中 **Products** 下的 dylib 查看，然后在 `AppBuilder` 中指定它：

```csharp
.With(new AvaloniaNativePlatformOptions
{
    AvaloniaNativeLibraryPath = "[Path to your dylib]",
})
```

### 在开发期间以 app bundle 方式运行

某些 macOS 特性要求你的应用以标准 `.app` bundle 形式运行。例如，如果不是这种形式，Xcode 的 Accessibility Inspector 就无法识别你的应用。

如果你不想走完整打包流程，也可以通过修改 `.csproj` 中的输出路径，让它看起来像一个 bundle 结构：

```xml
<OutputPath>bin\$(Configuration)\$(Platform)\MyApp.app/Contents/MacOS</OutputPath>
<AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
<UseAppHost>true</UseAppHost>
```

然后在 `Contents` 目录中放入一个有效的 `Info.plist`。关于 `Info.plist` 的细节，请参阅 [macOS 部署指南](/docs/deployment/macos)。

## Mac Catalyst 作为替代方案

Avalonia 也支持通过 Apple 的 Mac Catalyst 框架在 macOS 上运行 iOS 应用。这与本页介绍的 Avalonia Native 后端是不同的方案。Mac Catalyst 需要在 Mac 上构建，并依赖 `maccatalyst` .NET workload，因此你会失去从 Windows 或 Linux 交叉编译的能力。它主要适用于应用高度依赖 UIKit API，或需要把 Avalonia 嵌入到 MAUI 混合应用中的场景。对于大多数 Avalonia 应用来说，前文介绍的默认 macOS 后端仍然是更推荐的选择。更多信息请参阅 iOS 平台指南中的 [Mac Catalyst](/docs/platform-specific-guides/ios#mac-catalyst)。

## 另请参阅

- [部署到 macOS](/docs/deployment/macos)
- [iOS 平台指南](/docs/platform-specific-guides/ios)（包含 Mac Catalyst）
- [NativeMenu 控件参考](/controls/menus/nativemenu)
- [键盘与快捷键](/docs/input-interaction/keyboard-and-hotkeys)
