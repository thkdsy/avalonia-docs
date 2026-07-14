---
id: focus
title: 焦点
description: 了解如何在 Avalonia 中管理键盘焦点，包括 Tab 导航、方向导航（XYFocus）、焦点事件、伪类以及 FocusManager。
doc-type: concept
---

import DirectionalNavigationScreenshot from '/img/concepts/ui-concepts/user-input/directional-navigation.gif';

焦点指的是当前预期接收键盘输入的 [`InputElement`](/api/avalonia/input/inputelement)。获得焦点的控件通常会有某种视觉指示。最常见的例子是带有闪烁光标的 `TextBox`，但像 `Button` 和 [`Slider`](/api/avalonia/controls/slider) 这样的非文本控件同样也会参与焦点机制。

理解焦点的工作方式，有助于你构建具备良好无障碍性和键盘友好性的应用。本页将介绍核心焦点属性、事件、伪类，以及两种内置导航方案（Tab 导航和方向导航）。

## `IsFocused` and `Focusable`

`IsFocused` 是一个只读属性，用于跟踪某个 `InputElement` 当前是否持有焦点。

`Focusable` 属性用于启用或禁用 `InputElement` 获得焦点的能力。无法获得焦点的元素依然可以通过指针交互，因此在可能的情况下，你应为它们提供功能等价的键盘方式，例如热键。

```xml
<!-- 防止按钮接收键盘焦点 -->
<Button Content="仅可点击" Focusable="False" />
```

## 显式设置焦点

如果你想显式地将焦点赋给某个 `InputElement`，可以在代码中调用它的 `Focus()` 方法。你还可以选择指定 [`NavigationMethod`](/api/avalonia/input/navigationmethod) 和 `KeyModifiers`，以模拟某种特定导航流程触发的焦点变化。显式设置焦点常用于数据录入表单加载时自动聚焦到某个输入控件，或在当前输入完成后通过代码将焦点移动到下一个控件。

```csharp
// 视图加载时让控件获得焦点
myTextBox.Focus(NavigationMethod.Unspecified, KeyModifiers.None);
```

| `NavigationMethod` | 触发说明 |
|:--------------------|:---------------------------|
| `Tab`               | 按下 Tab 键 |
| `Pointer`           | 指针交互 |
| `Directional`       | 二维方向导航（[`XYFocus`](/api/avalonia/input/xyfocus)） |
| `Unspecified`       | 默认 |

## 焦点事件

`InputElement` 公开了 `GotFocus` 和 `LostFocus` 事件。`GotFocusEventArgs` 包含触发焦点变化的 `NavigationMethod` 和 `KeyModifiers`，你可以据此调整 UI 行为。

```csharp
myTextBox.GotFocus += (sender, e) =>
{
    if (e.NavigationMethod == NavigationMethod.Tab)
    {
        // 当用户通过 Tab 进入该输入框时，选中全部文本
        myTextBox.SelectAll();
    }
};
```

## 焦点伪类

当你为可获得焦点的控件编写样式时，这些伪类会非常有用。

| 伪类 | 说明 |
|:-----------------|:--------------------------------------------------------------|
| `:focus`         | 控件当前拥有焦点。 |
| `:focus-within`  | 控件自身拥有焦点，或其某个后代拥有焦点。 |
| `:focus-visible` | 控件拥有焦点，并且应显示可见的焦点指示。 |

:::tip
`FocusAdorner` 属性会为带有 `:focus-visible` 的控件显示默认的焦点视觉效果，通常是一个 `Border`。如果你已经通过 `:focus-visible` 自定义了焦点指示样式，请将 `FocusAdorner` 设为 `null`，以避免出现重复指示器。
:::

## `FocusManager`

`FocusManager` 提供了全局访问焦点功能的能力，例如获取当前拥有焦点的元素，或清除焦点。更多信息请参阅 [FocusManager 文档](/docs/services/focus-manager)。

```csharp
// 从 TopLevel 获取焦点管理器
var focusManager = TopLevel.GetTopLevel(myControl)?.FocusManager;

// 获取当前获得焦点的元素
var focused = focusManager?.GetFocusedElement();
```

## Tab 焦点导航

当你按下 Tab 键时，就会发生 Tab 焦点导航。凡是将 `IsTabStop` 属性设为 `true` 的 `InputElement` 都会参与 Tab 导航。`TabIndex` 属性用于指定优先级，数值越小越先被导航到。如果多个控件的 `TabIndex` 相同，则优先级由可视树遍历顺序决定。

`KeyboardNavigation.TabNavigation` 附加属性可以为作为容器的 `InputElement` 设置 `KeyboardNavigationMode`，从而修改 Tab 导航在其子元素之间移动的方式。

```xml
<!-- 在 StackPanel 内循环 Tab 焦点 -->
<StackPanel KeyboardNavigation.TabNavigation="Cycle">
    <TextBox TabIndex="0" Watermark="First field" />
    <TextBox TabIndex="1" Watermark="Second field" />
    <TextBox TabIndex="2" Watermark="Third field" />
</StackPanel>
```

| `KeyboardNavigationMode` | 容器内项目遍历方式 |
|:--------------------------|:----------------------------------------------------------|
| `Continue`                | 穿过当前项目后继续进入下一个容器 |
| `Cycle`                   | 在自身项目内循环并回绕 |
| `Contained`               | 在起始项或末尾项处停止 |
| `Once`                    | 容器及其子元素作为一个整体只接收一次焦点 |
| `None`                    | 项目不会通过 Tab 导航获得焦点 |
| `Local`                   | 仅在本地子树内考虑 `TabIndex` |

## 方向焦点导航

通过 `XYFocus` 进行的焦点导航是一种二维方向方案，它允许从当前聚焦控件出发，按左、右、上、下四个方向在空间中导航到其他控件。默认情况下，`XYFocus.NavigationModes` 允许 `Gamepad` 和 `Remote` 导航。

| `KeyDeviceType` | 设备 |
|:----------------|:------------------------------------------|
| `Disabled`      | 禁用所有按键设备的 XY 导航。 |
| `Keyboard`      | 可使用键盘方向键。 |
| `Gamepad`       | 可使用游戏手柄方向键。 |
| `Remote`        | 可使用遥控器。 |
| `Enabled`       | 可使用所有设备。 |

像 Android 这类能够原生发送手柄输入的设备，支持 Gamepad 输入。不过，Avalonia 目前仍缺少跨平台的 Gamepad API，因此暂时无法提供广泛的开箱即用支持。

### 导航策略

启用二维方向导航后，系统会使用一套消歧策略来选择导航目标。

| `XYFocusNavigationStrategy`    | 导航目标选择方式 |
|:-------------------------------|:------------------------------------------------------------------------------|
| `Auto`                         | 从祖先继承策略；如果没有祖先指定，则使用 `Projection`。 |
| `Projection`                   | 在导航方向上投射一条线，选中最先遇到的元素。 |
| `NavigationDirectionDistance`  | 选择距离导航线轴最近的元素。 |
| `RectilinearDistance`          | 根据最短曼哈顿距离选择最近元素。 |

### 显式导航

`XYFocus` 允许每个控件通过 `XYFocus.Up`、`XYFocus.Down`、`XYFocus.Left` 和 `XYFocus.Right` 显式指定某个方向键按下时的导航目标。它的优先级高于任何导航策略。

:::caution
焦点接管（focus engagement）目前尚未实现，因此当方向焦点导航与那些自身也会处理方向输入的控件一起使用时，可能会存在一些限制，尤其是在视觉表现上。
:::

### 示例

下面的示例演示了如何在 `WrapPanel` 中使用方向焦点导航。它显式允许从第一个元素导航到最后一个元素，反之亦然。

`Slider` 则展示了如何将导航与控件交互结合使用。在桌面端，当 `Slider` 获得焦点时按下 Enter 键，会进入一个交互状态，此时方向键将用于修改 `Slider.Value`，而不是触发导航。再次按下 Enter 则会结束该交互，并恢复方向焦点导航。

```xml title="DirectionalNavigation.axaml"
<Window
    XYFocus.NavigationModes="Enabled"
    XYFocus.UpNavigationStrategy="Projection"
    XYFocus.DownNavigationStrategy="Projection"
    XYFocus.LeftNavigationStrategy="Projection"
    XYFocus.RightNavigationStrategy="Projection">
    <Grid>
        <WrapPanel>
            <Button x:Name="first"
                Content="First"
                XYFocus.Left="{Binding #last}" />
            <Button Content="Second" />
            <Button Content="Third" />

            <Slider Width="100" Maximum="100" />

            <Button Content="Fourth" />
            <Button x:Name="last"
                Content="Last"
                XYFocus.Right="{Binding #first}" />
        </WrapPanel>
    </Grid>
</Window>
```

<Image light={DirectionalNavigationScreenshot} alt="方向导航示例" position="center" maxWidth={400} cornerRadius="true"/>

## 另请参阅

- [键盘与热键](/docs/input-interaction/keyboard-and-hotkeys)：按键绑定与键盘快捷键。
- [FocusManager](/docs/services/focus-manager)：全局焦点管理服务。
- [指针事件](/docs/input-interaction/pointer)：指针设备事件。
- [路由事件](/docs/input-interaction/routed-events)：事件如何在元素树中传播。
