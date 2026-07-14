---
id: listbox-how-to
title: "如何：使用 ListBox"
description: 了解 ListBox 的选择处理、项模板、虚拟化、样式以及高级使用模式。
doc-type: how-to
---

本指南介绍常见的 ListBox 使用场景：选择处理、项模板、虚拟化、样式以及高级模式。

## 项模板

使用 `ItemTemplate` 自定义每个项目的显示方式：

```xml
<ListBox ItemsSource="{Binding Contacts}">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <Grid ColumnDefinitions="48,*" Margin="4">
                <Border Grid.Column="0" Width="40" Height="40"
                        CornerRadius="20" Background="#E0E0E0"
                        ClipToBounds="True">
                    <TextBlock Text="{Binding Initials}"
                               HorizontalAlignment="Center"
                               VerticalAlignment="Center"
                               FontWeight="Bold" />
                </Border>
                <StackPanel Grid.Column="1" Margin="8,0,0,0"
                            VerticalAlignment="Center">
                    <TextBlock Text="{Binding Name}" FontWeight="SemiBold" />
                    <TextBlock Text="{Binding Email}" Foreground="Gray"
                               FontSize="12" />
                </StackPanel>
            </Grid>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

## 选择模式

### 单选（默认）

```xml
<ListBox SelectionMode="Single"
         SelectedItem="{Binding SelectedContact}"
         ItemsSource="{Binding Contacts}" />
```

### 多选

```xml
<ListBox SelectionMode="Multiple"
         ItemsSource="{Binding Contacts}" />
```

在多选模式下，用户可以通过点击项目来切换其选中状态。你可以通过 `SelectionChanged` 事件或 `SelectedItems` 属性访问已选项目。

### 切换选择

```xml
<ListBox SelectionMode="Toggle"
         ItemsSource="{Binding Contacts}" />
```

切换模式允许用户通过单击选中、再次单击取消选中，而无需按住 Ctrl。

### 始终保持选中

```xml
<ListBox SelectionMode="AlwaysSelected"
         ItemsSource="{Binding Contacts}" />
```

它会阻止用户取消所有项目的选择，始终至少保留一个选中项。

### 处理选择变化

```csharp
[ObservableProperty]
private Contact? _selectedContact;

partial void OnSelectedContactChanged(Contact? value)
{
    if (value is not null)
        LoadContactDetails(value);
}
```

或者使用事件方式：

```xml
<ListBox SelectionChanged="OnSelectionChanged" />
```

```csharp
private void OnSelectionChanged(object? sender, SelectionChangedEventArgs e)
{
    foreach (var added in e.AddedItems)
    {
        // 处理新选中的项目
    }
    foreach (var removed in e.RemovedItems)
    {
        // 处理取消选中的项目
    }
}
```

## 虚拟化

ListBox 默认启用虚拟化：它只会为可见项目创建控件。要让它生效，前提是 ListBox 的高度必须受到约束。

### 确保虚拟化已启用

```xml
<!-- 不推荐：StackPanel 提供无限高度，会导致虚拟化失效 -->
<StackPanel>
    <ListBox ItemsSource="{Binding LargeList}" />
</StackPanel>

<!-- 推荐：Grid 会约束高度 -->
<Grid RowDefinitions="*">
    <ListBox ItemsSource="{Binding LargeList}" />
</Grid>

<!-- 推荐：显式设置高度 -->
<ListBox ItemsSource="{Binding LargeList}" Height="400" />
```

### 滚动到某个项目

通过代码滚动，使某个项目进入可视区域：

```csharp
listBox.ScrollIntoView(targetItem);
```

或者按索引滚动到某项：

```csharp
listBox.ScrollIntoView(listBox.ItemsSource.ElementAt(50));
```

## 水平 ListBox

通过修改 items panel，让项目水平显示：

```xml
<ListBox ItemsSource="{Binding Tags}">
    <ListBox.ItemsPanel>
        <ItemsPanelTemplate>
            <WrapPanel />
        </ItemsPanelTemplate>
    </ListBox.ItemsPanel>
    <ListBox.ItemTemplate>
        <DataTemplate>
            <Border Background="#E8E8E8" CornerRadius="12" Padding="12,4">
                <TextBlock Text="{Binding}" />
            </Border>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

若要实现单行水平排列：

```xml
<ListBox ItemsSource="{Binding Items}">
    <ListBox.ItemsPanel>
        <ItemsPanelTemplate>
            <StackPanel Orientation="Horizontal" />
        </ItemsPanelTemplate>
    </ListBox.ItemsPanel>
</ListBox>
```

注意：使用 `StackPanel` 或 `WrapPanel` 会禁用虚拟化。对于大型水平列表，建议使用 `Orientation="Horizontal"` 的 `VirtualizingStackPanel`。

## 设置 ListBox 项样式

### 自定义选中外观

重写选中项目的显示效果：

```xml
<ListBox.Styles>
    <Style Selector="ListBoxItem:selected /template/ ContentPresenter">
        <Setter Property="Background" Value="#6366F1" />
    </Style>
    <Style Selector="ListBoxItem:selected /template/ ContentPresenter TextBlock">
        <Setter Property="Foreground" Value="White" />
    </Style>
    <Style Selector="ListBoxItem:pointerover /template/ ContentPresenter">
        <Setter Property="Background" Value="#F0F0F0" />
    </Style>
</ListBox.Styles>
```

### 移除选中高亮

如果你希望列表在选中时不显示明显的选中反馈：

```xml
<ListBox.Styles>
    <Style Selector="ListBoxItem">
        <Setter Property="Padding" Value="0" />
    </Style>
    <Style Selector="ListBoxItem:selected /template/ ContentPresenter">
        <Setter Property="Background" Value="Transparent" />
    </Style>
    <Style Selector="ListBoxItem:pointerover /template/ ContentPresenter">
        <Setter Property="Background" Value="Transparent" />
    </Style>
</ListBox.Styles>
```

### 项间距

在不修改模板的前提下，为项目之间增加间距：

```xml
<ListBox.Styles>
    <Style Selector="ListBoxItem">
        <Setter Property="Margin" Value="0,2" />
        <Setter Property="CornerRadius" Value="4" />
    </Style>
</ListBox.Styles>
```

## ListBox 项上的命令

在项目被点击时执行命令，并把该项目作为参数传入：

```xml
<ListBox ItemsSource="{Binding Items}">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <Button Content="{Binding Name}"
                    Command="{Binding $parent[ListBox].((vm:MainViewModel)DataContext).SelectItemCommand}"
                    CommandParameter="{Binding}"
                    HorizontalAlignment="Stretch"
                    Background="Transparent"
                    BorderThickness="0" />
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

## 空状态

当列表为空时显示提示信息：

```xml
<Panel>
    <ListBox ItemsSource="{Binding FilteredItems}"
             IsVisible="{Binding FilteredItems.Count}" />
    <TextBlock Text="No items found"
               IsVisible="{Binding !FilteredItems.Count}"
               HorizontalAlignment="Center"
               VerticalAlignment="Center"
               Foreground="Gray" />
</Panel>
```

## 带 CheckBox 的 ListBox

创建一个可勾选的列表：

```xml
<ListBox ItemsSource="{Binding Tasks}" SelectionMode="Toggle,Multiple">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <CheckBox Content="{Binding Title}"
                      IsChecked="{Binding IsCompleted}" />
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

## 项分组

使用“扁平列表 + 标题项”的模式来按组显示项目：

```csharp
public abstract class ListEntry { }

public class GroupHeader : ListEntry
{
    public string Title { get; init; } = "";
}

public class ContactEntry : ListEntry
{
    public Contact Contact { get; init; } = null!;
}
```

```xml
<ListBox ItemsSource="{Binding GroupedEntries}">
    <ListBox.DataTemplates>
        <DataTemplate DataType="local:GroupHeader">
            <TextBlock Text="{Binding Title}"
                       FontWeight="Bold" FontSize="14"
                       Margin="0,12,0,4" />
        </DataTemplate>
        <DataTemplate DataType="local:ContactEntry">
            <TextBlock Text="{Binding Contact.Name}" Margin="8,0" />
        </DataTemplate>
    </ListBox.DataTemplates>
    <ListBox.Styles>
        <!-- Make headers non-selectable -->
        <Style Selector="ListBoxItem:is(local:GroupHeader)">
            <Setter Property="IsHitTestVisible" Value="False" />
        </Style>
    </ListBox.Styles>
</ListBox>
```

有关如何构建分组列表的更多细节，请参阅 [Collection Views](/docs/data-binding/collection-views)。

## 另请参阅

- [ListBox Control Reference](/controls/data-display/collections/listbox): Property tables and basic examples.
- [Collection Views](/docs/data-binding/collection-views): Sorting, filtering, and grouping collections.
- [Performance](/docs/app-development/performance): Virtualization and large collection tips.
- [Data Templates](/docs/data-templates/introduction-to-data-templates): How templates work.
