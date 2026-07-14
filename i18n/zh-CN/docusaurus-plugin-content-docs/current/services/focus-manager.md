---
id: focus-manager
title: 焦点管理器
description: 使用 FocusManager 服务在 Avalonia 应用中管理键盘焦点，跟踪、设置和清除当前拥有焦点的元素。
doc-type: reference
---

[`FocusManager`](/api/avalonia/input/focusmanager) 服务负责管理应用中的键盘焦点。它会跟踪当前获得焦点的元素以及当前焦点作用域。

你可以通过 [`TopLevel`](/api/avalonia/controls/toplevel) 或 `Window` 的实例来访问 `FocusManager`。关于如何访问 `TopLevel`，请参阅 [TopLevel](/docs/fundamentals/top-level) 页面。

```csharp
var focusManager = window.FocusManager;
```

## 方法

### `GetFocusedElement()`

返回当前获得焦点的 `IInputElement`；如果没有任何元素获得焦点，则返回 `null`。

```csharp
IInputElement? GetFocusedElement()
```

你可以使用这个方法来检查当前是哪一个控件持有键盘焦点：

```csharp
var focused = focusManager.GetFocusedElement();
if (focused is TextBox textBox)
{
    // 用户当前正在编辑一个文本框
}
```

### `ClearFocus()`

移除当前获得焦点元素上的键盘焦点。调用该方法后，在其他元素再次获得焦点之前，`GetFocusedElement()` 都会返回 `null`。

```csharp
void ClearFocus()
```

## 使用提示

### 使控件获得焦点

通常你不需要通过 `FocusManager` 服务来让某个控件获得焦点。更常见的做法是直接在控件上调用 `Focus` 方法：

```csharp
bool hasFocused = button.Focus();
```

如果控件当前不可见，或者它的 `Focusable` 属性设为 `false`，那么 `Focus` 方法会返回 `false`。

### 监听全局焦点变更

`FocusManager.GetFocusedElement` 方法只会返回某一时刻当前获得焦点的控件，因此并不适合用于实时响应焦点变更。如果你想在所有 top level 范围内全局监听焦点变化，应订阅对应的路由事件：

```csharp
InputElement.GotFocusEvent.Raised.Subscribe(args =>
{
    var (sender, e) = args;
    // 处理焦点变化
});
```

### Tab 导航顺序

默认情况下，控件会按照它们在可视树中出现的顺序参与导航。若要修改 Tab 顺序，请为控件设置 `TabIndex` 属性：

```xml
<StackPanel>
    <TextBox TabIndex="2" PlaceholderText="Second" />
    <TextBox TabIndex="1" PlaceholderText="First" />
    <TextBox TabIndex="3" PlaceholderText="Third" />
</StackPanel>
```

### 阻止控件获得焦点

将 `Focusable` 设为 `False`，即可让控件不参与键盘导航：

```xml
<Button Content="不可获得焦点" Focusable="False" />
```

### 加载时设置焦点

如果你希望在视图加载时让某个特定控件获得焦点，可以重写 `OnLoaded` 并在目标控件上调用 `Focus`：

```csharp
protected override void OnLoaded(RoutedEventArgs e)
{
    base.OnLoaded(e);
    myTextBox.Focus();
}
```

## 另请参阅

- [焦点](/docs/input-interaction/focus)：焦点系统概览与焦点事件。
- [TopLevel](/docs/fundamentals/top-level)：从 `TopLevel` 访问平台服务。
