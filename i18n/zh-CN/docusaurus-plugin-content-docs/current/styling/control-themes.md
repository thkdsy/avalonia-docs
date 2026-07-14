---
id: control-themes
title: 控件主题
---

import StylingEllipseButtonScreenshot from '/img/concepts/ui-concepts/styling/ellipse-button.png';

控件主题建立在 [styles](/docs/styling/styles) 之上，用于为控件创建可切换的主题。与普通样式不同，普通样式一旦应用就会叠加并且无法移除；而控件主题则可以被整体替换。因此，当你需要为某个特定实例或某个 UI 区域切换控件的完整外观时，控件主题就是更合适的选择。

控件主题本质上也是样式，但它有几个重要区别：

- 控件主题没有 selector；取而代之的是一个 `TargetType` 属性，用来说明它作用于哪种控件
- 控件主题存放在 `ResourceDictionary` 中，而不是 `Styles` 集合里
- 控件主题通过设置控件的 `Theme` 属性来应用，通常会配合 `{StaticResource}` 标记扩展使用

:::tip
由于控件主题建立在样式之上，因此先理解 Avalonia 的[样式系统](/docs/styling/styles)会非常重要。
:::

:::info
控件主题通常用于 [templated（lookless）](/docs/custom-controls) 控件，但实际上它也可以应用到任何控件。不过，对于非模板控件来说，通常直接使用普通样式会更方便。
:::

## 示例：圆形按钮

下面的示例展示了一个简单的 `Button` 主题，它会把按钮显示成带椭圆背景的样式，带有一种 90 年代 Geocities 风格的复古感：

```xml title="App.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="AvaloniaApplication.App">
  <Application.Styles>
    <FluentTheme />
  </Application.Styles>

  <Application.Resources>
    // highlight-start
    <ControlTheme x:Key="EllipseButton" TargetType="Button">
      <Setter Property="Background" Value="Blue"/>
      <Setter Property="Foreground" Value="Yellow"/>
      <Setter Property="Padding" Value="8"/>
      <Setter Property="Template">
        <ControlTemplate>
          <Panel>
            <Ellipse Fill="{TemplateBinding Background}"
                     HorizontalAlignment="Stretch"
                     VerticalAlignment="Stretch"/>
            <ContentPresenter x:Name="PART_ContentPresenter"
                              Content="{TemplateBinding Content}"
                              Margin="{TemplateBinding Padding}"/>
          </Panel>
        </ControlTemplate>
      </Setter>
    </ControlTheme>
    // highlight-end
  </Application.Resources>
</Application>
```

```xml title='MainWindow.xaml'
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x='http://schemas.microsoft.com/winfx/2006/xaml'
        x:Class="Sandbox.MainWindow">
  // highlight-start
  <Button Theme="{StaticResource EllipseButton}"
          HorizontalAlignment="Center"
          VerticalAlignment="Center">
    Hello World!
  </Button>
  // highlight-end
</Window>
```

<Image light={StylingEllipseButtonScreenshot} alt="Ellipse button" position="center" maxWidth={400} cornerRadius="true" />

## 控件主题中的交互状态

和普通样式一样，控件主题也支持 [nested styles](/docs/styling/styles)，可以用来添加例如悬停和按下等交互状态。

## 示例：圆形按钮的悬停状态

借助嵌套样式，我们可以让按钮在鼠标悬停时改变颜色：

```xml title="App.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="AvaloniaApplication.App">
  <Application.Styles>
    <FluentTheme />
  </Application.Styles>

  <Application.Resources>
    <ControlTheme x:Key="EllipseButton" TargetType="Button">
      <Setter Property="Background" Value="Blue"/>
      <Setter Property="Foreground" Value="Yellow"/>
      <Setter Property="Padding" Value="8"/>
      <Setter Property="Template">
        <ControlTemplate>
          <Panel>
            <Ellipse Fill="{TemplateBinding Background}"
                     HorizontalAlignment="Stretch"
                     VerticalAlignment="Stretch"/>
            <ContentPresenter x:Name="PART_ContentPresenter"
                              Content="{TemplateBinding Content}"
                              Margin="{TemplateBinding Padding}"/>
          </Panel>
        </ControlTemplate>
      </Setter>
      
      // highlight-start
      <Style Selector="^:pointerover">
        <Setter Property="Background" Value="Red"/>
        <Setter Property="Foreground" Value="White"/>
      </Style>
      // highlight-end
    </ControlTheme>
  </Application.Resources>
</Application>
```

## 控件主题查找规则

控件主题有两种查找方式：

- 如果控件的 `Theme` 属性已设置，那么就直接使用该控件主题；否则
- Avalonia 会沿逻辑树向上查找一个 `ControlTheme` 资源，其 `x:Key` 与控件的 [style key](/docs/styling/styles) 相匹配

:::tip
如果你发现 Avalonia 无法找到你的主题，请确认控件返回的 [style key](/docs/styling/styles) 与控件主题的 `x:Key` 和 `TargetType` 相匹配。
:::

这实际上意味着，在定义控件主题时你有两种选择：

- **如果你希望控件主题应用到该控件的所有实例上**，那么请使用 `{x:Type}` 作为资源键。例如
  `<ControlTheme x:Key="{x:Type Button}" TargetType="Button">`
- **如果你只希望控件主题应用到部分特定实例上**，那么可以使用任意其他资源键，并通过 `{StaticResource}` 来查找该资源。通常这个键会是一个 `string`

:::info
这也意味着：任意时刻，一个控件只能应用一个控件主题。
:::

## 示例：让所有按钮都变成圆形

如果你想把这个控件主题应用到应用中的所有按钮上，只需要把控件主题的 `x:Key` 改成与 `Button` 类型匹配即可。

```xml title="App.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="AvaloniaApplication.App">
  <Application.Styles>
    <FluentTheme />
  </Application.Styles>

  <Application.Resources>
      // highlight-next-line
    <ControlTheme x:Key="{x:Type Button}" TargetType="Button">
      <Setter Property="Background" Value="Blue"/>
      <Setter Property="Foreground" Value="Yellow"/>
      <Setter Property="Padding" Value="8"/>
      <Setter Property="Template">
        <ControlTemplate>
          <Panel>
            <Ellipse Fill="{TemplateBinding Background}"
                     HorizontalAlignment="Stretch"
                     VerticalAlignment="Stretch"/>
            <ContentPresenter x:Name="PART_ContentPresenter"
                              Content="{TemplateBinding Content}"
                              Margin="{TemplateBinding Padding}"/>
          </Panel>
        </ControlTemplate>
      </Setter>
      
      <Style Selector="^:pointerover">
        <Setter Property="Background" Value="Red"/>
        <Setter Property="Foreground" Value="White"/>
      </Style>
    </ControlTheme>
  </Application.Resources>
</Application>
```

## TargetType

`ControlTheme.TargetType` 属性用于指定 setter 属性将作用于哪一种类型。如果你没有显式指定 `TargetType`，那么在 `Setter` 中就必须通过 `Property="ClassName.Property"` 这种语法给属性加上类名前缀。例如，不能只写 `FontSize`，而必须写成 `TextBlock.FontSize` 或 `Control.FontSize`。

## 另请参阅

- [ButtonCustomize](https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/ButtonCustomize) sample with a `WinClassicButtonTheme`
- Avalonia 内置控件的控件主题：
  - [Simple Theme](https://github.com/AvaloniaUI/Avalonia/tree/master/src/Avalonia.Themes.Simple/Controls)
  - [Fluent Theme](https://github.com/AvaloniaUI/Avalonia/tree/master/src/Avalonia.Themes.Fluent/Controls)
- [Styles](/docs/styling/styles)
- [Control template walkthrough](/docs/styling/control-template-walkthrough)
