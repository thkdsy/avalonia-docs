---
id: value-precedence
title: 属性值优先级
---

当多个值源同时为同一个 Avalonia 属性提供值时（例如本地值、样式 setter、动画值、继承值），Avalonia 必须决定最终采用哪个值。属性系统通过 `BindingPriority` 枚举定义的一套固定优先级顺序来完成这一判断。

## 优先级顺序

属性值会按照从高到低的优先级顺序进行解析：

| 优先级 | `BindingPriority` 值 | 说明 |
|---|---|---|
| **1（最高）** | `Animation` | 活动动画施加的值。 |
| **2** | `LocalValue` | 通过 `SetValue`、XAML 属性或代码直接设置在对象上的值。 |
| **3** | `StyleTrigger` | 由样式选择器在匹配临时条件时应用的值，例如伪类（`:pointerover`、`:pressed`、`:focus`）。 |
| **4** | `Template` | 在控件模板内部设置的值。 |
| **5** | `Style` | 由匹配控件类型或类名的样式选择器应用的值。 |
| **6** | `Inherited` | 从可视树中的祖先元素继承而来的值。 |
| **7（最低）** | `Unset` | 尚未设置任何值，此时使用属性注册时的默认值。 |

## 优先级如何生效

当你请求某个属性值时（通过 `GetValue` 或绑定），属性系统会按顺序检查每个优先级层级，并返回找到的第一个值。

### Example

假设一个 `Button` 拥有 `Foreground` 属性：

```xml
<!-- 应用级样式（优先级：Style） -->
<Application.Styles>
    <Style Selector="Button">
        <Setter Property="Foreground" Value="Black" />
    </Style>
    <Style Selector="Button:pointerover">
        <Setter Property="Foreground" Value="Blue" />
    </Style>
</Application.Styles>
```

```xml
<!-- 本地值（优先级：LocalValue） -->
<Button Foreground="Red" Content="Click me" />
```

在这个场景中：
- 按钮的 `Foreground` 是 **Red**，因为 `LocalValue` 的优先级高于 `Style` 和 `StyleTrigger`。
- 即使指针悬停在按钮上，`Foreground` 仍然保持 **Red**，因为 `LocalValue`（优先级 2）高于 `StyleTrigger`（优先级 3）。

如果移除本地 `Foreground="Red"` 属性：
- 按钮默认会显示为 **Black**（来自 `Style` 的 setter）。
- 当指针悬停在按钮上时，它会变为 **Blue**（来自 `:pointerover` 的 `StyleTrigger`）。

## 动画会覆盖一切

动画拥有最高优先级。只要动画处于活动状态，它的值就会覆盖本地值、样式值以及其他所有来源的值。这样可以确保平滑的视觉过渡不会被样式变化打断。

```xml
<Style Selector="Button:pointerover">
    <Style.Animations>
        <Animation Duration="0:0:0.2">
            <KeyFrame Cue="100%">
                <Setter Property="Opacity" Value="0.8" />
            </KeyFrame>
        </Animation>
    </Style.Animations>
</Style>
```

## 清除值

调用 `ClearValue` 时，你移除的是 `LocalValue` 优先级上的值。随后属性系统会回退到下一个可用的值源：

```csharp
// 设置一个本地值（覆盖样式）
myButton.SetValue(Button.ForegroundProperty, Brushes.Red);

// 清除本地值（样式重新生效）
myButton.ClearValue(Button.ForegroundProperty);
```

## 在指定优先级上设置值

在高级场景中，你可以使用 `SetValue` 的重载，在指定优先级上设置一个值：

```csharp
myButton.SetValue(Button.ForegroundProperty, Brushes.Red, BindingPriority.Style);
```

这主要由样式系统在内部使用。在大多数应用代码中，你通常设置的是本地值（也就是调用 `SetValue` 时的默认行为）。

## `SetCurrentValue`

`SetCurrentValue` 方法会在当前最高优先级层级上设置一个值，而不是直接写入 `LocalValue`。这在控件实现中很有用，因为你可以在不覆盖样式的前提下更新属性：

```csharp
// 设置值，但不创建 LocalValue 条目
myButton.SetCurrentValue(Button.ForegroundProperty, Brushes.Green);
```

## 对数据绑定的影响

绑定会在其来源对应的优先级层级上生效。通过样式创建的绑定会作用于 `Style` 或 `StyleTrigger` 层级；而直接在 XAML 中设置的绑定则作用于 `LocalValue` 层级：

```xml
<!-- 这个绑定作用于 LocalValue 优先级 -->
<Button Foreground="{Binding ButtonColor}" />
```

由于 `LocalValue` 绑定的优先级高于样式值，因此直接在 XAML 中绑定的属性值会覆盖任何样式设置的值，甚至包括由伪类触发的样式。

:::tip
如果你希望样式能够覆盖某个属性，请避免直接在 XAML 中设置它。更好的方式是在合适的层级使用样式来设置。
:::

## 另请参阅

- [属性系统概览](/docs/properties)：Avalonia 属性类型概览。
- [样式](/docs/styling/styles)：如何定义和应用样式。
- [动画](/docs/graphics-animation/animations)：动画如何与属性交互。
