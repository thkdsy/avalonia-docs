---
id: itemscontrol-how-to
title: "如何：使用 ItemsControl 和 ItemsRepeater"
description: 使用 ItemsControl 和 ItemsRepeater 构建超越 ListBox 能力的自定义集合布局。
doc-type: how-to
---

本指南介绍如何使用 [`ItemsControl`](/api/avalonia/controls/itemscontrol) 和 `ItemsRepeater` 来构建比 [`ListBox`](/api/avalonia/controls/listbox) 更灵活的自定义集合布局。

## 何时使用哪种控件

| 控件 | 选择能力 | 虚拟化 | 最适合的场景 |
|---|---|---|---|
| `ListBox` | 内置支持 | 是 | 可选列表 |
| `ItemsControl` | 无 | 默认不支持 | 小型集合、自定义布局 |
| `ItemsRepeater` | 无 | 是 | 大型集合、自定义布局、性能优先场景 |

如果你只是想展示一个集合，而不需要选择行为，就用 `ItemsControl`。如果你还需要在大数据集下保持虚拟化和更好的性能，则应使用 `ItemsRepeater`。

## ItemsControl 基础用法

`ItemsControl` 会使用数据模板来渲染每一项，但默认不带选择、高亮悬停或焦点样式：

```xml
<ItemsControl ItemsSource="{Binding Tags}">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <Border Background="#E0E0E0" CornerRadius="12" Padding="8,4" Margin="2">
                <TextBlock Text="{Binding}" />
            </Border>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
```

### 自定义面板

你可以通过 `ItemsPanel` 改变项目的排列方式：

```xml
<!-- 水平方向自动换行排列 -->
<ItemsControl ItemsSource="{Binding Tags}">
    <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
            <WrapPanel Orientation="Horizontal" />
        </ItemsPanelTemplate>
    </ItemsControl.ItemsPanel>
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <Border Background="#E8E8E8" CornerRadius="16" Padding="12,6" Margin="4">
                <TextBlock Text="{Binding Name}" />
            </Border>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
```

### 水平布局

```xml
<ItemsControl ItemsSource="{Binding Steps}">
    <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>
            <StackPanel Orientation="Horizontal" Spacing="16" />
        </ItemsPanelTemplate>
    </ItemsControl.ItemsPanel>
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <StackPanel Width="120">
                <Border Width="40" Height="40" CornerRadius="20"
                        Background="#6366F1" HorizontalAlignment="Center">
                    <TextBlock Text="{Binding Number}" Foreground="White"
                               HorizontalAlignment="Center" VerticalAlignment="Center" />
                </Border>
                <TextBlock Text="{Binding Title}" HorizontalAlignment="Center"
                           Margin="0,8,0,0" />
            </StackPanel>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
```

## ItemsRepeater

`ItemsRepeater` 是一个更底层、面向性能的控件。它支持虚拟化，也支持自定义布局算法。

### 基础用法

如果你希望 `ItemsRepeater` 能滚动，必须把它放在 `ScrollViewer` 内部：

```xml
<ScrollViewer>
    <ItemsRepeater ItemsSource="{Binding Items}">
        <ItemsRepeater.ItemTemplate>
            <DataTemplate>
                <Border Padding="8" Margin="0,0,0,4"
                        BorderBrush="#E0E0E0" BorderThickness="0,0,0,1">
                    <TextBlock Text="{Binding Title}" />
                </Border>
            </DataTemplate>
        </ItemsRepeater.ItemTemplate>
    </ItemsRepeater>
</ScrollViewer>
```

### 堆叠布局

默认布局会把项目按垂直方向堆叠排列。你也可以进一步配置间距：

```xml
<ItemsRepeater ItemsSource="{Binding Items}">
    <ItemsRepeater.Layout>
        <StackLayout Spacing="8" Orientation="Vertical" />
    </ItemsRepeater.Layout>
    <ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <Border Background="#F5F5F5" CornerRadius="4" Padding="12">
                <TextBlock Text="{Binding}" />
            </Border>
        </DataTemplate>
    </ItemsRepeater.ItemTemplate>
</ItemsRepeater>
```

### 水平堆叠

```xml
<ScrollViewer HorizontalScrollBarVisibility="Auto"
              VerticalScrollBarVisibility="Disabled">
    <ItemsRepeater ItemsSource="{Binding Cards}">
        <ItemsRepeater.Layout>
            <StackLayout Spacing="12" Orientation="Horizontal" />
        </ItemsRepeater.Layout>
        <ItemsRepeater.ItemTemplate>
            <DataTemplate>
                <Border Width="200" Height="150" Background="#6366F1"
                        CornerRadius="8" Padding="16">
                    <TextBlock Text="{Binding Title}" Foreground="White" />
                </Border>
            </DataTemplate>
        </ItemsRepeater.ItemTemplate>
    </ItemsRepeater>
</ScrollViewer>
```

### 自动换行布局（UniformGridLayout）

以自动换行的响应式网格方式显示项目：

```xml
<ScrollViewer>
    <ItemsRepeater ItemsSource="{Binding Photos}">
        <ItemsRepeater.Layout>
            <UniformGridLayout MinItemWidth="150" MinItemHeight="150"
                               MinRowSpacing="8" MinColumnSpacing="8" />
        </ItemsRepeater.Layout>
        <ItemsRepeater.ItemTemplate>
            <DataTemplate>
                <Border CornerRadius="4" ClipToBounds="True">
                    <Image Source="{Binding Thumbnail}" Stretch="UniformToFill" />
                </Border>
            </DataTemplate>
        </ItemsRepeater.ItemTemplate>
    </ItemsRepeater>
</ScrollViewer>
```

`UniformGridLayout` 会根据可用宽度以及 `MinItemWidth` 自动计算列数。

## 为 ItemsRepeater 添加点击处理

由于 `ItemsRepeater` 本身不提供内置选择能力，因此通常需要通过 `Button` 或 `Tapped` 事件自行实现点击处理：

```xml
<ItemsRepeater ItemsSource="{Binding Items}">
    <ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <Button Command="{Binding $parent[ItemsRepeater].((vm:MainViewModel)DataContext).SelectCommand}"
                    CommandParameter="{Binding}"
                    HorizontalAlignment="Stretch"
                    HorizontalContentAlignment="Stretch"
                    Background="Transparent" BorderThickness="0" Padding="0">
                <Border Padding="12" Background="#F8F8F8" CornerRadius="4">
                    <TextBlock Text="{Binding Name}" />
                </Border>
            </Button>
        </DataTemplate>
    </ItemsRepeater.ItemTemplate>
</ItemsRepeater>
```

## 空状态

当集合为空时，可以显示一个占位内容：

```xml
<Panel>
    <ScrollViewer>
        <ItemsRepeater ItemsSource="{Binding Items}"
                       IsVisible="{Binding Items.Count}">
            <!-- item template -->
        </ItemsRepeater>
    </ScrollViewer>

    <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center"
                IsVisible="{Binding !Items.Count}" Spacing="8">
        <PathIcon Data="{StaticResource EmptyIcon}" Width="48" Height="48"
                  Foreground="Gray" />
        <TextBlock Text="No items yet" Foreground="Gray" />
    </StackPanel>
</Panel>
```

## 使用 PreparingContainer 自定义容器

每当 `ItemsControl` 为某个数据项创建或重用容器时，都会触发 `PreparingContainer` 事件。你可以利用它来完成那些仅靠 `DataTemplate` 不方便表达的逐项定制逻辑，例如根据数据内容应用条件样式：

```csharp
myItemsControl.PreparingContainer += (sender, e) =>
{
    if (e.Item is TodoItem todo && todo.IsOverdue)
    {
        e.Container.Classes.Add("overdue");
    }
};
```

与之配套的 `ClearingContainer` 事件会在容器被回收或移除时触发，你可以在其中清理之前附加上的自定义内容。

## 性能建议

- 使用 `ItemsRepeater` 搭配 `StackLayout` 或 `UniformGridLayout`，以获得自动虚拟化能力。
- 在大型集合中，尽量避免在 `ItemsControl.ItemsPanel` 中使用 `WrapPanel`，因为它不支持虚拟化。
- 保持项目模板足够轻量，过于复杂的模板会拖慢滚动性能。
- 对于超大集合（例如 10,000 项以上），优先选择 `ItemsRepeater` 而不是 `ListBox`。

## 另请参阅

- [Performance](/docs/app-development/performance)：虚拟化与集合性能优化。
- [Data Templates](/docs/data-templates/introduction-to-data-templates)：数据模板的工作方式。
- [ListBox How-To](/docs/how-to/listbox-how-to)：当你需要选择行为时应使用的控件。
