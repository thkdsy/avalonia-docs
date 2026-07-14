---
id: radiobutton
title: RadioButton
description: 一个互斥选项控件，允许用户从一组选项中选择其中一个。
doc-type: reference
---

`RadioButton` 控件表示一组选项，用户一次只能选择其中一个。被选中的选项显示为实心圆，未选中的选项显示为空心圆。每个单选按钮的内容会作为标签显示在圆圈旁边。

一组单选按钮可以处于“没有任何选项被选中”的状态。不过，一旦用户选中了某个选项，仅通过用户输入无法再回到全部未选中的状态。若要以编程方式取消整组选中状态，请将组内每个单选按钮的 `IsChecked` 都设置为 `false`。

## 分组行为

具有相同 `GroupName` 值的单选按钮会组成一个互斥分组。当你选择其中一个选项时，同一分组中之前已选中的其他选项会自动取消选中。

如果你没有设置 `GroupName`，Avalonia 会按其父容器对单选按钮进行分组。同一个父面板中的所有 `RadioButton` 都会作为一个分组来行为。当你需要在同一个父容器中创建多个相互独立的分组时，请为每组分配不同的 `GroupName` 字符串。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 说明 |
| ----------- | ----------- |
| `GroupName` | 定义一组选项共用的名称，使它们以单选按钮的方式互斥交互。 |
| `IsChecked` | 单选按钮选项是否被选中（`true`）或未选中（`false`）。 |
| `IsEnabled` | 单选按钮选项是否启用。禁用选项会以淡化样式显示。 |
| `Content` | 显示在单选圆圈旁边的标签或内容。 |
| `Command` | 当单选按钮被选中时调用的 `ICommand`。 |

## 示例

此示例展示了两个彼此独立工作的单选按钮分组：

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui">
  <StackPanel Margin="20">
    <TextBlock Margin="0 10 0 5">First Group</TextBlock>
      <RadioButton GroupName="First Group"
                Content="First Option"/>
      <RadioButton GroupName="First Group"
                Content="Second Option"/>
      <RadioButton IsEnabled="False"
                GroupName="First Group"
                Content="Third Option"/>

    <TextBlock Margin="0 10 0 5">Second Group</TextBlock>
      <RadioButton GroupName="Second Group"
                Content="Fourth Option"/>
      <RadioButton GroupName="Second Group"
                Content="Fifth Option"/>
  </StackPanel>
</UserControl>
```

</XamlPreview>

## 绑定到视图模型

你可以将 `IsChecked` 绑定到视图模型中的布尔属性。这个示例为每个选项都使用了一个简单的布尔值：

```csharp
public partial class NotificationViewModel : ObservableObject
{
    [ObservableProperty]
    private bool _notifyByEmail = true;

    [ObservableProperty]
    private bool _notifyBySms;

    [ObservableProperty]
    private bool _noNotifications;
}
```

```xml
<StackPanel Spacing="4">
    <RadioButton GroupName="Notify"
                 Content="Email"
                 IsChecked="{Binding NotifyByEmail}" />
    <RadioButton GroupName="Notify"
                 Content="SMS"
                 IsChecked="{Binding NotifyBySms}" />
    <RadioButton GroupName="Notify"
                 Content="None"
                 IsChecked="{Binding NoNotifications}" />
</StackPanel>
```

## 绑定到枚举

一种常见模式是将单选按钮绑定到视图模型中的某个枚举属性。可以使用转换器将每个枚举值映射为一个布尔值：

```csharp
public enum ShippingMethod { Standard, Express, Overnight }

public partial class OrderViewModel : ObservableObject
{
    [ObservableProperty]
    private ShippingMethod _selectedShipping = ShippingMethod.Standard;
}
```

```xml
<StackPanel Spacing="4">
    <RadioButton Content="Standard (5-7 days)"
                 IsChecked="{Binding SelectedShipping,
                     Converter={StaticResource EnumToBoolConverter},
                     ConverterParameter={x:Static vm:ShippingMethod.Standard}}" />
    <RadioButton Content="Express (2-3 days)"
                 IsChecked="{Binding SelectedShipping,
                     Converter={StaticResource EnumToBoolConverter},
                     ConverterParameter={x:Static vm:ShippingMethod.Express}}" />
    <RadioButton Content="Overnight"
                 IsChecked="{Binding SelectedShipping,
                     Converter={StaticResource EnumToBoolConverter},
                     ConverterParameter={x:Static vm:ShippingMethod.Overnight}}" />
</StackPanel>
```

## 水平布局

你可以通过将单选按钮放入一个水平 `StackPanel` 中来实现水平排列：

```xml
<StackPanel Orientation="Horizontal" Spacing="16">
    <RadioButton GroupName="Size" Content="Small" />
    <RadioButton GroupName="Size" Content="Medium" IsChecked="True" />
    <RadioButton GroupName="Size" Content="Large" />
</StackPanel>
```

## 另请参阅

- [CheckBox](/controls/input/selectors/checkbox)
- [ToggleButton](/controls/input/buttons/togglebutton)
- [Button](/controls/input/buttons/button)
- [RadioButton API Reference](/api/avalonia/controls/radiobutton)
