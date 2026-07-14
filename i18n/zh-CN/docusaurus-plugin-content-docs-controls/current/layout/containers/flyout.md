---
id: flyout
title: Flyout
---

import FlyoutShowAttachedScreenshot from '/img/controls/flyout/flyout-show-attached.gif';

Flyout 是一种可关闭的容器，可以附加到某些“宿主”控件上；尽管 flyout 本身并不是控件。它们会在宿主控件获得焦点时显示，并通过多种方式再次隐藏。

Flyout 可以包含简单内容，也可以包含更丰富、组合式的 UI 内容。

在 _Avalonia_ 应用中，Flyout 可以声明为资源，并在两个或更多宿主控件之间共享。

## 示例

Flyout 通过宿主控件的 [`Flyout`](/api/avalonia/controls/flyout) 属性附加到宿主控件上。例如：

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             Padding="20">
  <Button Content="Button with flyout">
    <Button.Flyout >
      <Flyout>This is the button flyout.</Flyout>
    </Button.Flyout>
  </Button>
</UserControl>
```

</XamlPreview>

:::caution
只有 button 和 split button 控件支持 `Flyout` 属性。对于其他 _Avalonia_ 内置控件，你可以改用 `AttachedFlyout` 属性来附加 flyout。
:::

对于没有 `Flyout` 属性的控件，请使用 `AttachedFlyout` 属性。
这种 flyout 不会自动显示，需要在 code-behind 中以编程方式触发。

```xml
<Border Background="Red" PointerPressed="Border_PointerPressed">
    <FlyoutBase.AttachedFlyout>
        <Flyout>
            <TextBlock Text="Red Rectangle Flyout." />
        </Flyout>
    </FlyoutBase.AttachedFlyout>
</Border>
```

```csharp
public void Border_PointerPressed(object sender, PointerPressedEventArgs args)
{
    var ctl = sender as Control;
    if (ctl != null)
    {
        FlyoutBase.ShowAttachedFlyout(ctl);
    }
}
```

<Image light={FlyoutShowAttachedScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 说明 |
| ----------------- | ------------------------------------------------------------------------------------ |
| `Content` | 显示在 flyout 内部的内容。 |
| `ContentTemplate` | 应用于 `Content` 的 `DataTemplate`。当 `Content` 绑定到视图模型对象时会很有用。 |
| `Placement` | flyout 相对于其附加控件打开的位置。 |
| `ShowMode` | 描述 flyout 的显示和隐藏方式。见下方选项。 |

## 显示模式

该设置描述了 flyout 的显示和隐藏方式：

<table><thead><tr><th width="259">模式</th><th>说明</th></tr></thead><tbody><tr><td><code>Standard</code></td><td>当附加到的控件获得焦点时，flyout 会显示；当该控件失去焦点时（例如用户按 Tab 切走，或点击别处），flyout 会隐藏。</td></tr><tr><td><code>Transient</code></td><td></td></tr><tr><td><code>TransientWithDismiss OnPointerMoveAway</code></td><td></td></tr></tbody></table>

## 所有 flyout 的常用方法

| 方法 | 说明 |
| ----------------------- | --------------------------------------------------------------------------------------- |
| `ShowAt(Control)` | 在指定目标处显示 Flyout。 |
| `ShowAt(Control, bool)` | 在指定目标处显示 Flyout，但将其放置到当前指针位置。 |
| `Hide` | 隐藏 Flyout。 |

## 共享 Flyout

你可以在应用中的两个或更多元素之间共享 flyout。例如，下面展示了如何共享来自窗口资源集合中的 flyout：

```xml
<Window.Resources>
    <Flyout x:Key="MySharedFlyout">
        <!-- Flyout content here -->
    </Flyout>
</Window.Resources>

<Button Content="Click me!" Flyout="{StaticResource MySharedFlyout}" />

<Button Content="Now click me!" Flyout="{StaticResource MySharedFlyout}" />
```

## 设置 flyout 样式

尽管 flyout 本身不是控件，但你仍然可以通过为 `Flyout` 用来显示内容的 presenter 设置样式，自定义其整体外观。对于普通 `Flyout`，对应的是 `FlyoutPresenter`；对于 `MenuFlyout`，对应的是 `MenuFlyoutPresenter`。由于这些 presenter 并不会被直接暴露出来，因此你可以通过 `FlyoutBase` 上的 `FlyoutPresenterClasses` 属性传递特定样式类，以便作用于某些特定的 flyout。

```xml
<Style Selector="FlyoutPresenter.mySpecialClass">
    <Setter Property="Background" Value="Red" />
</Style>

<Flyout FlyoutPresenterClasses="mySpecialClass">
    <!-- Flyout content here -->
</Flyout>
```

## 另请参阅

- [Flyout API reference](/api/avalonia/controls/flyout)
- [`Flyout.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Flyouts/Flyout.cs)
