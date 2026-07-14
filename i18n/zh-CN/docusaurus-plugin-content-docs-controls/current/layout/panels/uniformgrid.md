---
id: uniformgrid
title: UniformGrid
description: 一个将子控件排列到每个单元格尺寸都相同的网格中的面板。
doc-type: reference
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

[`UniformGrid`](/api/avalonia/controls/primitives/uniformgrid) 会将可用空间划分为尺寸相同的单元格。你可以指定要创建多少行和列，而每个子控件会按照它们出现的顺序依次放入下一个可用单元格。与 `Grid` 不同，你不需要定义行定义和列定义，也不需要将子元素显式指定到某个单元格中。这使得 `UniformGrid` 成为工具栏、调色板或图标网格等简单、均匀布局场景中的理想选择。

## 常用属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `Rows` | `int` | 设置相等行的数量。当设为 `0`（默认值）时，会根据子元素数量和 `Columns` 值自动计算行数。 |
| `Columns` | `int` | 设置相等列的数量。当设为 `0`（默认值）时，会根据子元素数量和 `Rows` 值自动计算列数。 |
| `FirstColumn` | `int` | 设置第一个子元素所在列的偏移量。可用于在第一行开头预留空单元格。 |
| `RowSpacing` | `double` | 设置行之间的垂直间距。 |
| `ColumnSpacing` | `double` | 设置列之间的水平间距。 |

## 尺寸计算方式

当你同时设置 `Rows` 和 `Columns` 时，网格会精确创建对应数量的单元格。如果你只设置其中一个维度，则另一个维度会自动计算，以便容纳所有子元素。如果两者都未设置，`UniformGrid` 会默认使用接近正方形的排列方式。

每个单元格的宽度和高度都相同。单元格尺寸由总可用空间（减去间距后）在每个方向上平均分配得到。默认情况下，子元素会拉伸以填满其单元格，但你也可以通过各个子控件上的 `HorizontalAlignment` 和 `VerticalAlignment` 来控制这一行为。

## 基本示例

下面的示例创建了一个单行网格，其中包含三个尺寸相同的彩色矩形。

<XamlPreview>

```xml
<UniformGrid xmlns="https://github.com/avaloniaui"
             Rows="1" Columns="3"
             ColumnSpacing="10"
             Margin="20">
    <Rectangle Fill="Navy" />
    <Rectangle Fill="White" />
    <Rectangle Fill="Red" />
</UniformGrid>
```

</XamlPreview>

## 多行网格示例

下面的示例创建了一个具有 3 行 4 列的 `UniformGrid`，并用 12 个矩形填充它。每个 `Rectangle` 都会按行优先顺序自动分配到下一个单元格中。

<Tabs
  defaultValue="xaml"
  values={[
      { label: 'XAML', value: 'xaml', },
      { label: 'C#', value: 'cs', },
  ]}
>
<TabItem value="xaml">

```xml
<UniformGrid Rows="3" Columns="4">
  <Rectangle Width="50" Height="50" Fill="#330000"/>
  <Rectangle Width="50" Height="50" Fill="#660000"/>
  <Rectangle Width="50" Height="50" Fill="#990000"/>
  <Rectangle Width="50" Height="50" Fill="#CC0000"/>
  <Rectangle Width="50" Height="50" Fill="#FF0000"/>
  <Rectangle Width="50" Height="50" Fill="#FF3300"/>
  <Rectangle Width="50" Height="50" Fill="#FF6600"/>
  <Rectangle Width="50" Height="50" Fill="#FF9900"/>
  <Rectangle Width="50" Height="50" Fill="#FFCC00"/>
  <Rectangle Width="50" Height="50" Fill="#FFFF00"/>
  <Rectangle Width="50" Height="50" Fill="#FFFF33"/>
  <Rectangle Width="50" Height="50" Fill="#FFFF66"/>
</UniformGrid>
```

</TabItem>
<TabItem value="cs">

```csharp
// 创建 UniformGrid
var myUniformGrid = new UniformGrid
{
    Rows = 3,
    Columns = 4
};

// 添加 12 个带有颜色渐变的矩形
for (int i = 0; i < 12; i++)
{
    var rectangle = new Rectangle
    {
        Fill = new SolidColorBrush(Color.FromRgb((byte)(i * 20), 0, 0)),
        Width = 50,
        Height = 50
    };
    myUniformGrid.Children.Add(rectangle);
}
```

</TabItem>

</Tabs>

## 使用 `FirstColumn`

你可以使用 `FirstColumn` 属性为第一个子元素设置偏移，从而在第一行开头留出空单元格。当你希望内容不是从最左侧单元格开始时，这会非常有用。

```xml
<UniformGrid Rows="2" Columns="3" FirstColumn="1">
  <Button Content="A" />
  <Button Content="B" />
  <Button Content="C" />
  <Button Content="D" />
  <Button Content="E" />
</UniformGrid>
```

在这个示例中，第一行的第一个单元格是空的。按钮 “A” 会出现在第一行的第二列。

## 提示

- 如果你需要不同尺寸的单元格，请改用 `Grid`。
- 当你添加的子元素数量超过单元格数量时，额外的子元素仍会参与布局，但可能会出现在可见区域之外。
- `UniformGrid` 会遵守子控件上的 `Margin`，因此除了 `RowSpacing` 和 `ColumnSpacing` 之外，你还可以为单个项目添加额外间距。

## 另请参阅

- [Grid](/controls/layout/panels/grid)
- [WrapPanel](/controls/layout/panels/wrappanel)
- [StackPanel](/controls/layout/panels/stackpanel)
- [UniformGrid API reference](https://reference.avaloniaui.net/api/Avalonia.Controls.Primitives/UniformGrid/)
