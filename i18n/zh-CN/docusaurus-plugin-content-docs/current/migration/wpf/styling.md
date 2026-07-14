---
id: styling
title: 样式系统
description: WPF 与 Avalonia 在选择器、样式类和主题方面的关键样式差异。
doc-type: migration
---

Avalonia 的样式系统是从 WPF 迁移时最大的概念变化之一。它不再沿用 WPF 中基于资源字典的方案，而是采用一种类似 CSS 的样式模型，其中包含选择器、样式类和伪类。本指南将梳理关键差异，并展示各个领域中的实际迁移模式。

## 样式声明

在 WPF 中，样式通常定义为资源，并通过类型或键来引用。而在 Avalonia 中，样式存放在专用的 [`Styles`](/api/avalonia/styling/styles) 集合中，并通过类似 CSS 的选择器来匹配控件。

**WPF：**

```xml
<Window.Resources>
    <Style TargetType="Button">
        <Setter Property="Background" Value="SteelBlue"/>
        <Setter Property="Foreground" Value="White"/>
    </Style>
</Window.Resources>
```

**Avalonia：**

```xml
<Window.Styles>
    <Style Selector="Button">
        <Setter Property="Background" Value="SteelBlue"/>
        <Setter Property="Foreground" Value="White"/>
    </Style>
</Window.Styles>
```

关键差异：

| 方面 | WPF | Avalonia |
|---|---|---|
| 存储位置 | `Resources` 字典 | `Styles` 集合 |
| 匹配机制 | `TargetType` 属性 | `Selector` 属性（类似 CSS） |
| 作用域 | 应用于该资源之下的可视树 | 应用于 `Styles` 拥有者之下的可视树 |
| 继承模型 | 资源查找会沿树向上查找 | 样式按选择器特异性自上而下匹配 |

## 选择器 vs TargetType

WPF 使用 `TargetType` 将样式匹配到控件类型。Avalonia 则使用受 CSS 启发的选择器语法来替代它。选择器可以按类型、类名、名称、属性状态、嵌套结构等方式进行匹配。

**WPF（匹配所有 TextBlock）：**

```xml
<Style TargetType="TextBlock">
    <Setter Property="Foreground" Value="Gray"/>
</Style>
```

**Avalonia（匹配所有 TextBlock）：**

```xml
<Style Selector="TextBlock">
    <Setter Property="Foreground" Value="Gray"/>
</Style>
```

**Avalonia（匹配 StackPanel 中的 TextBlock）：**

```xml
<Style Selector="StackPanel > TextBlock">
    <Setter Property="Foreground" Value="Gray"/>
</Style>
```

**Avalonia（匹配具名控件）：**

```xml
<Style Selector="TextBlock#MyHeader">
    <Setter Property="FontSize" Value="24"/>
</Style>
```

常见选择器模式：

| 选择器 | 含义 |
|---|---|
| `Button` | 所有 Button 控件 |
| `Button.primary` | 带有 `primary` 样式类的 Button |
| `StackPanel > Button` | 作为 StackPanel 直接子元素的 Button |
| `Button:pointerover` | 处于 pointer-over 状态的 Button |
| `Button:not(:disabled)` | 未禁用的 Button |
| `TextBlock#title` | `Name="title"` 的 TextBlock |
| `Button.primary:pointerover` | 处于 pointer-over 状态的主按钮 |

完整选择器参考请参阅 [样式选择器](/docs/styling/style-selectors)。

## 样式类 vs x:Key

在 WPF 中，如果想区分同一控件类型的不同样式，通常会给样式设置 `x:Key`，然后通过 `Style="{StaticResource MyStyle}"` 来引用。Avalonia 则改用**样式类**，它的工作方式很像 CSS class。

**WPF:**

```xml
<Window.Resources>
    <Style x:Key="PrimaryButton" TargetType="Button">
        <Setter Property="Background" Value="SteelBlue"/>
        <Setter Property="Foreground" Value="White"/>
    </Style>
</Window.Resources>

<Button Style="{StaticResource PrimaryButton}" Content="Save"/>
```

**Avalonia:**

```xml
<Window.Styles>
    <Style Selector="Button.primary">
        <Setter Property="Background" Value="SteelBlue"/>
        <Setter Property="Foreground" Value="White"/>
    </Style>
</Window.Styles>

<Button Classes="primary" Content="Save"/>
```

一个控件可以同时拥有多个类，而且这些类还可以动态切换：

```xml
<Button Classes="primary large" Content="Save"/>
```

你也可以在 code-behind 中切换类：

```csharp
myButton.Classes.Add("active");
myButton.Classes.Remove("active");
```

这种方式不再需要管理资源键，并提供了更灵活的组合模型。

## 从触发器迁移到伪类

WPF 在样式内部使用 `Trigger`、`DataTrigger` 和 `EventTrigger`。Avalonia 则通过**伪类**和基于选择器的匹配机制来替代这些能力。

### 从属性触发器到伪类

**WPF（属性触发器）：**

```xml
<Style TargetType="Button">
    <Setter Property="Background" Value="Gray"/>
    <Style.Triggers>
        <Trigger Property="IsMouseOver" Value="True">
            <Setter Property="Background" Value="LightBlue"/>
        </Trigger>
        <Trigger Property="IsPressed" Value="True">
            <Setter Property="Background" Value="DarkBlue"/>
        </Trigger>
    </Style.Triggers>
</Style>
```

**Avalonia（伪类）：**

```xml
<Style Selector="Button">
    <Setter Property="Background" Value="Gray"/>
</Style>
<Style Selector="Button:pointerover">
    <Setter Property="Background" Value="LightBlue"/>
</Style>
<Style Selector="Button:pressed">
    <Setter Property="Background" Value="DarkBlue"/>
</Style>
```

WPF 中常见触发条件与 Avalonia 伪类的对应关系：

| WPF 触发属性 | Avalonia 伪类 |
|---|---|
| `IsMouseOver` | `:pointerover` |
| `IsPressed` | `:pressed` |
| `IsEnabled="False"` | `:disabled` |
| `IsChecked="True"` | `:checked` |
| `IsFocused` | `:focus` |
| `IsSelected` | `:selected` |
| `IsExpanded` | `:expanded` |

完整列表请参阅 [伪类](/docs/styling/pseudoclasses)。

### DataTrigger 的迁移方式

WPF 的 `DataTrigger` 会根据数据绑定值来应用 setter。在 Avalonia 中没有完全直接的一对一等价物，因此需要根据场景采用不同方式。

**方式 1：直接绑定并使用转换器。**

当你只需要根据绑定值改变单个属性时，这种方式最合适：

```xml
<TextBlock Text="{Binding Status}"
           Foreground="{Binding Status, Converter={StaticResource StatusToColorConverter}}"/>
```

**方式 2：使用样式类配合选择器。**

如果你的 ViewModel 暴露了一个可映射到视觉状态的属性，可以在 code-behind 中设置样式类，或者借助行为（behavior）来设置，然后再通过选择器进行匹配：

```xml
<Style Selector="Border.error">
    <Setter Property="BorderBrush" Value="Red"/>
    <Setter Property="BorderThickness" Value="2"/>
</Style>
```

**方式 3：对基于尺寸的触发使用容器查询。**

在 WPF 中，一种很常见的做法是把 `DataTrigger` 绑定到 `ActualWidth` 或 `ActualHeight`（通常会再配合转换器），以便在不同尺寸下调整布局。而 Avalonia 则提供了专门为这种模式设计的容器查询作为替代方案。

**WPF（基于 ActualWidth 的 DataTrigger）：**

```xml
<Style TargetType="UniformGrid">
    <Setter Property="Columns" Value="3"/>
    <Style.Triggers>
        <DataTrigger Binding="{Binding ActualWidth,
                     RelativeSource={RelativeSource AncestorType=Border},
                     Converter={StaticResource LessThanConverter},
                     ConverterParameter=600}"
                     Value="True">
            <Setter Property="Columns" Value="1"/>
        </DataTrigger>
    </Style.Triggers>
</Style>
```

**Avalonia（容器查询）：**

```xml
<Border Container.Name="main" Container.Sizing="Width">
    <Border.Styles>
        <Style Selector="UniformGrid#cards">
            <Setter Property="Columns" Value="3"/>
        </Style>
        <ContainerQuery Name="main" Query="max-width:600">
            <Style Selector="UniformGrid#cards">
                <Setter Property="Columns" Value="1"/>
            </Style>
        </ContainerQuery>
    </Border.Styles>

    <UniformGrid x:Name="cards">
        <!-- content -->
    </UniformGrid>
</Border>
```

容器查询不再需要转换器和 `RelativeSource` 绑定。它们还可以一次性作用于多个属性，并组合宽度与高度条件。完整语法请参阅 [容器查询](/docs/styling/container-queries)，而关于如何构建自适应 UI，请参阅 [响应式布局](/docs/layout/responsive-layouts)。

### 从 EventTrigger 迁移到基于伪类的动画

WPF 中的 `EventTrigger` 会在路由事件发生时启动动画。而在 Avalonia 中，动画通常定义在样式内部，并通过伪类或样式类来激活。

**WPF:**

```xml
<Style TargetType="Border">
    <Style.Triggers>
        <EventTrigger RoutedEvent="MouseEnter">
            <BeginStoryboard>
                <Storyboard>
                    <DoubleAnimation Storyboard.TargetProperty="Opacity"
                                     To="1" Duration="0:0:0.3"/>
                </Storyboard>
            </BeginStoryboard>
        </EventTrigger>
    </Style.Triggers>
</Style>
```

**Avalonia:**

```xml
<Style Selector="Border">
    <Setter Property="Opacity" Value="0.5"/>
    <Setter Property="Transitions">
        <Transitions>
            <DoubleTransition Property="Opacity" Duration="0:0:0.3"/>
        </Transitions>
    </Setter>
</Style>
<Style Selector="Border:pointerover">
    <Setter Property="Opacity" Value="1"/>
</Style>
```

Avalonia 使用 `Transitions` 系统，你需要声明哪些属性应进行动画以及动画持续时间。之后，只要这些属性因为样式或伪类变化而发生改变，动画就会自动触发。

## ControlTheme 与隐式样式

在 WPF 中，隐式样式（即带 `TargetType` 但没有 `x:Key` 的 `Style`）通常用于定义控件的默认外观，其中也包括其 `ControlTemplate`。在 Avalonia 中，这一角色由 [`ControlTheme`](/api/avalonia/styling/controltheme) 承担。

`ControlTheme` 是创建“无外观”控件模板的机制。它存储在 `Resources` 字典中（而不是 `Styles` 集合里），并通过类型进行查找。

**WPF（带模板的隐式样式）：**

```xml
<Style TargetType="Button">
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="Button">
                <Border Background="{TemplateBinding Background}"
                        CornerRadius="4"
                        Padding="{TemplateBinding Padding}">
                    <ContentPresenter HorizontalAlignment="Center"
                                      VerticalAlignment="Center"/>
                </Border>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

**Avalonia（ControlTheme）：**

```xml
<ControlTheme x:Key="{x:Type Button}" TargetType="Button">
    <Setter Property="Template">
        <ControlTemplate>
            <Border Background="{TemplateBinding Background}"
                    CornerRadius="4"
                    Padding="{TemplateBinding Padding}">
                <ContentPresenter HorizontalAlignment="Center"
                                  VerticalAlignment="Center"/>
            </Border>
        </ControlTemplate>
    </Setter>

    <Style Selector="^:pointerover">
        <Setter Property="Background" Value="LightBlue"/>
    </Style>
    <Style Selector="^:pressed">
        <Setter Property="Background" Value="DarkBlue"/>
    </Style>
</ControlTheme>
```

关于 `ControlTheme` 的要点：

- 它存放在 `Resources` 中，而不是 `Styles` 中。
- 它的键通常是 `{x:Type ControlType}`，因此会自动应用到该类型的所有实例。
- `ControlTheme` 内部的嵌套样式使用 `^` 选择器来引用模板化控件自身。
- 与类似 CSS 的 `Style` 不同，`ControlTheme` **不会级联**。同一时间一个控件只会应用一个 `ControlTheme`。

更多细节请参阅 [控件主题](/docs/styling/control-themes)。

## TemplateBinding

WPF 与 Avalonia 都支持使用 `TemplateBinding` 将模板中的元素连接到模板化控件的属性上。不过，两者有一个重要差异：

| 方面 | WPF | Avalonia |
|---|---|---|
| 绑定方向 | 默认双向 | **仅支持 OneWay** |
| 双向替代方案 | 不需要 | 使用普通 `Binding` 并配合 `RelativeSource={RelativeSource TemplatedParent}` |

如果你在 Avalonia 的控件模板中需要双向绑定，请将下面这种写法：

```xml
<!-- 在 Avalonia 中这里只支持 OneWay -->
<TextBox Text="{TemplateBinding SearchText}"/>
```

替换为：

```xml
<!-- 模板中的双向绑定 -->
<TextBox Text="{Binding SearchText, RelativeSource={RelativeSource TemplatedParent}, Mode=TwoWay}"/>
```

## 另请参阅

- [样式](/docs/styling/styles)
- [控件主题](/docs/styling/control-themes)
- [伪类](/docs/styling/pseudoclasses)
- [样式选择器](/docs/styling/style-selectors)
- [容器查询](/docs/styling/container-queries)：基于尺寸的样式机制，可替代 WPF 中依赖 ActualWidth/ActualHeight 的 DataTrigger 模式。
- [响应式布局](/docs/layout/responsive-layouts)：使用容器查询和可回流面板构建自适应布局。
