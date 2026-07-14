---
id: itemsrepeater
title: ItemsRepeater
description: 用于显示绑定集合中重复数据的虚拟化、布局驱动控件。
doc-type: reference
---

import ItemsRepeaterVerticalScreenshot from '/img/controls/itemsrepeater/itemsrepeater-vertical.png';
import ItemsRepeaterHorizontalScreenshot from '/img/controls/itemsrepeater/itemsrepeater-horizontal.gif';

`ItemsRepeater` 用于显示绑定集合中的重复数据。与 [`ListBox`](/api/avalonia/controls/listbox) 不同，它不提供内置的选择、滚动或控件外观。相应地，它让你能够完全控制项目的布局与呈现方式，同时仍支持面向大型数据集的 UI 虚拟化。

你通常需要为 `ItemsRepeater` 提供两项内容：

- **数据源**：通过 `ItemsSource` 属性提供。
- **数据模板**：通过 `ItemTemplate` 属性提供，用于定义每个项目的显示方式。

默认情况下，项目会使用垂直 `StackLayout` 进行排列。你也可以通过设置 `Layout` 属性，将其改为水平 `StackLayout` 或 `UniformGridLayout`。

## 何时使用 `ItemsRepeater`

当你需要轻量级、支持虚拟化、但不带内置选择功能的列表或网格时，可以使用 `ItemsRepeater`。下表将它与相关控件进行了对比：

| 控件 | 选择功能 | 虚拟化 | 最适合 |
|---|---|---|---|
| `ListBox` | 内置 | 是 | 可选择的列表 |
| `ItemsControl` | 无 | 否（默认） | 小型集合、自定义布局 |
| `ItemsRepeater` | 无 | 是 | 大型集合、自定义布局、性能优先场景 |

如果你需要选择行为，可以考虑将每个项目包裹在 `Button` 中（参见下方的[处理点击事件](#处理点击事件)），或者直接改用 `ListBox`。

## 常用属性

以下属性通常最常使用：

| 属性 | 说明 |
|---|---|
| `ItemsSource` | 作为数据源使用的绑定集合。 |
| `ItemTemplate` | 定义每个项目视觉结构的 `DataTemplate`。 |
| `Layout` | 用于排列项目的布局策略，默认值为垂直 `StackLayout`。 |

## 垂直列表示例

此示例将一个餐具项目的可观察集合绑定到 `ItemsRepeater`，并通过数据模板提供自定义显示格式：

```xml
<StackPanel Margin="20">
  <TextBlock Margin="0 5">List of crockery:</TextBlock>
  <ItemsRepeater ItemsSource="{Binding CrockeryList}">
    <ItemsRepeater.ItemTemplate>
      <DataTemplate>
        <Border Margin="0,10,0,0"
                CornerRadius="5"
                BorderBrush="Blue" BorderThickness="1"
                Padding="5">
          <StackPanel Orientation="Horizontal">
            <TextBlock Text="{Binding Title}" />
            <TextBlock Margin="5 0" FontWeight="Bold"
                       Text="{Binding Number}" />
          </StackPanel>
        </Border>
      </DataTemplate>
    </ItemsRepeater.ItemTemplate>
  </ItemsRepeater>
</StackPanel>
```

```csharp title='C# View Model'
using AvaloniaControls.Models;
using System.Collections.Generic;
using System.Collections.ObjectModel;

namespace AvaloniaControls.ViewModels
{
    public class MainWindowViewModel : ViewModelBase
    {
        public ObservableCollection<Crockery> CrockeryList { get; set; }

        public MainWindowViewModel()
        {
            CrockeryList = new ObservableCollection<Crockery>(new List<Crockery>
            {
                new Crockery("dinner plate", 12),
                new Crockery("side plate", 12),
                new Crockery("breakfast bowl", 6),
                new Crockery("cup", 10),
                new Crockery("saucer", 10),
                new Crockery("mug", 6),
                new Crockery("milk jug", 1)
            });
        }
    }
}
```

```csharp title='C# Item Class'
public class Crockery
{
    public string Title { get; set; }
    public int Number { get; set; }

    public Crockery(string title, int number)
    {
        Title = title;
        Number = number;
    }
}
```

<Image light={ItemsRepeaterVerticalScreenshot} alt="ItemsRepeater 显示餐具项目的垂直列表" position="center" maxWidth={400} cornerRadius="true"/>

## 水平列表示例

你可以通过将 `Layout` 属性设置为水平 `StackLayout` 来横向显示项目。将 `ItemsRepeater` 包裹在 [`ScrollViewer`](/api/avalonia/controls/scrollviewer) 中，这样右侧溢出的项目仍然可以访问：

```xml
<StackPanel Margin="20">
  <TextBlock Margin="0 5">List of crockery:</TextBlock>
  <ScrollViewer HorizontalScrollBarVisibility="Auto">
    <ItemsRepeater ItemsSource="{Binding CrockeryList}" Margin="0 20">
      <ItemsRepeater.Layout>
        <StackLayout Spacing="40"
                     Orientation="Horizontal" />
      </ItemsRepeater.Layout>
      <ItemsRepeater.ItemTemplate>
        <DataTemplate>
          <Border Margin="0,10,0,0"
                  CornerRadius="5"
                  BorderBrush="Blue" BorderThickness="1"
                  Padding="5">
            <StackPanel Orientation="Horizontal">
              <TextBlock Text="{Binding Title}" />
              <TextBlock Margin="5 0" FontWeight="Bold"
                         Text="{Binding Number}" />
            </StackPanel>
          </Border>
        </DataTemplate>
      </ItemsRepeater.ItemTemplate>
    </ItemsRepeater>
  </ScrollViewer>
</StackPanel>
```

<Image light={ItemsRepeaterHorizontalScreenshot} alt="ItemsRepeater 显示带滚动条的水平餐具列表" position="center" maxWidth={400} cornerRadius="true"/>

## 布局选项

`Layout` 属性接受任意派生自 `AttachedLayout` 的对象。Avalonia 内置了以下两种常用选项：

### `StackLayout`

将项目排列为单行，可为垂直方向（默认）或水平方向。

| 属性 | 说明 |
|---|---|
| `Orientation` | `Vertical`（默认）或 `Horizontal`。 |
| `Spacing` | 每个项目之间的像素间距。 |

### `UniformGridLayout`

将项目排列到由等尺寸单元格组成的可换行网格中。列数会根据可用宽度自动调整，因此很适合响应式卡片布局。

```xml
<ScrollViewer>
    <ItemsRepeater ItemsSource="{Binding Products}">
        <ItemsRepeater.Layout>
            <UniformGridLayout MinItemWidth="250" MinItemHeight="180"
                               MinColumnSpacing="12" MinRowSpacing="12" />
        </ItemsRepeater.Layout>
        <ItemsRepeater.ItemTemplate>
            <DataTemplate>
                <Border Background="White" CornerRadius="8" Padding="16"
                        BorderBrush="#E5E7EB" BorderThickness="1">
                    <StackPanel Spacing="8">
                        <TextBlock Text="{Binding Name}" FontWeight="Bold" />
                        <TextBlock Text="{Binding Description}" TextWrapping="Wrap"
                                   Foreground="Gray" />
                    </StackPanel>
                </Border>
            </DataTemplate>
        </ItemsRepeater.ItemTemplate>
    </ItemsRepeater>
</ScrollViewer>
```

| 属性 | 说明 |
|---|---|
| `MinItemWidth` | 每个项目的最小宽度。 |
| `MinItemHeight` | 每个项目的最小高度。 |
| `MinColumnSpacing` | 项目之间的最小水平间距。 |
| `MinRowSpacing` | 项目之间的最小垂直间距。 |
| `MaximumRowsOrColumns` | 发生换行前允许的最大行数或列数。 |

## 处理点击事件

由于 `ItemsRepeater` 没有内置选择功能，你可以将每个项目包裹在 `Button` 中，并在点击时调用视图模型上的命令：

```xml
<ItemsRepeater ItemsSource="{Binding Items}">
    <ItemsRepeater.ItemTemplate>
        <DataTemplate>
            <Button Command="{Binding $parent[ItemsRepeater].((vm:MainViewModel)DataContext).SelectCommand}"
                    CommandParameter="{Binding}"
                    Background="Transparent" Padding="0">
                <Border Padding="12">
                    <TextBlock Text="{Binding Name}" />
                </Border>
            </Button>
        </DataTemplate>
    </ItemsRepeater.ItemTemplate>
</ItemsRepeater>
```

如果你更偏好基于事件的方式而不是命令，也可以在单个项目上处理 `Tapped` 或 `PointerPressed` 事件。

## 滚动与虚拟化

`ItemsRepeater` 不包含内置滚动视图。若要启用滚动，请将其包裹在 `ScrollViewer` 中：

```xml
<ScrollViewer>
    <ItemsRepeater ItemsSource="{Binding LargeCollection}">
        <ItemsRepeater.ItemTemplate>
            <DataTemplate>
                <TextBlock Text="{Binding}" Margin="4" />
            </DataTemplate>
        </ItemsRepeater.ItemTemplate>
    </ItemsRepeater>
</ScrollViewer>
```

当你将 `ItemsRepeater` 放入 `ScrollViewer` 中时，虚拟化会自动启用。此时只会创建当前可见的项目（外加少量缓冲项目），因此即使集合中有数千项，也能保持较低的内存占用和流畅的滚动体验。

## 另请参阅

- [ItemsControl](/controls/data-display/collections/itemscontrol)
- [ListBox](/controls/data-display/collections/listbox)
- [如何：使用 ItemsControl 和 ItemsRepeater](/docs/how-to/itemscontrol-how-to)
- [选择布局面板](/docs/layout/choosing-a-layout-panel)
- [响应式布局](/docs/how-to/responsive-layout-how-to)
