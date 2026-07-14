---
id: virtualkeyboardscope
title: VirtualKeyboardScope
description: 一个容器控件，根据输入焦点自动显示和隐藏虚拟键盘。
doc-type: reference
tags:
  - avalonia pro
  - avalonia enterprise
---

`VirtualKeyboardScope` 控件是一个容器，它根据输入焦点自动管理[虚拟键盘](/controls/input/text-input/virtualkeyboard)的可见性。当作用域内的文本输入控件获得焦点时，键盘出现。当焦点移到非文本控件或完全丢失焦点时，键盘隐藏。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 概述

`VirtualKeyboardScope` 是将虚拟键盘集成到应用程序中的推荐方式。它提供了无缝的体验，键盘仅在需要时出现。它还处理输入焦点和定位，以便键盘不会遮挡活动输入控件。

## 属性

| 属性 | 类型 | 描述 |
|----------|------|-------------|
| `InputMethods` | `IEnumerable\<VirtualKeyboardInputMethod>` | 获取或设置可供用户使用的输入方法集合。 |

## 使用示例

### 最小实现

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <VirtualKeyboardScope InputMethods="en-US:kbd:standard">
        <StackPanel>
            <TextBlock>输入您的姓名：</TextBlock>
            <TextBox />
            <TextBlock>评论：</TextBlock>
            <TextBox AcceptsReturn="True" Height="100" />
        </StackPanel>
    </VirtualKeyboardScope>
</Window>
```

### 多种输入方法

```xml
<VirtualKeyboardScope InputMethods="en-US:kbd:standard, de:kbd:standard, ja:ime:kana">
    <StackPanel>
        <TextBox PlaceholderText="在此输入" />
        <TextBlock>键盘将支持英语、德语和日语输入</TextBlock>
    </StackPanel>
</VirtualKeyboardScope>
```

### 动态绑定 `InputMethods`

```xml
<VirtualKeyboardScope InputMethods="{Binding SelectedInputMethods}">
    <StackPanel>
        <TextBox />
        <ListBox ItemsSource="{Binding AvailableInputMethods}"
                 SelectionMode="Multiple"
                 Selection="{Binding SelectedInputMethodsCollection}" />
    </StackPanel>
</VirtualKeyboardScope>
```

### 代码后置配置

```csharp
// 使用 SelectMany + ToList 获取特定语言的输入方法
var inputMethods = new[] { "en-US", "ja", "de" }
    .SelectMany(VirtualKeyboardInputMethod.GetInputMethodsForLanguage)
    .ToList();

// 分配给 VirtualKeyboardScope
myKeyboardScope.InputMethods = inputMethods;
```

## 使用 `TextInputOptions`

您可以使用 `TextInputOptions` 附加属性自定义输入字段与虚拟键盘的交互方式：

```xml
<VirtualKeyboardScope InputMethods="en-US:kbd:standard">
    <StackPanel>
        <TextBox TextInputOptions.ContentType="Email"
                 PlaceholderText="输入电子邮件地址" />

        <TextBox TextInputOptions.ContentType="Digits"
                 TextInputOptions.ReturnKeyType="Done"
                 PlaceholderText="输入 PIN 码" />

        <TextBox TextInputOptions.ReturnKeyType="Search"
                 PlaceholderText="搜索..." />
    </StackPanel>
</VirtualKeyboardScope>
```

## 多个键盘

您可以在应用程序中放置多个 `VirtualKeyboardScope` 控件。一次只显示一个键盘，对应于当前持有焦点的作用域。

```xml
<Grid ColumnDefinitions="*, *">
    <!-- 第一个作用域：英语和德语 -->
    <VirtualKeyboardScope Grid.Column="0" InputMethods="en-US:kbd:standard, de:kbd:standard">
        <StackPanel>
            <TextBlock>表单 1</TextBlock>
            <TextBox />
        </StackPanel>
    </VirtualKeyboardScope>

    <!-- 第二个作用域：英语和日语 -->
    <VirtualKeyboardScope Grid.Column="1" InputMethods="en-US:kbd:standard, ja:ime:kana">
        <StackPanel>
            <TextBlock>表单 2</TextBlock>
            <TextBox />
        </StackPanel>
    </VirtualKeyboardScope>
</Grid>
```

## 最佳实践

- **将作用域放在正确层级。** 将 `VirtualKeyboardScope` 放置在 `Window` 或 `UserControl` 的根级别，或者放置在一个包裹所有应触发键盘的文本输入控件的层级。
- **选择相关的输入方法。** 提供与目标受众匹配的输入方法。考虑为应用程序支持的每个区域提供至少一种语言布局。
- **考虑减少的屏幕空间。** `VirtualKeyboardScope` 会在键盘出现时自动滚动以保持焦点输入可见。但是，您可能仍需要调整布局，以便在键盘显示时内容仍然可用。
- **利用只读和禁用状态。** 键盘不会为只读或禁用的文本输入控件出现。对应该可选中但不可编辑的文本使用 `IsReadOnly="True"`。

## 实用说明

- `VirtualKeyboardScope` 通常用于自助服务终端和嵌入式 Linux 场景，这些场景中没有物理键盘。在带有硬件键盘的桌面平台上，用户不会看到屏幕键盘。
- 如果您需要完全控制键盘在布局中的位置，请改用独立的 [`VirtualKeyboard` 控件](/controls/input/text-input/virtualkeyboard/virtualkeyboard-control)。
- `InputMethods` 属性接受逗号分隔的字符串（例如 `"en-US:kbd:standard, de:kbd:standard"`）或绑定的 `VirtualKeyboardInputMethod` 对象集合。使用字符串形式进行快速原型设计，当您的可用语言在运行时确定时使用绑定形式。

## 另请参阅

- [VirtualKeyboard](/controls/input/text-input/virtualkeyboard/virtualkeyboard-control) 用于手动键盘放置
- [TextBox](/controls/input/text-input/textbox) 用于标准文本输入控件。
