---
id: templated-controls
title: 如何创建模板控件
description: 使用控件主题、模板部件和伪类构建无外观模板控件。
doc-type: how-to
---

模板控件是指其外观完全由 [`ControlTemplate`](/api/avalonia/markup/xaml/templates/controltemplate) 定义的控件。这种方式将控件的视觉结构与行为分离，使开发者和设计师能够在不修改逻辑的前提下重新设置控件样式。如果你熟悉 WPF，这类控件有时也被称为“无外观（lookless）控件”，因为控件类本身不包含渲染代码。

Avalonia 的内置控件（例如 `Button`、`TextBox` 和 `ListBox`）都是模板控件。你也可以按照同样的模式构建自己的控件。

## 创建模板控件

要创建模板控件，请定义一个继承自 `TemplatedControl` 的类，并使用 `StyledProperty` 注册所需的自定义属性。

```csharp
public class ToggleLabel : TemplatedControl
{
    public static readonly StyledProperty<string> LabelTextProperty =
        AvaloniaProperty.Register<ToggleLabel, string>(nameof(LabelText), "Default");

    public string LabelText
    {
        get => GetValue(LabelTextProperty);
        set => SetValue(LabelTextProperty, value);
    }
}
```

这样你就得到了一个具有 `LabelText` 属性、但尚未具备可视外观的控件。它的视觉表现将由控件主题提供。

:::caution 不要设置 DataContext = this
不要在自定义控件的构造函数中写 `DataContext = this`。这样会覆盖控件使用者原本期望从父级可视树继承而来的 `DataContext`。外部写在你的控件上的绑定（例如 `<MyControl Items="{Binding SelectedItems}" />`）会针对你的控件类型本身进行解析，而不是针对父级的 ViewModel，进而导致静默的绑定失败。

模板控件不需要自引用的 `DataContext`。应在控件模板内部使用 [`TemplateBinding`](#templatebinding-details) 访问控件自身属性，并让 `DataContext` 自然地从父级流入。
:::

## 定义控件主题

每个模板控件都需要一个默认的 `ControlTheme`，其中包含它的 `ControlTemplate`。通常这会放在诸如 `Themes/Generic.axaml` 这样的资源字典中，并包含到应用资源里。

```xml
<ControlTheme x:Key="{x:Type local:ToggleLabel}" TargetType="local:ToggleLabel">
    <Setter Property="Template">
        <ControlTemplate>
            <Border Background="{TemplateBinding Background}" Padding="8">
                <TextBlock Text="{TemplateBinding LabelText}" />
            </Border>
        </ControlTemplate>
    </Setter>
</ControlTheme>
```

关键点：

- `x:Key="{x:Type local:ToggleLabel}"` 可确保 Avalonia 自动将该主题应用到所有 `ToggleLabel` 实例上。
- `TargetType` 会限制主题的作用类型，从而让属性设置器和模板绑定都针对正确的类型解析。
- 在 `ControlTemplate` 内部，使用 [`TemplateBinding`](/api/avalonia/data/templatebinding) 绑定到模板控件本身的属性。

## 模板部件

有时模板控件需要与模板内部的特定元素交互。按照惯例，这些元素名称会使用 `PART_` 前缀。

可以重写 `OnApplyTemplate`，在模板应用完成后查找这些具名部件：

```csharp
protected override void OnApplyTemplate(TemplateAppliedEventArgs e)
{
    base.OnApplyTemplate(e);
    var button = e.NameScope.Find<Button>("PART_Button");
    if (button is not null)
    {
        button.Click += OnButtonClick;
    }
}
```

由于用户可以替换控件模板，因此在查找模板部件时必须始终检查 `null`。自定义模板可能会省略默认模板中存在的某些部件。

## TemplateBinding 细节

当你创建控件模板并希望绑定到模板父级时，可以这样写：

```xml
<TextBlock Name="tb" Text="{TemplateBinding Caption}"/>

<!-- 等价写法 -->
<TextBlock Name="tb" Text="{Binding Caption, RelativeSource={RelativeSource TemplatedParent}}"/>
```

虽然这里展示的两种语法在大多数情况下等价，但它们仍有一些区别：

1. `TemplateBinding` 只接受单个属性，而不支持属性路径，因此如果你想绑定属性路径，就必须使用第二种写法：

    ```xml
    <!-- 这样不能工作，因为 TemplateBinding 只接受单个属性 -->
    <TextBlock Name="tb" Text="{TemplateBinding Caption.Length}"/>

    <!-- 此时必须改用这种语法 -->
    <TextBlock Name="tb" Text="{Binding Caption.Length, RelativeSource={RelativeSource TemplatedParent}}"/>
    ```
2. 出于性能原因，`TemplateBinding` 只支持 `OneWay` 模式（这与 [WPF 相同](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/advanced/templatebinding-markup-extension#remarks)）。这意味着 `TemplateBinding` 实际上等价于 `{Binding RelativeSource={RelativeSource TemplatedParent}, Mode=OneWay}`。如果控件模板中需要 `TwoWay` 绑定，就必须使用下面所示的完整语法。注意，`Binding` 与 `TemplateBinding` 不同，它还会使用默认绑定模式。

    ```xml
    {Binding RelativeSource={RelativeSource TemplatedParent}, Mode=TwoWay}
    ```
3. `TemplateBinding` 只能用于 `StyledElement`。

```xml
<!-- 这样不能工作，因为 GeometryDrawing 不是 StyledElement。 -->
<GeometryDrawing Brush="{TemplateBinding Foreground}"/>

<!-- 此时必须改用这种语法。 -->
<GeometryDrawing Brush="{Binding Foreground, RelativeSource={RelativeSource TemplatedParent}}"/>
```

## 伪类

模板控件可以通过伪类暴露视觉状态。这使主题作者无需访问代码后置，就能根据控件状态应用不同样式。

你可以在控件逻辑中设置伪类：

```csharp
PseudoClasses.Set(":active", isActive);
```

然后在控件主题中对其进行匹配：

```xml
<Style Selector="^:active">
    <Setter Property="Background" Value="Blue" />
</Style>
```

## 另请参阅

- [定义属性](/docs/custom-controls/defining-properties)
- [定义事件](/docs/custom-controls/defining-events)
- [控件主题](/docs/styling/control-themes)
- [控件模板实战讲解](/docs/styling/control-template-walkthrough)
