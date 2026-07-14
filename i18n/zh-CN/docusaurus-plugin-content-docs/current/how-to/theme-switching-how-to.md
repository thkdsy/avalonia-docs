---
id: theme-switching-how-to
title: "如何：在浅色与深色主题之间切换"
description: 实现浅色/深色主题切换、持久化用户选择，并创建能感知主题变化的资源。
doc-type: how-to
---

本指南将介绍如何实现浅色/深色主题切换、保存用户偏好，并创建能够在主题变化时自动响应的主题感知资源。

## 全局设置主题

你可以在应用程序级别通过设置 `App.axaml` 中的 `RequestedThemeVariant`，来覆盖系统默认主题：

```xml title="App.axaml"
<Application RequestedThemeVariant="Dark">
    <Application.Styles>
        <FluentTheme />
    </Application.Styles>
</Application>
```

`RequestedThemeVariant` 属性支持三个值：

- `Default`：跟随操作系统主题。
- `Light`：强制使用浅色主题。
- `Dark`：强制使用深色主题。

## 运行时切换主题

如果你希望通过代码切换主题，可以直接设置当前 `Application` 实例上的 `RequestedThemeVariant`：

```csharp
if (Application.Current is { } app)
{
    app.RequestedThemeVariant = ThemeVariant.Dark;
}
```

### 在视图模型中切换主题

你可以把 `ToggleSwitch` 绑定到视图模型属性上，让用户在浅色和深色模式之间切换。下面的示例使用了 MVVM Community Toolkit 的源生成器：

```csharp title="SettingsViewModel.cs"
public partial class SettingsViewModel : ObservableObject
{
    [ObservableProperty]
    private bool _isDarkMode;

    partial void OnIsDarkModeChanged(bool value)
    {
        if (Application.Current is { } app)
        {
            app.RequestedThemeVariant = value ? ThemeVariant.Dark : ThemeVariant.Light;
        }
    }
}
```

```xml title="SettingsView.axaml"
<ToggleSwitch IsChecked="{Binding IsDarkMode}"
              OnContent="Dark" OffContent="Light" />
```

### 三选一：浅色、深色、跟随系统

如果你希望给用户增加第三个“跟随系统”的选项，可以使用一个包含三项的 `ComboBox`，并把该选项映射到 `ThemeVariant.Default`：

```csharp title="SettingsViewModel.cs"
public partial class SettingsViewModel : ObservableObject
{
    [ObservableProperty]
    private string _themeChoice = "System";

    partial void OnThemeChoiceChanged(string value)
    {
        if (Application.Current is not { } app) return;

        app.RequestedThemeVariant = value switch
        {
            "Light" => ThemeVariant.Light,
            "Dark" => ThemeVariant.Dark,
            _ => ThemeVariant.Default // 跟随系统
        };
    }
}
```

```xml title="SettingsView.axaml"
<ComboBox SelectedItem="{Binding ThemeChoice}">
    <ComboBoxItem Content="System" />
    <ComboBoxItem Content="Light" />
    <ComboBoxItem Content="Dark" />
</ComboBox>
```

## 持久化主题选择

如果你希望用户的主题偏好在应用重启后仍然保留，就需要在关闭时保存，在启动时恢复。下面的示例展示了如何在 `App.axaml.cs` 的重写方法中加载已保存的偏好：

```csharp title="App.axaml.cs"
public override void OnFrameworkInitializationCompleted()
{
    // 加载已保存的主题设置
    var settings = new SettingsService().Load();
    RequestedThemeVariant = settings.Theme switch
    {
        "Light" => ThemeVariant.Light,
        "Dark" => ThemeVariant.Dark,
        _ => ThemeVariant.Default
    };

    base.OnFrameworkInitializationCompleted();
}
```

关于如何实现一个完整的 `SettingsService` 来读写用户偏好，请参阅 [Data persistence how-to](/docs/how-to/data-persistence-how-to)。

## 使用 ThemeDictionaries 实现主题感知颜色

你可以定义会根据当前激活主题自动变化的资源。做法是在 `Application.Resources` 中加入 `ResourceDictionary.ThemeDictionaries`，并分别为 `Light` 和 `Dark` 提供独立字典：

```xml title="App.axaml"
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.ThemeDictionaries>
            <ResourceDictionary x:Key="Light">
                <SolidColorBrush x:Key="CardBackground" Color="#FFFFFF" />
                <SolidColorBrush x:Key="CardBorder" Color="#E5E7EB" />
                <SolidColorBrush x:Key="TextPrimary" Color="#111827" />
            </ResourceDictionary>
            <ResourceDictionary x:Key="Dark">
                <SolidColorBrush x:Key="CardBackground" Color="#1F2937" />
                <SolidColorBrush x:Key="CardBorder" Color="#374151" />
                <SolidColorBrush x:Key="TextPrimary" Color="#F9FAFB" />
            </ResourceDictionary>
        </ResourceDictionary.ThemeDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

使用 `DynamicResource` 引用这些资源，这样当主题切换时，它们也会同步更新：

```xml
<Border Background="{DynamicResource CardBackground}"
        BorderBrush="{DynamicResource CardBorder}"
        BorderThickness="1" CornerRadius="8" Padding="16">
    <TextBlock Text="Theme-aware card" Foreground="{DynamicResource TextPrimary}" />
</Border>
```

:::tip
对于主题变体资源，请始终使用 `DynamicResource`，而不要使用 `StaticResource`。因为 `StaticResource` 只会在加载时解析一次，之后用户切换主题时不会自动更新。
:::

## 使用 ThemeVariantScope 实现混合主题

你可以通过 `ThemeVariantScope` 强制让 UI 的某个局部区域使用特定主题。这在你希望窗口中的某个部分固定保持某种主题，而不受全局主题影响时非常有用：

```xml
<StackPanel Spacing="16">
    <!-- 这一部分无论全局主题如何，都始终使用浅色 -->
    <ThemeVariantScope RequestedThemeVariant="Light">
        <Border Background="{DynamicResource SystemRegionColor}" Padding="16" CornerRadius="8">
            <TextBlock Text="Always light theme" />
        </Border>
    </ThemeVariantScope>

    <!-- 这一部分始终使用深色主题 -->
    <ThemeVariantScope RequestedThemeVariant="Dark">
        <Border Background="{DynamicResource SystemRegionColor}" Padding="16" CornerRadius="8">
            <TextBlock Text="Always dark theme" />
        </Border>
    </ThemeVariantScope>
</StackPanel>
```

## 检测当前主题

你可以在运行时通过 `ActualThemeVariant` 属性检查当前激活的主题变体。这在你需要执行一些无法通过 XAML 表达的逻辑时很有帮助，例如选择平台特定资源：

```csharp
if (Application.Current is { } app)
{
    var currentTheme = app.ActualThemeVariant;

    if (currentTheme == ThemeVariant.Dark)
    {
        // 深色模式专属逻辑
    }
}
```

## 响应主题变化

你可以订阅 `ActualThemeVariantChanged` 事件，以便在主题变化时作出响应。这对于更新那些不在 XAML 中声明的资源尤其有用，例如图表颜色、地图瓦片或第三方控件配置：

```csharp
if (Application.Current is { } app)
{
    app.ActualThemeVariantChanged += (sender, args) =>
    {
        var isDark = app.ActualThemeVariant == ThemeVariant.Dark;
        // 更新图表颜色、地图瓦片等资源
    };
}
```

## 自定义主题变体

除了 `Light` 和 `Dark` 之外，你还可以定义自己的命名主题变体。做法是创建一个新的 `ThemeVariant`，并为它指定一个回退主题，这样当某些资源未显式定义时，Avalonia 就会使用这个回退变体：

```csharp
public static readonly ThemeVariant HighContrast = new("HighContrast", ThemeVariant.Light);
```

然后为你的自定义变体名称添加一个对应的 `ThemeDictionary` 条目：

```xml
<ResourceDictionary.ThemeDictionaries>
    <ResourceDictionary x:Key="HighContrast">
        <SolidColorBrush x:Key="CardBackground" Color="Black" />
        <SolidColorBrush x:Key="TextPrimary" Color="Yellow" />
    </ResourceDictionary>
</ResourceDictionary.ThemeDictionaries>
```

如果要启用这个自定义变体，可以像内置主题那样把它赋给 `RequestedThemeVariant`：

```csharp
app.RequestedThemeVariant = HighContrast;
```

## 另请参阅

- [Theme variants](/docs/styling/theme-variants)：主题变体系统的完整参考。
- [Resource dictionary](/docs/app-development/resource-dictionary)：资源查找与合并机制说明。
- [Themes](/docs/styling/themes)：`FluentTheme` 与 `SimpleTheme` 的配置方式。
- [Data persistence how-to](/docs/how-to/data-persistence-how-to)：如何跨会话保存设置。
