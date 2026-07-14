---
id: itemscontrol
title: ItemsControl
description: Avalonia 中 ItemsControl 的参考文档，它是一个用于显示重复数据的基础控件，可完全控制布局和项目外观。
doc-type: reference
---

import exampleScreenshot from '/img/controls/itemscontrol/itemscontrol-with-custom-layout-and-formatting.gif';

[`ItemsControl`](/api/avalonia/controls/itemscontrol) 是用于显示重复数据的控件基类（例如 [`ListBox`](/api/avalonia/controls/listbox) 和 `ComboBox`）。它本身不包含内置的格式化、选择或滚动行为。你可以结合数据绑定、样式和数据模板，创建一个完全自定义的重复数据展示控件。

:::tip
如果你需要内置的选择支持，请改用 [`ListBox`](/controls/data-display/collections/listbox)。如果你需要滚动功能，可以将 `ItemsControl` 包裹在 `ScrollViewer` 中，或者直接考虑默认已包含滚动支持的 `ListBox`。
:::

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 说明 |
|---|---|
| `ItemsSource` | 作为控件数据源的绑定集合。 |
| `ItemTemplate` | 应用于每个项目的 `DataTemplate`。可用于控制单个项目的显示外观。 |
| `ItemsPanel` | 用于承载生成项目的面板。默认值为 `StackPanel`。若要替换它，请参阅 [custom ItemsPanel](/docs/custom-controls/custom-itemspanel)。 |
| `Styles` | 应用于 `ItemsControl` 子元素的样式。 |
| `DisplayMemberBinding` | 当你未提供 `ItemTemplate` 时，用于选择要显示的属性的绑定。 |

## 实用说明

- **请为 `ItemsSource` 使用 `ObservableCollection<T>`**，这样当你在运行时添加或移除项目时，UI 就能自动更新。普通的 `List<T>` 不会通知控件数据发生了变化。
- **`ItemsControl` 不会对其子元素进行虚拟化**。如果你需要处理大量项目，请考虑使用带虚拟化布局的 [`ItemsRepeater`](/controls/data-display/collections/itemsrepeater)，或者默认启用虚拟化的 `ListBox`。
- To arrange items horizontally instead of vertically, replace the default `ItemsPanel`:

```xml
<ItemsControl.ItemsPanel>
  <ItemsPanelTemplate>
    <WrapPanel Orientation="Horizontal" />
  </ItemsPanelTemplate>
</ItemsControl.ItemsPanel>
```

- 由于 `ItemsControl` 没有内置滚动条，超出可用高度的内容会被裁剪。如果你需要滚动支持，请将该控件包裹在 `ScrollViewer` 中：

```xml
<ScrollViewer>
  <ItemsControl ItemsSource="{Binding MyItems}" />
</ScrollViewer>
```

## 示例

此示例将一个餐具项目的可观察集合绑定到 `ItemsControl` 上。每个项目都通过 `DataTemplate` 提供自定义布局与格式：

<Image light={exampleScreenshot} alt="ItemsControl displaying a formatted list of crockery items" position="center" maxWidth={400} cornerRadius="true"/>

```xml title="XAML"
<StackPanel Margin="20">
  <TextBlock Margin="0 5">List of crockery:</TextBlock>
  <ItemsControl ItemsSource="{Binding CrockeryList}">
    <ItemsControl.ItemTemplate>
      <DataTemplate>
        <Border Margin="0,10,0,0"
                CornerRadius="5"
                BorderBrush="Gray" BorderThickness="1"
                Padding="5">
          <StackPanel Orientation="Horizontal">
            <TextBlock Text="{Binding Title}" />
            <TextBlock Margin="5 0" FontWeight="Bold"
                       Text="{Binding Number}" />
          </StackPanel>
        </Border>
      </DataTemplate>
    </ItemsControl.ItemTemplate>
  </ItemsControl>
</StackPanel>
```

```csharp title="C# view model"
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

```csharp title="C# item class"
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

## 另请参阅

- [ListBox](/controls/data-display/collections/listbox)
- [ItemsRepeater](/controls/data-display/collections/itemsrepeater)
- [Carousel](/controls/data-display/collections/carousel)
- [DataGrid](/controls/data-display/structured-data/datagrid)
- [Custom ItemsPanel](/docs/custom-controls/custom-itemspanel)
- [Data templates](/docs/data-templates/introduction-to-data-templates)
- [ItemsControl API reference](/api/avalonia/controls/itemscontrol)
- [`ItemsControl.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/ItemsControl.cs)
