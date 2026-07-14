---
id: toggleswitch
title: ToggleSwitch
description: 一个用于二元设置的滑动开关控件，支持自定义开启和关闭状态的显示内容。
doc-type: reference
---

[`ToggleSwitch`](/api/avalonia/controls/toggleswitch) 控件提供一个可在开和关之间切换的滑动开关。它的行为类似于 [`CheckBox`](/api/avalonia/controls/checkbox)，但采用轨道加滑块的视觉形式，更适合移动端和触控优先的界面。

当你需要即时生效的开/关设置时，例如启用深色模式或切换通知状态，就可以使用 `ToggleSwitch`。如果是表单中让用户从列表里选择多个选项的场景，通常 `CheckBox` 更合适。

## 常用属性

你最常使用的通常是这些属性：

| 属性      | 类型      | 说明                                                        |
| ------------- | --------- | ------------------------------------------------------------------ |
| `IsChecked`   | `bool?`   | 获取或设置当前开关状态。`true` 表示开启，`false` 表示关闭。 |
| `OnContent`   | `object`  | 开关处于开启状态时显示的内容。默认值为 "On"。         |
| `OffContent`  | `object`  | 开关处于关闭状态时显示的内容。默认值为 "Off"。       |
| `KnobTransitions` | `Transitions` | 状态变化时应用到滑块上的过渡效果。    |

## 事件

| 事件              | 说明                              |
| ------------------ | ---------------------------------------- |
| `IsCheckedChanged` | 当 `IsChecked` 值发生变化时引发。 |

## 基本示例

在 AXAML 中放置一个 `ToggleSwitch`，并将 `IsChecked` 绑定到视图模型中的布尔属性：

```xml
<ToggleSwitch IsChecked="{Binding IsEnabled}" />
```

## 自定义开/关标签

你可以使用自定义字符串替换默认的 "On" 和 "Off" 文本：

```xml
<ToggleSwitch IsChecked="{Binding IsDarkMode}"
              OnContent="Dark"
              OffContent="Light" />
```

## 隐藏标签

将两个内容属性都设为空字符串，即可只显示滑动开关本身：

```xml
<ToggleSwitch IsChecked="{Binding IsActive}"
              OnContent=""
              OffContent="" />
```

当周围布局已经为该设置提供说明文字时，这种方式会很有用。

## 富内容

你可以使用任意控件作为开和关状态下显示的内容。下面的示例将 `PathIcon` 与 `TextBlock` 组合在一起：

```xml
<ToggleSwitch IsChecked="{Binding NotificationsEnabled}">
    <ToggleSwitch.OnContent>
        <StackPanel Orientation="Horizontal" Spacing="6">
            <PathIcon Data="{StaticResource bell_regular}" Width="14" />
            <TextBlock Text="Enabled" />
        </StackPanel>
    </ToggleSwitch.OnContent>
    <ToggleSwitch.OffContent>
        <StackPanel Orientation="Horizontal" Spacing="6">
            <PathIcon Data="{StaticResource bell_off_regular}" Width="14" />
            <TextBlock Text="Disabled" />
        </StackPanel>
    </ToggleSwitch.OffContent>
</ToggleSwitch>
```

## 绑定到视图模型

在视图模型中创建布尔属性，并将每个 `ToggleSwitch` 绑定到对应属性：

```csharp
public partial class SettingsViewModel : ObservableObject
{
    [ObservableProperty]
    private bool _isDarkMode;

    [ObservableProperty]
    private bool _notificationsEnabled = true;

    partial void OnIsDarkModeChanged(bool value)
    {
        // 应用主题切换
    }
}
```

```xml
<StackPanel Spacing="12">
    <ToggleSwitch IsChecked="{Binding IsDarkMode}"
                  OnContent="Dark Mode" OffContent="Light Mode" />
    <ToggleSwitch IsChecked="{Binding NotificationsEnabled}"
                  OnContent="Notifications On" OffContent="Notifications Off" />
</StackPanel>
```

由于 `ToggleSwitch` 默认使用双向绑定，因此切换开关会立即更新视图模型中的属性。

## 设置表单模式

一种常见布局是在左侧放说明文字，在右侧放不带标签的 `ToggleSwitch`：

```xml
<StackPanel Spacing="16">
    <Grid ColumnDefinitions="*,Auto">
        <StackPanel>
            <TextBlock Text="Auto-save" FontWeight="SemiBold" />
            <TextBlock Text="Save changes automatically"
                       Foreground="Gray" FontSize="12" />
        </StackPanel>
        <ToggleSwitch Grid.Column="1" IsChecked="{Binding AutoSave}"
                      OnContent="" OffContent="" />
    </Grid>

    <Grid ColumnDefinitions="*,Auto">
        <StackPanel>
            <TextBlock Text="Spell check" FontWeight="SemiBold" />
            <TextBlock Text="Check spelling as you type"
                       Foreground="Gray" FontSize="12" />
        </StackPanel>
        <ToggleSwitch Grid.Column="1" IsChecked="{Binding SpellCheck}"
                      OnContent="" OffContent="" />
    </Grid>
</StackPanel>
```

将 `OnContent` 和 `OffContent` 设为空字符串，可以去掉冗余标签，因为 `TextBlock` 已经描述了每项设置。

## 在 `ToggleSwitch` 与 `CheckBox` 之间做选择

| 对比项 | `ToggleSwitch` | `CheckBox` |
| ------------- | -------------- | ---------- |
| 视觉风格  | 滑动开关 | 复选标记 |
| 最适合的场景 | 设置项、即时开关状态 | 表单字段、多选列表 |
| 三态支持 | 否 | 是（通过 `IsThreeState`） |
| 平台体验 | 更适合移动端和触控 | 传统桌面风格 |

当变更需要立即生效时，选择 `ToggleSwitch`。当用户需要先确认或提交表单后才应用更改时，选择 `CheckBox`。

## 另请参阅

- [CheckBox](/controls/input/selectors/checkbox)
- [ToggleButton](/controls/input/buttons/togglebutton)
- [RadioButton](/controls/input/buttons/radiobutton)
