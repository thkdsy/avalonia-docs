---
id: usercontrol
title: UserControl
description: 用于创建具有预定义 XAML 布局的可复用组合控件的基类。
doc-type: reference
---

[`UserControl`](/api/avalonia/controls/usercontrol) 控件是一种 [ContentControl](/controls/data-display/contentcontrol)，表示在预定义布局中的一组可复用控件。

[`UserControl`](/api/avalonia/controls/usercontrol) 实际上只是在 `ContentControl` 之上提供了很少的附加功能。区别在于，你通常不会直接创建 `UserControl` 类的实例；相反，一般会为应用中要显示的每个“视图”创建一个新的 `UserControl` 子类。

## 常见属性

| 属性 | 说明 |
| :--- | :--- |
| `Content` | 控件中要显示的内容 |

## 基本示例

下面的示例定义了一个简单的 `UserControl`，它使用 `StackPanel` 布局，并包含一个 `TextBlock` 和一个 `Button`：

```xml title='MyCustomView.axaml'
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyApp.MyCustomView">
    <StackPanel>
        <TextBlock Text="Hello from a UserControl!" />
        <Button Content="Click me" />
    </StackPanel>
</UserControl>
```

然后，你可以通过引用其命名空间，在 `Window` 或任何其他容器中使用这个 `UserControl`：

```xml
<Window xmlns:local="clr-namespace:MyApp">
    <local:MyCustomView />
</Window>
```

## 何时使用 `UserControl`

`UserControl` 是在 MVVM 应用中创建视图的标准方式。应用中的每个视图通常都是一个 `UserControl` 子类，并配有对应的视图模型。如果你需要一个支持自定义模板和主题样式的控件，可以考虑创建 [`TemplatedControl`](/docs/custom-controls/templated-controls)。

## 另请参阅

- [ContentControl](/controls/data-display/contentcontrol)
- [Creating custom controls](/docs/custom-controls/defining-properties)
- [UserControl API reference](/api/avalonia/controls/usercontrol)
- [`UserControl.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/UserControl.cs)
