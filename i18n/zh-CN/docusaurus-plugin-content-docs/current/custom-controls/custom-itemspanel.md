---
id: custom-itemspanel
title: 自定义 ItemsPanel
description: 使用诸如 Canvas 之类的自定义面板替换 ItemsControl 中默认的项目面板。
doc-type: how-to
---

import ItemsControlCanvasScreenshot from '/img/guides/ui-development/custom-controls/itemscontrol-with-canvas.png';

每个 `ItemsControl` 都会使用一个项目面板来排列其子元素。默认情况下，大多数控件会使用 `StackPanel` 或 `VirtualizingStackPanel` 来处理简单的垂直或水平列表。你可以替换这个面板，以实现内置默认面板不支持的布局方式。

以下控件都支持自定义 `ItemsPanel`：

- [`ItemsControl`](/controls/data-display/collections/itemscontrol)
- [`TreeView`](/controls/data-display/structured-data/treeview)
- [`Carousel`](/controls/data-display/collections/carousel)
- [`Menu`](/controls/menus/menu)
- [`ComboBox`](/controls/input/selectors/combobox)
- [`ListBox`](/controls/data-display/collections/listbox)

## 为什么要替换默认面板

默认面板很适合线性列表，但在某些场景下你可能需要不同的排列方式：

- **绝对定位**：使用 [`Canvas`](/api/avalonia/controls/canvas) 将项目放在特定坐标位置。
- **换行布局**：使用 `WrapPanel` 让项目在多行或多列中流动排列。
- **自定义排列**：使用[自定义面板](/docs/custom-controls/custom-panel)并编写你自己的 `ArrangeOverride` 逻辑，以支持径向布局、仪表盘或其他非线性设计。

:::warning[性能注意事项]
当你将默认面板替换为 `Canvas` 或 `WrapPanel` 这类不支持虚拟化的面板时，控件会为集合中的每一项都创建一个 UI 元素。对于大型集合（数百或数千项），这会显著增加内存占用并降低渲染性能。如果你需要在大数据集场景中实现自定义布局，请考虑构建一个继承自 `VirtualizingPanel` 的自定义面板，以保留虚拟化支持。
:::

## 示例

这个示例将一个包含矩形数据的 `ObservableCollection` 绑定到 `ItemsControl`。其中 `ItemsControl.ItemsPanel` 被设置为 `Canvas`，并通过样式结合附加属性来定位画布中的每个矩形。

```xml title="AXAML"
<ItemsControl ItemsSource="{Binding TileList}">
  <ItemsControl.ItemsPanel>
    <ItemsPanelTemplate>
      <Canvas Width="50" Height="50" Background="Yellow" Margin="3"/>
    </ItemsPanelTemplate>
  </ItemsControl.ItemsPanel>
  <ItemsControl.ItemTemplate>
    <DataTemplate>
      <Rectangle Fill="Green" Height="{Binding Size}" Width="{Binding Size}"/>
    </DataTemplate>
  </ItemsControl.ItemTemplate>
  <ItemsControl.Styles>
    <Style Selector="ContentPresenter"  x:DataType="vm:Tile">
      <Setter Property="Canvas.Left" Value="{Binding TopX}"/>
      <Setter Property="Canvas.Top" Value="{Binding TopY}"/>
    </Style>
  </ItemsControl.Styles>
</ItemsControl>
```

```csharp title='C# View Model'
using AvaloniaControls.Models;
using System.Collections.Generic;
using System.Collections.ObjectModel;

namespace AvaloniaControls.ViewModels
{
    public class MainWindowViewModel
    {
        public ObservableCollection<Tile> TileList { get; set; }
        
        public MainWindowViewModel()
        {
            TileList = new ObservableCollection<Tile>(new List<Tile>
            {
                new Tile(10, 10, 10),
                new Tile(10, 20, 20),
                new Tile(10, 30, 30),
            });    
        }
    }
}
```

```csharp title='C# Item Class'
public record Tile(int Size, int TopX, int TopY);
```

<Image light={ItemsControlCanvasScreenshot} alt="ItemsControl with a Canvas panel displaying positioned rectangles" position="center" maxWidth={400} cornerRadius="true"/>

## 另请参阅

- [创建自定义面板](/docs/custom-controls/custom-panel)
- [附加属性](/docs/custom-controls/attached-properties)
- [`ItemsControl`](/controls/data-display/collections/itemscontrol)
