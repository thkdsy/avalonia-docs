---
id: style-selectors
title: 样式选择器
description: 了解 Avalonia 样式选择器如何使用类似 CSS 的语法，通过类型、类、名称和状态匹配控件。
doc-type: concept
---

Avalonia 使用样式选择器，通过一种类似 CSS（层叠样式表）的自定义 XAML 语法来匹配控件。选择器决定某个样式会应用到哪些控件上。

## 选择器速查

| 选择器 | 说明 |
|---|---|
| [`Button`](/api/avalonia/controls/button) | 选择所有 `Button` 控件。 |
| `Button.red` | 选择所有带有 `red` 样式类的 `Button` 控件。 |
| `Button.red.large` | 选择同时带有 `red` 和 `large` 样式类的 `Button` 控件。 |
| `Button:focus` | 选择所有激活了 `:focus` 伪类的 `Button` 控件。 |
| `Button.red:focus` | 选择同时带有 `red` 样式类并激活了 `:focus` 伪类的 `Button` 控件。 |
| `Button#myButton` | 选择 `Name="myButton"` 的 `Button`。 |
| `StackPanel Button.xl` | 选择作为 `StackPanel` 后代（任意层级）的 `Button.xl` 控件。 |
| `StackPanel > Button.xl` | 选择作为 `StackPanel` 直接子元素的 `Button.xl` 控件。 |
| `Button /template/ ContentPresenter` | 选择 `Button` 控件模板内部的指定控件（本例中为 [`ContentPresenter`](/api/avalonia/controls/presenters/contentpresenter)）。 |
| `:is(Button)` | 选择 `Button` 或其派生类型的控件。 |
| `:not(Button.red)` | 选择不匹配 `Button.red` 的控件。 |
| `Button:nth-child(2n+1)` | 在同级元素中选择奇数位置的 `Button` 控件。 |

## 选择器如何工作

样式选择器通过 `Style` 的 `Selector` 属性指定：

```xml
<Style Selector="Button.primary:pointerover">
    <Setter Property="Background" Value="DarkBlue" />
</Style>
```

这个选择器会匹配所有带有 `primary` 样式类、且当前被指针悬停的 `Button` 控件。它由多个部分从左到右组成：

1. `Button` 匹配控件类型
2. `.primary` 匹配样式类
3. `:pointerover` 匹配伪类（状态）

## 选择器特异性

当多个样式同时匹配同一个控件时，更具体的选择器会胜出。特异性按以下顺序决定：

1. 名称选择器（`#name`）最具体
2. 属性选择器和伪类选择器（如 `:pointerover`、`[IsEnabled=True]`）
3. 样式类选择器（`.primary`）
4. 类型选择器（`Button`）
5. 后代/子元素组合符根据树中的位置解析

如果两个选择器具有相同特异性，则后声明的那个会生效。

## 完整参考

有关所有选择器格式、运算符和组合符的完整说明，请参阅 [style selector syntax](/docs/styling/style-selector-syntax) 参考页。

## 另请参阅

- [Style selector syntax](/docs/styling/style-selector-syntax)
- [Styles](/docs/styling/styles)
- [Style classes](/docs/styling/style-classes)
- [Pseudoclasses](/docs/styling/pseudoclasses)
