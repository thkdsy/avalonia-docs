---
id: textbox-how-to
title: "如何：使用 TextBox"
description: 了解 Avalonia 中 TextBox 的验证、格式化、输入限制、选区以及自定义方式。
doc-type: how-to
---

本指南介绍常见的 TextBox 使用场景：验证、格式化、输入限制、文本选择以及自定义外观。

## 基础文本绑定

使用 `TwoWay` 模式绑定 `Text` 属性（这也是 `TextBox.Text` 的默认模式）：

```xml
<TextBox Text="{Binding Username}" PlaceholderText="Enter username" />
```

```csharp
[ObservableProperty]
private string _username = "";
```

## 占位文本

在 TextBox 为空时显示提示文本：

```xml
<TextBox PlaceholderText="Search..." />
<TextBox PlaceholderText="Enter email address" />
```

当用户开始输入时，占位文本会消失；当文本被清空后，它又会重新显示。

如果你想自定义占位文本颜色，可以设置 `PlaceholderForeground`：

```xml
<TextBox PlaceholderText="Search..." PlaceholderForeground="Gray" />
```

默认情况下，占位文本会以 50% 透明度渲染。你可以通过覆盖主题资源 `TextControlPlaceholderOpacity` 来全局修改这个值，例如为了提升无障碍场景下的对比度：

```xml
<Application.Resources>
    <x:Double x:Key="TextControlPlaceholderOpacity">0.7</x:Double>
</Application.Resources>
```

## 密码输入

使用 `PasswordChar` 隐藏已输入字符：

```xml
<TextBox PasswordChar="*" PlaceholderText="Password" />
<TextBox PasswordChar="●" PlaceholderText="Password" />
```

如果你想实现“显示/隐藏密码”切换，可以绑定 `RevealPassword`：

```xml
<Grid ColumnDefinitions="*,Auto">
    <TextBox x:Name="PasswordBox" PasswordChar="●" Text="{Binding Password}" />
    <ToggleButton Grid.Column="1" Content="Show"
                  IsChecked="{Binding #PasswordBox.RevealPassword}" />
</Grid>
```

## 多行输入

启用多行文本输入：

```xml
<TextBox AcceptsReturn="True"
         TextWrapping="Wrap"
         Height="120"
         PlaceholderText="Enter your message..." />
```

| 属性 | 作用 |
|---|---|
| `AcceptsReturn="True"` | 允许按 Enter 创建新行 |
| `TextWrapping="Wrap"` | 长行自动换行，而不是横向滚动 |
| `AcceptsTab="True"` | 允许按 Tab 插入制表符 |

## 只读与禁用

```xml
<!-- 只读：可以选择和复制，但不能编辑 -->
<TextBox Text="{Binding DisplayValue}" IsReadOnly="True" />

<!-- 禁用：完全不可交互 -->
<TextBox Text="{Binding DisabledValue}" IsEnabled="False" />
```

## 文本选择

### 聚焦时全选

让 TextBox 在获得焦点时自动全选文本：

```csharp
private void OnTextBoxGotFocus(object? sender, GotFocusEventArgs e)
{
    if (sender is TextBox textBox)
    {
        textBox.SelectAll();
    }
}
```

```xml
<TextBox GotFocus="OnTextBoxGotFocus" Text="{Binding Value}" />
```

### 通过代码控制选择范围

```csharp
// 选择一个范围
myTextBox.SelectionStart = 5;
myTextBox.SelectionEnd = 10;

// 全选
myTextBox.SelectAll();

// 获取选中的文本
string selected = myTextBox.SelectedText;
```

## 输入验证

### 使用数据注解

使用 `INotifyDataErrorInfo`，可以直接在 TextBox 上显示验证错误：

```csharp
public partial class FormViewModel : ObservableValidator
{
    [ObservableProperty]
    [NotifyDataErrorInfo]
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid email format")]
    private string _email = "";
}
```

```xml
<TextBox Text="{Binding Email}" PlaceholderText="Email" />
```

当验证失败时，TextBox 会显示红色边框以及错误信息。详情请参阅 [Validation in Data Binding](/docs/data-binding/binding-validation)。

### 限制输入字符

你可以处理 `TextChanging` 事件来过滤输入：

```csharp
private void OnTextChanging(object? sender, TextChangingEventArgs e)
{
    // 只允许数字
    if (sender is TextBox textBox)
    {
        var newText = textBox.Text;
        if (newText is not null && !newText.All(char.IsDigit))
        {
            e.Cancel = true;
        }
    }
}
```

## 最大长度

限制可输入的字符数量：

```xml
<TextBox MaxLength="50" PlaceholderText="Max 50 characters" />
```

## 文本变化事件

你可以在文本变化时执行联想搜索或实时预览等逻辑：

```xml
<TextBox Text="{Binding SearchText}" />
```

```csharp
[ObservableProperty]
private string _searchText = "";

partial void OnSearchTextChanged(string value)
{
    ApplyFilter(value);
}
```

如果你希望使用去抖搜索（避免每次按键都触发筛选），请参阅 [Performance](/docs/app-development/performance#debounce-rapid-input)。

## 内嵌内容（左侧/右侧）

可以通过 `InnerLeftContent` 和 `InnerRightContent` 在 TextBox 内嵌入图标或按钮：

```xml
<TextBox PlaceholderText="Search..." InnerLeftContent="🔍">
    <TextBox.InnerRightContent>
        <Button Content="✕" Command="{Binding ClearSearchCommand}"
                Background="Transparent" BorderThickness="0"
                Padding="4" />
    </TextBox.InnerRightContent>
</TextBox>
```

## 撤销与重做

TextBox 原生支持标准键盘快捷键的撤销/重做（Ctrl+Z / Ctrl+Shift+Z），无需额外编写代码。

若要清除撤销历史：

```csharp
myTextBox.Clear(); // Clears text and undo history
```

## 样式设置

### 自定义外观

```xml
<Style Selector="TextBox.custom">
    <Setter Property="Background" Value="#F8F8F8" />
    <Setter Property="BorderBrush" Value="#E0E0E0" />
    <Setter Property="BorderThickness" Value="1" />
    <Setter Property="CornerRadius" Value="8" />
    <Setter Property="Padding" Value="12,8" />
</Style>

<Style Selector="TextBox.custom:focus">
    <Setter Property="BorderBrush" Value="#6366F1" />
    <Setter Property="BorderThickness" Value="2" />
</Style>

<Style Selector="TextBox.custom:error">
    <Setter Property="BorderBrush" Value="#EF4444" />
</Style>
```

### Removing the focus border

```xml
<Style Selector="TextBox.borderless">
    <Setter Property="BorderThickness" Value="0" />
    <Setter Property="Background" Value="Transparent" />
</Style>

<Style Selector="TextBox.borderless:focus">
    <Setter Property="BorderThickness" Value="0" />
</Style>
```

## Context Menu

TextBox has a built-in context menu with Cut, Copy, and Paste. To customize it:

```xml
<TextBox Text="{Binding Value}">
    <TextBox.ContextMenu>
        <ContextMenu>
            <MenuItem Header="Cut" Command="{Binding $parent[TextBox].Cut}" InputGesture="Ctrl+X" />
            <MenuItem Header="Copy" Command="{Binding $parent[TextBox].Copy}" InputGesture="Ctrl+C" />
            <MenuItem Header="Paste" Command="{Binding $parent[TextBox].Paste}" InputGesture="Ctrl+V" />
            <Separator />
            <MenuItem Header="Select All" InputGesture="Ctrl+A" />
        </ContextMenu>
    </TextBox.ContextMenu>
</TextBox>
```

## See Also

- [TextBox Control Reference](/controls/input/text-input/textbox): Property summary.
- [Validation in Data Binding](/docs/data-binding/binding-validation): Data annotation and INotifyDataErrorInfo validation.
- [Data Binding Syntax](/docs/data-binding/data-binding-syntax): Binding modes and parameters.
