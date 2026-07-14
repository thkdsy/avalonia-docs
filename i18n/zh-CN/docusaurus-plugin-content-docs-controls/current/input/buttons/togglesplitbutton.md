---
id: togglesplitbutton
title: ToggleSplitButton
---

import ToggleSplitButtonTextListScreenshot from '/img/controls/buttons/togglesplitbutton/togglesplitbutton-text-list.png';

[`ToggleSplitButton`](/api/avalonia/controls/togglesplitbutton) 的行为类似一个带有主区域和次区域的 [`ToggleButton`](/controls/input/buttons/togglebutton)，这两个部分都可以分别按下。主区域的行为与普通 `ToggleButton` 相同，而次区域会打开一个包含附加操作的 [`Flyout`](/controls/menus/menuflyout)。

:::info
`ToggleSplitButton` 只有两种状态：选中和未选中。它不像标准 `ToggleButton` 那样支持不确定状态。这是有意为之，以保持与 WinUI 一致，同时限制该控件的使用场景。`ToggleSplitButton` 应仅用于开启/关闭某项功能。从可用性角度来看，将它用于其他用途通常都不是好做法。
:::

## 这是合适的控件吗？

`ToggleSplitButton` 是一种相当专门化的控件，应该只在从用户角度看用途非常明确的场景中使用。它的目的是在开启/关闭某项功能的同时，允许指定一些额外配置，而不是只使用默认配置。

与 [`SplitButton`](/controls/input/buttons/splitbutton) 一样，最常见的操作应作为默认操作，并显示在主区域中。但与 `SplitButton` 不同的是，按下主区域并不会执行某个动作，而是用于开启或关闭该功能。该功能的附加配置应放到次区域按下时显示的 [`Flyout`](/api/avalonia/controls/flyout) 中。

:::info
在 `Flyout` 中按下某个配置项时，应当执行以下两种行为之一：（1）以所选配置开启该功能；（2）将该功能切换为所选配置。按下 `Flyout` 中的配置项不应该关闭该功能；关闭只能通过切换主区域来完成。
:::

## 常用属性

| 属性 | 说明 |
| ----------- | -------------------------------------------------------------- |
| `Content` | 显示在主区域中的内容 |
| `Flyout` | 当次区域被点击时显示的 `Flyout` |
| `Command` | 当主按钮被点击时要调用的命令 |
| `IsChecked` | 获取或设置 `ToggleSplitButton` 是否处于选中状态 |

## 伪类

| 伪类 | 说明 |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `:pressed` | 当整个 `ToggleSplitButton` 通过键盘输入（如 Space 或 Enter）被按下时设置。在这种状态下，不区分主区域和次区域。 |
| `:flyout-open` | 当 `Flyout` 打开时设置。 |
| `:checked` | 当 `ToggleSplitButton` 处于选中状态时设置。(`IsChecked="true"`) |

## 示例

### 基本示例

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             Padding="20">
  <ToggleSplitButton Content="Content"
                   IsChecked="{Binding IsChecked}">
    <ToggleSplitButton.Flyout>
        <MenuFlyout Placement="Bottom">
            <MenuItem Header="Item 1">
                <MenuItem Header="Subitem 1" />
                <MenuItem Header="Subitem 2" />
                <MenuItem Header="Subitem 3" />
            </MenuItem>
            <MenuItem Header="Item 2"
                      InputGesture="Ctrl+A" />
            <MenuItem Header="Item 3" />
        </MenuFlyout>
    </ToggleSplitButton.Flyout>
  </ToggleSplitButton>
</UserControl>
```

</XamlPreview>

### 带编号或项目符号列表的文本编辑器

<Image light={ToggleSplitButtonTextListScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

延续 `SplitButton` 的文本编辑器示例，`ToggleSplitButton` 的一个常见用途是为文本添加项目符号列表或编号列表。在这个示例中，主区域用于开启或关闭列表，而次区域会打开一个 `Flyout`，允许选择项目符号样式或编号样式。

```xml
<!-- We have the following Icons defined in our Resources -->
<PathGeometry x:Key="IconData.NumberedList"> {{ Path Data }} </PathGeometry>
<PathGeometry x:Key="IconData.BulletedList"> {{ Path Data }} </PathGeometry>
```

```xml
<ToggleSplitButton IsChecked="{Binding TextEditorHasList}">
    <ToggleSplitButton.Content>
        <!-- 注意：此示例中内容保持静态，但你也可以使用动态内容 -->
        <PathIcon Data="{DynamicResource IconData.BulletedList}" />
    </ToggleSplitButton.Content>
    <ToggleSplitButton.Flyout>
        <Flyout Placement="Bottom">
            <!-- 注意：此示例中内容保持静态，但你也可以使用动态内容 -->
            <ListBox Height="200" Width="200" >
                <ListBoxItem>
                    <StackPanel Orientation="Horizontal">
                        <PathIcon Data="{DynamicResource IconData.NumberedList}" />
                        <TextBlock Text="Numbered List" />
                    </StackPanel>
                </ListBoxItem>
                <ListBoxItem>
                    <StackPanel Orientation="Horizontal">
                        <PathIcon Data="{DynamicResource IconData.BulletedList}" />
                        <TextBlock Text="Bulleted List" />
                    </StackPanel>
                </ListBoxItem>
            </ListBox>
        </Flyout>
    </ToggleSplitButton.Flyout>
</ToggleSplitButton>
```

## 另请参阅

- [ToggleSplitButton API reference](/api/avalonia/controls/togglesplitbutton)
- [`ToggleSplitButton.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/SplitButton/ToggleSplitButton.cs)
