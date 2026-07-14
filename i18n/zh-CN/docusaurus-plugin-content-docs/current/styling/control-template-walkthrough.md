---
id: control-template-walkthrough
title: 控件模板实战讲解
---

本教程将从零开始构建一个完整的控件模板，并解释其中的每个组成部分。完成后，你将理解如何为任意 Avalonia 控件重新定义模板。

## 前提条件

- 一个包含 `Window` 的 Avalonia 项目，以便你添加样式和控件。
- 熟悉 [styles](/docs/styling/styles) 与 [XAML](/docs/fundamentals/avalonia-xaml) 的基础知识。

## 什么是控件模板？

控件模板定义了控件的视觉结构。每个 Avalonia 控件都有一个由主题提供的默认模板。你可以完全替换这个模板，从而改变控件外观，同时保留其行为。

## 第 1 步：创建一个基础按钮模板

先从一个最小化的按钮模板开始，它只负责渲染内容：

```xml
<Window.Styles>
    <Style Selector="Button.custom">
        <Setter Property="Template">
            <ControlTemplate>
                <Border Background="{TemplateBinding Background}"
                        BorderBrush="{TemplateBinding BorderBrush}"
                        BorderThickness="{TemplateBinding BorderThickness}"
                        CornerRadius="{TemplateBinding CornerRadius}"
                        Padding="{TemplateBinding Padding}">
                    <ContentPresenter Content="{TemplateBinding Content}"
                                      ContentTemplate="{TemplateBinding ContentTemplate}"
                                      HorizontalContentAlignment="{TemplateBinding HorizontalContentAlignment}"
                                      VerticalContentAlignment="{TemplateBinding VerticalContentAlignment}" />
                </Border>
            </ControlTemplate>
        </Setter>
        <Setter Property="Background" Value="#6366F1" />
        <Setter Property="Foreground" Value="White" />
        <Setter Property="BorderThickness" Value="0" />
        <Setter Property="CornerRadius" Value="6" />
        <Setter Property="Padding" Value="16,8" />
        <Setter Property="HorizontalContentAlignment" Value="Center" />
    </Style>
</Window.Styles>

<Button Classes="custom" Content="Click Me" />
```

### 关键概念

- **`ControlTemplate`** 定义了一个视觉树，用来替换控件的默认外观。
- **`TemplateBinding`** 绑定到模板父级（也就是 Button）的属性。它比 `{Binding RelativeSource={RelativeSource TemplatedParent}}` 更高效，但只支持 `OneWay`。
- **`ContentPresenter`** 负责显示按钮的 `Content` 属性。没有它，按钮内容将无法显示。

## 第 2 步：使用伪类添加视觉状态

Avalonia 使用伪类（类似 CSS）而不是 WPF 的 VisualStateManager。下面添加一些交互状态：

```xml
<Style Selector="Button.custom">
    <Setter Property="Template">
        <ControlTemplate>
            <Border x:Name="PART_Border"
                    Background="{TemplateBinding Background}"
                    BorderBrush="{TemplateBinding BorderBrush}"
                    BorderThickness="{TemplateBinding BorderThickness}"
                    CornerRadius="{TemplateBinding CornerRadius}"
                    Padding="{TemplateBinding Padding}">
                <ContentPresenter Content="{TemplateBinding Content}"
                                  ContentTemplate="{TemplateBinding ContentTemplate}"
                                  HorizontalContentAlignment="{TemplateBinding HorizontalContentAlignment}"
                                  VerticalContentAlignment="{TemplateBinding VerticalContentAlignment}" />
            </Border>
        </ControlTemplate>
    </Setter>

    <!-- Default state -->
    <Setter Property="Background" Value="#6366F1" />
    <Setter Property="Foreground" Value="White" />
    <Setter Property="BorderThickness" Value="0" />
    <Setter Property="CornerRadius" Value="6" />
    <Setter Property="Padding" Value="16,8" />
    <Setter Property="HorizontalContentAlignment" Value="Center" />
</Style>

<!-- Hover state -->
<Style Selector="Button.custom:pointerover">
    <Setter Property="Background" Value="#818CF8" />
</Style>

<!-- Pressed state -->
<Style Selector="Button.custom:pressed">
    <Setter Property="Background" Value="#4F46E5" />
</Style>

<!-- Disabled state -->
<Style Selector="Button.custom:disabled">
    <Setter Property="Background" Value="#C7D2FE" />
    <Setter Property="Foreground" Value="#9CA3AF" />
</Style>

<!-- Focused state -->
<Style Selector="Button.custom:focus-visible">
    <Setter Property="BorderBrush" Value="White" />
    <Setter Property="BorderThickness" Value="2" />
</Style>
```

### 常见伪类

| 伪类 | 激活时机 |
|---|---|
| `:pointerover` | 指针位于控件上方 |
| `:pressed` | 控件正处于按下状态 |
| `:disabled` | 控件已禁用（`IsEnabled="False"`） |
| `:focus` | 控件拥有键盘焦点 |
| `:focus-visible` | 控件因键盘导航而拥有焦点（不是鼠标点击） |
| `:checked` | ToggleButton/CheckBox/RadioButton 已选中 |
| `:unchecked` | ToggleButton/CheckBox/RadioButton 未选中 |
| `:selected` | 项目已被选中（例如 ListBoxItem） |

## 第 3 步：添加动画

使用 `Transitions` 属性让状态之间平滑过渡：

```xml
<Style Selector="Button.custom">
    <!-- ...template and setters from above... -->
    <Setter Property="Transitions">
        <Transitions>
            <BrushTransition Property="Background" Duration="0:0:0.15" />
            <BrushTransition Property="BorderBrush" Duration="0:0:0.15" />
            <ThicknessTransition Property="BorderThickness" Duration="0:0:0.15" />
        </Transitions>
    </Setter>
</Style>
```

现在，背景颜色会在悬停、按下和普通状态之间平滑过渡。

## 第 4 步：使用模板部件

对于更复杂的模板，可以按照 `PART_` 约定为内部元素命名。控件的代码后置逻辑可以定位并操作这些部件：

```xml
<ControlTemplate>
    <Grid>
        <Border x:Name="PART_Background"
                Background="{TemplateBinding Background}"
                CornerRadius="{TemplateBinding CornerRadius}" />

        <Border x:Name="PART_Highlight"
                Background="White" Opacity="0"
                CornerRadius="{TemplateBinding CornerRadius}" />

        <ContentPresenter x:Name="PART_ContentPresenter"
                          Content="{TemplateBinding Content}"
                          Margin="{TemplateBinding Padding}"
                          HorizontalContentAlignment="{TemplateBinding HorizontalContentAlignment}"
                          VerticalContentAlignment="{TemplateBinding VerticalContentAlignment}" />
    </Grid>
</ControlTemplate>
```

之后你就可以在伪类样式中针对这些部件进行设置：

```xml
<Style Selector="Button.custom:pointerover /template/ Border#PART_Highlight">
    <Setter Property="Opacity" Value="0.1" />
</Style>

<Style Selector="Button.custom:pressed /template/ Border#PART_Highlight">
    <Setter Property="Opacity" Value="0.2" />
</Style>
```

`/template/` 选择器会进入控件模板的视觉树。`#PART_Highlight` 则按名称进行匹配。

## 第 5 步：整合完整模板

下面是一个较为完整的自定义按钮模板：

```xml
<Window.Styles>
    <Style Selector="Button.pill">
        <Setter Property="Background" Value="#6366F1" />
        <Setter Property="Foreground" Value="White" />
        <Setter Property="BorderThickness" Value="0" />
        <Setter Property="CornerRadius" Value="999" />
        <Setter Property="Padding" Value="20,10" />
        <Setter Property="HorizontalContentAlignment" Value="Center" />
        <Setter Property="Cursor" Value="Hand" />
        <Setter Property="Transitions">
            <Transitions>
                <BrushTransition Property="Background" Duration="0:0:0.2" />
                <TransformOperationsTransition Property="RenderTransform" Duration="0:0:0.1" />
            </Transitions>
        </Setter>
        <Setter Property="RenderTransform" Value="scale(1)" />
        <Setter Property="Template">
            <ControlTemplate>
                <Border Background="{TemplateBinding Background}"
                        CornerRadius="{TemplateBinding CornerRadius}"
                        Padding="{TemplateBinding Padding}"
                        BoxShadow="0 2 4 0 #20000000">
                    <ContentPresenter Content="{TemplateBinding Content}"
                                      ContentTemplate="{TemplateBinding ContentTemplate}"
                                      HorizontalContentAlignment="{TemplateBinding HorizontalContentAlignment}"
                                      VerticalContentAlignment="{TemplateBinding VerticalContentAlignment}" />
                </Border>
            </ControlTemplate>
        </Setter>
    </Style>

    <Style Selector="Button.pill:pointerover">
        <Setter Property="Background" Value="#818CF8" />
    </Style>

    <Style Selector="Button.pill:pressed">
        <Setter Property="Background" Value="#4F46E5" />
        <Setter Property="RenderTransform" Value="scale(0.97)" />
    </Style>

    <Style Selector="Button.pill:disabled">
        <Setter Property="Background" Value="#E5E7EB" />
        <Setter Property="Foreground" Value="#9CA3AF" />
    </Style>
</Window.Styles>

<StackPanel Spacing="12" Margin="20">
    <Button Classes="pill" Content="Primary Action" />
    <Button Classes="pill" Content="Disabled" IsEnabled="False" />
</StackPanel>
```

## 验证结果

运行应用后，你应该能看到一个带紫色背景的胶囊形按钮。将鼠标悬停到按钮上时，背景颜色会变浅；按下时，颜色会变深并略微缩小；禁用按钮应显示为灰色。如果过渡设置正确，颜色变化会以动画形式平滑切换，而不是瞬间跳变。

## 自定义模板建议

- 始终使用 `TemplateBinding` 绑定 `Padding`、`Background`、`BorderBrush`、`BorderThickness` 和 `CornerRadius`，这样模板才能正确响应外部设置的属性值。
- 内容控件使用 `ContentPresenter`，项控件使用 `ItemsPresenter`。
- 状态管理优先使用伪类选择器，而不是触发器。
- 模板部件建议使用 `PART_` 前缀命名，便于理解和代码访问。
- 使用 `Transitions` 实现平滑状态切换，而不是依赖离散的 setter 变化。
- 同时在浅色和深色主题下测试，确保模板能正确适配不同主题变体。

## 另请参阅

- [Control themes](/docs/styling/control-themes)：了解主题如何为所有控件使用模板。
- [Style selectors](/docs/styling/style-selectors)：用于匹配控件和状态的选择器语法。
- [Pseudo-classes](/docs/styling/pseudoclasses)：所有可用伪类的说明。
