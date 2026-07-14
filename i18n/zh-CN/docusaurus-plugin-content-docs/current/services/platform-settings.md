---
id: platform-settings
title: 平台设置
description: 通过 PlatformSettings 服务访问平台特定设置，例如点击尺寸、双击时间、系统颜色和快捷键配置。
doc-type: reference
---

`PlatformSettings` 服务提供对平台特定设置和信息的访问。其中一些设置会在运行时随着用户修改操作系统偏好而变化，因此你的应用可以动态作出响应。

你可以通过任意 `Visual` 上的 `GetPlatformSettings` 扩展方法访问 `PlatformSettings`。有关如何访问平台服务的更多细节，请参阅 [TopLevel](/docs/fundamentals/top-level) 页面。

```csharp title="Getting PlatformSettings"
var platformSettings = myControl.GetPlatformSettings();
```

## 方法

### `GetTapSize(PointerType type)`

返回围绕 pointer-down 位置的矩形尺寸；只有当 pointer-up 发生在该范围内时，才会被识别为一次点击手势。该值以设备无关像素为单位。

```csharp
Size GetTapSize(PointerType type);
```

### `GetDoubleTapSize(PointerType type)`

返回围绕 pointer-down 位置的矩形尺寸；只有当 pointer-up 发生在该范围内时，才会被识别为一次双击手势。该值以设备无关像素为单位。

```csharp
Size GetDoubleTapSize(PointerType type);
```

### `GetDoubleTapTime(PointerType type)`

返回双击手势中第一次点击与第二次点击之间允许的最大时间间隔。

```csharp
TimeSpan GetDoubleTapTime(PointerType type);
```

### `GetColorValues()`

返回当前系统颜色值，包括是否启用了深色模式，以及用户选择的强调色。

```csharp
PlatformColorValues GetColorValues();
```

:::tip
虽然内置的 `FluentTheme` 支持自动切换强调色，但你也可以使用这个方法根据操作系统颜色设置应用自定义逻辑。
:::

## 属性

### `HoldWaitDuration`

获取从 pointer 按下到 `Holding` 事件触发之间的时间间隔。

```csharp
TimeSpan HoldWaitDuration { get; }
```

### `HotkeyConfiguration`

获取 Avalonia 应用的平台特定快捷键配置。该属性返回一个 `PlatformHotkeyConfiguration` 对象，其中包含复制、粘贴、剪切、全选、撤销等常见操作的按键手势。

```csharp
PlatformHotkeyConfiguration HotkeyConfiguration { get; }
```

:::tip
当你的应用需要以平台感知的方式处理 Copy、Paste、Cut 等常见手势时，`HotkeyConfiguration` 会特别有用。
:::

下面的示例演示了如何检查某个按键事件是否匹配平台的 Copy 手势：

```csharp title="Handling platform-specific hotkeys"
protected override void OnKeyDown(KeyEventArgs e)
{
    var hotkeys = this.GetPlatformSettings()?.HotkeyConfiguration;
    if (hotkeys is not null && hotkeys.Copy.Any(g => g.Matches(e)))
    {
        // Handle Copy hotkey.
    }
}
```

## 事件

### `ColorValuesChanged`

当系统颜色值发生变化时触发。这里包括深色模式和强调色的变化。你可以订阅此事件，以实时更新应用的外观。

```csharp
event EventHandler<PlatformColorValues>? ColorValuesChanged;
```

下面的示例订阅颜色变化，并记录新的主题变体：

```csharp title="Responding to system color changes"
var platformSettings = myControl.GetPlatformSettings();
if (platformSettings is not null)
{
    platformSettings.ColorValuesChanged += (sender, values) =>
    {
        Debug.WriteLine($"Theme variant: {values.ThemeVariant}");
    };
}
```

## 另请参阅

- [TopLevel](/docs/fundamentals/top-level)：从控件访问平台服务。
- [Pointer events](/docs/input-interaction/pointer)：处理指针输入与手势。
- [Gestures](/docs/input-interaction/gestures)：处理点击和其他手势事件。
