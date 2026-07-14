---
id: style-precedence
title: 样式优先级
---

当多个样式、本地值和动画同时作用于控件的同一个属性时，Avalonia 会使用固定的优先级顺序来决定最终采用哪个值。理解这个顺序，对于排查“样式看起来没有生效”的问题非常重要。

## 值来源（从高到低）

| 优先级 | 来源 | 示例 |
|---|---|---|
| 1 | **Animation** | 一个作用于 `Opacity` 的活动 `<Animation>` 关键帧 |
| 2 | **Local value** | `<Button Foreground="Red" />` 或代码中的 `SetValue` |
| 3 | **Style trigger** | 匹配 `:pointerover` 或 `:pressed` 等伪类的样式 |
| 4 | **Template** | 在 `ControlTemplate` 中设置的值 |
| 5 | **Style** | 匹配控件类型或类的样式 |
| 6 | **Inherited** | 从视觉树祖先继承来的值 |
| 7 | **Default** | 属性注册时定义的默认值 |

属性系统会按顺序检查这些层级，并返回找到的第一个值。完整 API 细节请参阅 [Property value precedence](/docs/properties/value-precedence)。

## 为什么样式可能没有生效

最常见的样式问题是：**本地值会挡住样式值**。由于本地值（优先级 2）高于样式触发器（优先级 3）和普通样式（优先级 5），因此直接在 XAML 控件上设置的属性值总是会获胜：

```xml
<Window.Styles>
    <Style Selector="Button">
        <Setter Property="Foreground" Value="Blue" />
    </Style>
    <Style Selector="Button:pointerover">
        <Setter Property="Foreground" Value="Green" />
    </Style>
</Window.Styles>

<!-- 这个按钮始终是红色，即使悬停时也是，因为 LocalValue 的优先级高于 Style 和 StyleTrigger -->
<Button Foreground="Red" Content="Always Red" />

<!-- 这个按钮默认是蓝色，悬停时变成绿色 -->
<Button Content="Styled correctly" />
```

**解决方法：** 删除本地值，让样式系统接管该属性；或者将这个值移动到样式中。

## 样式声明顺序

当两个处于**同一优先级层级**的样式同时作用于同一个属性时，后声明的样式会生效：

```xml
<Window.Styles>
    <Style Selector="Button">
        <Setter Property="Background" Value="Blue" />
    </Style>

    <!-- 由于声明得更晚，因此这个值会生效 -->
    <Style Selector="Button">
        <Setter Property="Background" Value="Red" />
    </Style>
</Window.Styles>
```

来自不同来源的样式会按照它们在逻辑树中的位置进行评估，顺序是从控件向上直到应用。声明在 `UserControl` 上的样式之所以会覆盖 `App.axaml` 中匹配的样式，是因为更近的作用域会在更后面被评估。

## 伪类触发器与基础样式

伪类选择器（如 `:pointerover`、`:pressed`、`:focus`、`:disabled`）工作在 `StyleTrigger` 优先级层级上，高于普通 `Style` 层级。这意味着对于同一个属性，伪类样式会覆盖基础类型样式：

```xml
<!-- 优先级：Style -->
<Style Selector="Button">
    <Setter Property="Background" Value="#6366F1" />
</Style>

<!-- 优先级：StyleTrigger（高于 Style） -->
<Style Selector="Button:pointerover">
    <Setter Property="Background" Value="#818CF8" />
</Style>
```

当指针进入按钮时，`StyleTrigger` 的值会生效；当指针离开时，又会恢复为 `Style` 的值。

## 动画会覆盖一切

动画处于最高优先级。只要动画正在运行，它的值就会覆盖其他所有来源，包括本地值：

```xml
<Style Selector="Button.pulse">
    <Style.Animations>
        <Animation Duration="0:0:1" IterationCount="Infinite">
            <KeyFrame Cue="50%">
                <Setter Property="Opacity" Value="0.5" />
            </KeyFrame>
            <KeyFrame Cue="100%">
                <Setter Property="Opacity" Value="1.0" />
            </KeyFrame>
        </Animation>
    </Style.Animations>
</Style>
```

即使是 `<Button Opacity="0.8" Classes="pulse" />`，在动画运行期间也会在 0.5 和 1.0 之间脉动变化。

## 继承值

有些属性（如 `FontSize`、`Foreground`、`FlowDirection`）会从祖先控件继承值。继承值的优先级倒数第二，因此任意样式、模板或本地值都可以覆盖它：

```xml
<!-- 通过继承为所有后代设置 FontSize -->
<StackPanel TextElement.FontSize="18">
    <!-- 继承 18 -->
    <TextBlock Text="Large text" />

    <!-- 样式覆盖继承值 -->
    <TextBlock Text="Small text" Classes="caption" />
</StackPanel>

<Style Selector="TextBlock.caption">
    <Setter Property="FontSize" Value="12" />
</Style>
```

## 调试优先级问题

可以使用 DevTools（运行时按 <kbd>F12</kbd>）检查属性值。**Properties** 面板会显示当前值及其来源，帮助你判断到底是哪一层提供了最终生效的值。

## 另请参阅

- [Property value precedence](/docs/properties/value-precedence)
- [Styles](/docs/styling/styles)
- [Pseudoclasses](/docs/styling/pseudoclasses)
- [Styling best practices](/docs/styling/style-best-practices)
