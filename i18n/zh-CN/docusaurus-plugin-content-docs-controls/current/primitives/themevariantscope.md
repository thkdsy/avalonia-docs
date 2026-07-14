---
id: themevariantscope
title: ThemeVariantScope
description: 一个基础控件，可为可视树中的某个区域覆盖当前激活的主题变体（浅色或深色）。
doc-type: reference
---

[`ThemeVariantScope`](/api/avalonia/controls/themevariantscope) 控件可为可视树中的某个区域覆盖当前激活的主题变体（浅色或深色）。放置在 `ThemeVariantScope` 内部的所有控件都会使用指定的变体，而不受应用程序或窗口级设置的影响。当你需要让界面中的某一部分使用与应用其他部分不同的主题时，这会非常有用。

## 常见使用场景

- 强制让侧边栏或面板始终以深色模式呈现，而应用其余部分使用浅色模式。
- 在设置页面上并排显示两种主题变体的预览效果。
- 在布局中创建具有对比效果的区域，以增强视觉层次感。

## 常用属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `RequestedThemeVariant` | `ThemeVariant` | 在此作用域内要应用的主题变体。可选值：`Light`、`Dark`、`Default`。设置为 `Default` 时会恢复继承的主题变体。 |
| `ActualThemeVariant` | `ThemeVariant` | 只读。此作用域当前实际生效的主题变体。 |

## 基本示例

你可以强制让界面中的某个区域使用浅色主题，而窗口的其他部分保持深色主题：

```xml
<Window RequestedThemeVariant="Dark">
    <StackPanel Spacing="8" Margin="16">
        <Button Content="Dark-themed button" />

        <ThemeVariantScope RequestedThemeVariant="Light">
            <StackPanel Spacing="8">
                <Button Content="Light-themed button" />
                <TextBox PlaceholderText="Light-themed input" />
            </StackPanel>
        </ThemeVariantScope>
    </StackPanel>
</Window>
```

## 并排主题预览

一个常见用法是同时显示两种主题变体，例如在主题设置页面上：

```xml
<Grid ColumnDefinitions="*,*" Margin="16">
    <ThemeVariantScope Grid.Column="0" RequestedThemeVariant="Light">
        <Border Background="{DynamicResource SystemControlBackgroundAltHighBrush}"
                Padding="16" CornerRadius="8">
            <StackPanel Spacing="8">
                <TextBlock Text="Light Theme" FontWeight="SemiBold" />
                <Button Content="Sample Button" />
                <CheckBox Content="Sample Checkbox" IsChecked="True" />
                <Slider Value="60" />
            </StackPanel>
        </Border>
    </ThemeVariantScope>

    <ThemeVariantScope Grid.Column="1" RequestedThemeVariant="Dark">
        <Border Background="{DynamicResource SystemControlBackgroundAltHighBrush}"
                Padding="16" CornerRadius="8">
            <StackPanel Spacing="8">
                <TextBlock Text="Dark Theme" FontWeight="SemiBold" />
                <Button Content="Sample Button" />
                <CheckBox Content="Sample Checkbox" IsChecked="True" />
                <Slider Value="60" />
            </StackPanel>
        </Border>
    </ThemeVariantScope>
</Grid>
```

## 恢复为继承主题

将 `RequestedThemeVariant="Default"` 设为默认值，即可清除覆盖并重新继承父级作用域的主题变体：

```xml
<ThemeVariantScope RequestedThemeVariant="Light">
    <StackPanel>
            <!-- 这些控件使用浅色变体 -->
        <Button Content="Light" />

        <ThemeVariantScope RequestedThemeVariant="Default">
            <!-- 这些控件从窗口/应用继承主题 -->
            <Button Content="Inherited" />
        </ThemeVariantScope>
    </StackPanel>
</ThemeVariantScope>
```

## 嵌套作用域

你可以嵌套多个 `ThemeVariantScope` 控件来创建多个主题区域。每个作用域都会独立解析自己的主题变体，因此子级作用域会覆盖父级作用域设置的值：

```xml
<ThemeVariantScope RequestedThemeVariant="Dark">
    <!-- 这里的所有内容都使用深色变体 -->
    <StackPanel Spacing="8">
        <Button Content="Dark button" />

        <ThemeVariantScope RequestedThemeVariant="Light">
            <!-- 这个区域切换为浅色 -->
            <Button Content="Light button inside dark scope" />
        </ThemeVariantScope>
    </StackPanel>
</ThemeVariantScope>
```

## 感知主题的资源

定义在 `ThemeDictionaries` 中的资源会响应 `ThemeVariantScope`。每个作用域都会独立解析自己的主题变体，因此同一个 `DynamicResource` 键会根据它所在的作用域返回不同的值：

```xml
<Window.Resources>
    <ResourceDictionary>
        <ResourceDictionary.ThemeDictionaries>
            <ResourceDictionary x:Key="Light">
                <SolidColorBrush x:Key="CardBrush">White</SolidColorBrush>
            </ResourceDictionary>
            <ResourceDictionary x:Key="Dark">
                <SolidColorBrush x:Key="CardBrush">#1E1E1E</SolidColorBrush>
            </ResourceDictionary>
        </ResourceDictionary.ThemeDictionaries>
    </ResourceDictionary>
</Window.Resources>
```

```xml
<ThemeVariantScope RequestedThemeVariant="Light">
    <!-- 使用 White -->
    <Border Background="{DynamicResource CardBrush}" />
</ThemeVariantScope>

<ThemeVariantScope RequestedThemeVariant="Dark">
    <!-- 使用 #1E1E1E -->
    <Border Background="{DynamicResource CardBrush}" />
</ThemeVariantScope>
```

## 在代码中设置主题变体

你可以在代码后置中设置 `RequestedThemeVariant`，从而在运行时切换主题变体：

```csharp
myScope.RequestedThemeVariant = ThemeVariant.Dark;
```

你也可以将该属性绑定到视图模型，让用户能够动态切换主题：

```xml
<ThemeVariantScope RequestedThemeVariant="{Binding SelectedTheme}">
    <ContentControl Content="{Binding CurrentPage}" />
</ThemeVariantScope>
```

```csharp
public class MainViewModel : ViewModelBase
{
    private ThemeVariant _selectedTheme = ThemeVariant.Default;

    public ThemeVariant SelectedTheme
    {
        get => _selectedTheme;
        set => this.RaiseAndSetIfChanged(ref _selectedTheme, value);
    }
}
```

## 另请参阅

- [主题变体](/docs/styling/theme-variants)：浅色/深色主题支持和主题字典完整指南。
- [如何切换主题](/docs/how-to/theme-switching-how-to)：在应用中实现主题切换。
- [资源](/docs/app-development/resources)：资源系统概览。
