---
id: property-setters
title: 属性 Setter
description: 学习如何在样式中使用 setter、绑定和模板来定义属性值，并理解 setter 的优先级规则。
doc-type: reference
---

import SetterPrecedenceAnimationWrongScreenshot from '/img/reference/styles/setter-precedence-animation-wrong.gif';
import SetterPrecedenceAnimationCorrectScreenshot from '/img/reference/styles/setter-precedence-animation-correct.gif';

属性 setter 用于定义：当 Avalonia 通过选择器匹配到某个控件后，该样式应为其应用哪些属性值。

## 基础用法

Setter 是以属性和值成对出现的 XAML 写法，格式如下：

```xml
<Setter Property="propertyName" Value="newValueString"/>
```

例如：

```xml
<Setter Property="FontSize" Value="24"/>
<Setter Property="Padding" Value="4 2 0 4"/>
```

你也可以使用长写法，把某个控件属性设置为一个带有多个属性值的对象，如下所示：

```xml
<Setter Property="MyProperty">
   <MyObject Property1="My Value" Property2="999"/>
</Setter>
```

样式同样可以通过绑定来设置属性。在正常完成选择过程后，Avalonia 会从目标控件的 data context 中取值。例如：

```xml
<Setter Property="FontSize" Value="{Binding SelectedFontSize}"/>
```

:::caution
setter 中的绑定始终是针对**目标控件的** `DataContext` 进行解析的。即使样式定义在 `<Application.Styles>` 中，它也仍然会绑定到被匹配控件的 `DataContext`，而不是 `Application` 自己设置的某个 `DataContext`。这是因为 `Application` 本身不属于视觉树或逻辑树，所以在它上面设置 `DataContext` 不会对 setter 绑定产生影响。

如果你需要在应用层级应用某些可配置值（例如用户选择的颜色），请优先使用 `DynamicResource` 配合运行时资源更新，而不是使用数据绑定。详情请参阅 [Resources overview](/docs/app-development/resources) 和 [How to switch themes](/docs/how-to/theme-switching-how-to)。
:::

## 样式优先级

当一个选择器同时匹配多个样式时，决定哪个属性 setter 优先生效的规则有两条：

* 样式集合在应用结构中的位置——越“接近”控件的优先级越高。
* 样式在该集合中的书写位置——越“靠后”定义的优先级越高。

例如，这首先意味着：定义在窗口级别的样式会覆盖定义在应用级别的样式。其次，如果被选中的样式集合位于同一层级，那么文件中写得更靠后的定义具有更高优先级。

:::caution
如果你拿样式类去类比 CSS，需要特别注意：**与 CSS 不同**，Avalonia 中 `Classes` 属性里类名的排列顺序不会影响 setter 的优先级。也就是说，如果下面这两个样式类都会设置颜色，那么无论怎么排列，结果都一样：

```xml
<Button Classes="h1 blue"/>
<Button Classes="blue h1"/>
```
:::

## 值回退

只要某个样式匹配到控件，它内部所有 setter 都会应用到该控件上。如果之后由于选择器条件变化，导致该样式不再匹配这个控件，那么相关属性值就会回退到下一个更高优先级来源提供的值。

## 可变值

[`Setter`](/api/avalonia/styling/setter) 会创建一个单独的 `Value` 实例，并把它应用到所有匹配该样式的控件上。如果这个对象本身是可变的，那么对它的修改会反映到所有这些控件上。

如果 setter 的值本身是一个对象，并且这个对象内部又带有绑定，那么它将无法访问目标控件的 data context，因为可能会有多个目标控件共用同一个 setter 值实例。下面这种样式就可能出现这种情况：

```xml
<Style Selector="local|MyControl">
  <Setter Property="MyProperty">
     <MyObject Property1="{Binding MyViewModelProperty}"/>
  </Setter>
</Style>
```

这意味着：在上面的例子中，setter 内部绑定的源实际上会是 `MyObject.DataContext`，而不是 `MyControl.DataContext`。如果 `MyObject` 自身没有 data context，那么这个绑定就无法生成值。

如果你使用的是编译绑定，那么就需要在 `<Style>` 元素上显式指定绑定源的数据类型：

```xml
<Style Selector="MyControl" x:DataType="MyViewModelClass">
  <Setter Property="ControlProperty" Value="{Binding MyViewModelProperty}" />
</Style>
```

## Setter 中的数据模板

如前所述，如果你在 setter 中没有使用**数据模板**，那么它只会创建一个 setter 值实例，并在所有匹配控件之间共享。如果你希望根据模板来生成不同值，那么应把目标控件内容放进一个模板元素中，如下所示：

```xml
<Style Selector="Border.empty">
  <Setter Property="Child">
    <Template>
      <TextBlock>No content available.</TextBlock>
    </Template>
  </Setter>
</Style>
```

## Setter 优先级

Avalonia 的 `Setter` 会先按 [`BindingPriority`](/api/avalonia/data/bindingpriority) 排序，然后再考虑视觉树中的就近性，最后再考虑 `Styles` 集合中的顺序。这个优先级规则是针对每个 `StyledProperty` 单独生效的，因此样式系统可以进行组合。`DirectProperty` 和普通 CLR 属性无法被样式化，所以它们不参与这套优先级规则。

## BindingPriority 值

```csharp
Animation = -1, // Highest priority
LocalValue = 0,
StyleTrigger,
Template,
Style,
Inherited,
Unset = int.MaxValue, // Lowest priority
```

## XAML 中的 BindingPriority 分配

`BindingPriority` 不能在 XAML 中显式设置。下面的示例展示了在不同场景下它是如何被隐式分配的。这对于设计和排查样式行为非常关键。

### 动画

`Animation` 拥有最高的 `BindingPriority`，它会应用到 `Keyframe` 中的 `Setter`，以及更广义上的过渡系统中。

```xml
<Button Background="Green" Content="Bounces from Red to Blue">
    <Button.Styles>
        <Style Selector="Button">
            <Style.Animations>
                <Animation IterationCount="Infinite" Duration="0:0:2">
                    <KeyFrame Cue="0%">
                        <Setter Property="Background" Value="Red" />
                    </KeyFrame>
                    <KeyFrame Cue="100%">
                        <Setter Property="Background" Value="Blue" />
                    </KeyFrame>
                </Animation>
            </Style.Animations>
        </Style>
    </Button.Styles>
</Button>
```

### 本地值

当某个 XAML 属性在 [`ControlTemplate`](/api/avalonia/markup/xaml/templates/controltemplate) 之外被直接设置时，它会被赋予 `LocalValue` 优先级。下面两个 `Background` 的写法都属于 `LocalValue` 优先级。

```xml
<Button Background="Orange" />
<Button Background="{DynamicResource ButtonBrush}" />
```

:::tip

资源标记扩展不会影响优先级本身。

:::

### StyleTrigger

当某个 `Selector` 具有条件激活特性时，相关 `Setter` 的 `BindingPriority` 会从 `Style` 提升为 `StyleTrigger`。只要选择器中存在任意条件激活部分，那么这些选择器之间的优先级就是相同的，不取决于激活器数量，也不取决于它在选择器中的位置。Avalonia 并没有 CSS 那种 Specificity（特异性）概念。

```xml
<Style Selector="Button:pointerover /template/ ContentPresenter#PART_ContentPresenter">
    <Setter Property="Background" Value="Orange" />
</Style>
```

:::tip

样式类、伪类、子元素位置以及属性匹配选择器都属于条件选择器；而按控件名选择并不属于条件选择器。

:::

### 模板

当某个属性是在 `ControlTemplate` 内部被直接设置时，它就具有 `Template` 优先级。下面示例中的 `BorderThickness`、`Background` 和 `Padding` 都属于这一类。

```xml
<ControlTemplate>
    <Border BorderThickness="2">
        <Button Background="{DynamicResource ButtonBrush}" Padding="{TemplateBinding Padding}" />
    </Border>
</ControlTemplate>
```
### 样式

当某个 `Setter` 定义在一个不带条件激活的 `Style` 中时，它会使用 `Style` 优先级。

```xml
<Style Selector="Button /template/ ContentPresenter#PART_ContentPresenter">
    <Setter Property="Background" Value="Orange" />
</Style>
```

:::tip

特别需要注意的是：它的优先级低于 `Template`。因此，这类选择器不能用来覆盖前面 `Template` 示例中直接设置的那些属性。

:::

### 继承

当某个属性没有被设置时，它可能会从父级继承属性值。这需要在属性注册阶段指定，或者通过 `OverrideMetadata` 来设置。

```csharp
public static readonly StyledProperty<bool> UseLayoutRoundingProperty =
    AvaloniaProperty.Register<Layoutable, bool>(
        nameof(UseLayoutRounding),
        defaultValue: true,
        inherits: true);
```

## 视觉树中的就近性

当多个 `Setter` 具有相同的 `BindingPriority` 时，就会根据它们相对于目标 `Control` 在视觉树中的位置来决定优先级。向上遍历所需节点最少的那个 `Setter` 会胜出。对于这一步来说，内联样式中的 `Setter` 具有最高优先级。

```xml
<Window>
    <Window.Styles>
        <Style Selector="Button">
            <Setter Property="FontSize" Value="16" />
            <Setter Property="Foreground" Value="Red" />
        </Style>
    </Window.Styles>
    <StackPanel>
        <StackPanel.Styles>
            <Style Selector="Button">
                <Setter Property="FontSize" Value="24" />
            </Style>
        </StackPanel.Styles>

        <Button Content="This Has FontSize=24 with Foreground=Red" />
    </StackPanel>
</Window>
```

## Styles 集合顺序

当 `BindingPriority` 和视觉树中的就近性都相同时，最终决定因素就是 `Styles` 集合中的顺序。最后一个适用的 `Setter` 会生效。

```xml
<StackPanel>
    <StackPanel.Styles>
        <Style Selector="Button.small">
            <Setter Property="FontSize" Value="12" />
        </Style>
        <Style Selector="Button.big">
            <Setter Property="FontSize" Value="24" />
        </Style>
    </StackPanel.Styles>

    <Button Classes="small big" Content="This Has FontSize=24" />
    <Button Classes="big small" Content="This Also Has FontSize=24" />
</StackPanel>
```

:::info

这些 Button 虽然以不同顺序声明了 `Classes`，但这不会影响 Setter 的优先级。

:::

## BindingPriority 不会传播

回顾上面的 `Animation` 示例。当你把鼠标悬停上去时，带动画的背景会被静态背景替换掉，尽管 `BindingPriority.Animation` 拥有最高优先级。原因在于该 `Selector` 作用到了错误的 `Control`。要找出问题原因，就必须检查对应的 `ControlTheme`。

<Image light={SetterPrecedenceAnimationWrongScreenshot} alt="Button animation overridden by pointer-over style due to incorrect selector target" position="center" maxWidth={400} cornerRadius="true"/>

```xml title='ControlTheme for Button, Trimmed'
<ControlTheme x:Key="{x:Type Button}" TargetType="Button">
    <Setter Property="Background" Value="{DynamicResource ButtonBackground}"/>
    <Setter Property="Template">
        <ControlTemplate>
            <ContentPresenter x:Name="PART_ContentPresenter"
                              Background="{TemplateBinding Background}"/>
        </ControlTemplate>
    </Setter>

    <Style Selector="^:pointerover /template/ ContentPresenter#PART_ContentPresenter">
        <Setter Property="Background" Value="{DynamicResource ButtonBackgroundPointerOver}"/>
    </Style>
</ControlTheme>
```

最上面的 `Setter` 以 `Style` 优先级将 `ButtonBackground` 应用到 [`Button`](/api/avalonia/controls/button) 上。真正负责渲染 `Background` 的是 `ContentPresenter`，它具有 `Template` 优先级。它会读取已经应用到 `Button` 上的 `ButtonBackground`。

但是，当 `Button` 被悬停时，`:pointerover` 选择器会以 `StyleTrigger` 优先级激活，覆盖 `TemplateBinding`，并改为读取 `ButtonBackgroundPointerOver`。这样一来，就绕过了原始 `Animation` 选择器所针对的 `Button.Background`。下表对这一过程进行了总结：

| Background Setters and Styles While Hovered                         | Priority                          | Location        |
|---------------------------------------------------------------------|-----------------------------------|-----------------|
| ~~Background="Green"~~                                              | LocalValue                        | Button          |
| Background="Red"                                                    | Animation (Overrides LocalValue)  | Keyframe        |
| ~~`<ContentPresenter Background="{TemplateBinding Background}"/>`~~ | Template                          | ControlTemplate |
| `^:pointerover /template/ ContentPresenter#PART_ContentPresenter`   | StyleTrigger (Overrides Template) | ControlTheme    |

正确做法是：将 `Setter` 直接作用到 `ContentPresenter` 上，并确保其优先级至少达到 `StyleTrigger`。`BindingPriority.Animation` 就满足这个要求。这个结论只有在检查原始 `ControlTemplate` 后才能得出，它也说明了仅仅依赖优先级本身，并不足以有效地控制应用样式。

```xml title='Corrected to override :pointerover priority'
<Button Background="Green" Content="Bounces from Red to Blue">
    <Button.Styles>
        <Style Selector="Button /template/ ContentPresenter#PART_ContentPresenter">
            <Style.Animations>
                <Animation IterationCount="Infinite" Duration="0:0:2">
                    <KeyFrame Cue="0%">
                        <Setter Property="Background" Value="Red" />
                    </KeyFrame>
                    <KeyFrame Cue="100%">
                        <Setter Property="Background" Value="Blue" />
                    </KeyFrame>
                </Animation>
            </Style.Animations>
        </Style>
    </Button.Styles>
</Button>
```

<Image light={SetterPrecedenceAnimationCorrectScreenshot} alt="Button animation working correctly after targeting the ContentPresenter directly" position="center" maxWidth={400} cornerRadius="true"/>

## See also

- [Styles](/docs/styling/styles)
- [Style precedence](/docs/styling/style-precedence)
- [Control themes](/docs/styling/control-themes)
