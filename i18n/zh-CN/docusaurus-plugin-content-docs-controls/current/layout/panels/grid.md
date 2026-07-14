---
id: grid
title: Grid
description: 了解如何在 Avalonia 中使用 Grid 面板按行和列排列子控件。
doc-type: reference
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import GridSharedSizeGroupScreenshot from '/img/controls/grid/grid-sharedsizegroup.png';
import GridSampleScreenshot from '/img/controls/grid/grid_example.png';

[`Grid`](/api/avalonia/controls/grid) 控件适用于按列和行排列子控件。你可以为 `Grid` 定义绝对尺寸、比例尺寸以及自动尺寸的行列布局。

`Grid` 中的每个子控件都可以通过列和行坐标放置到某个单元格中。这些坐标从零开始，并且默认值都为 0。

如果你将多个子控件放在同一个单元格中，它们会按照在 XAML 中出现的顺序绘制。这是除 `Panel` 之外实现图层堆叠的另一种策略。

:::caution
如果你省略了 `Grid` 子控件的列和行坐标，它们都会被绘制到左上角（column=0，row=0）。
:::

你还可以让子控件跨越多行、多列，或同时跨越多行和多列。

## 常用属性

你最常使用的通常是以下属性：

| 属性 | 说明 |
|------------------------|---------------------------------------------------------------------|
| ColumnDefinitions | 描述 `Grid` 中各列宽度的尺寸定义。 |
| RowDefinitions | 描述 `Grid` 中各行高度的尺寸定义。 |
| ShowGridLines | 显示单元格之间的网格线（虚线）。 |
| Grid.Column | 将控件布局到指定的零基列中。 |
| Grid.Row | 将控件布局到指定的零基行中。 |
| Grid.ColumnSpan | 让控件跨越一列或多列。 |
| Grid.RowSpan | 让控件跨越一行或多行。 |
| Grid.IsSharedSizeScope | 将该控件定义为 `SharedSizeGroup` 的包含作用域 |

## 尺寸定义

你可以将行和列的尺寸定义为：

- Absolute - 以设备无关像素（整数）为单位的固定尺寸
- Proportional - 按 `Grid` 剩余空间比例分配的尺寸
- Automatic - 根据其中子控件自动适配的尺寸

尺寸定义既可以写成简写列表，也可以展开为完整的 XAML 元素形式。

完整写法支持额外约束，例如 `SharedSizeGroup`，以及指定绝对尺寸下的最小值和最大值。

### 绝对尺寸定义

绝对尺寸定义在列表格式中写作整数。例如：

`ColumnDefinitions="200, 200, 300"`

使用完整展开的 XAML，效果等同于：

```xml
<Grid>
   <Grid.ColumnDefinitions>
       <ColumnDefinition Width="200"></ColumnDefinition>
       <ColumnDefinition Width="200"></ColumnDefinition>
       <ColumnDefinition Width="300"></ColumnDefinition>
   </Grid.ColumnDefinitions>
</Grid>
```

### 比例尺寸定义

比例尺寸定义使用星号表示占用可用 `Grid` 空间的比例。例如，若要创建两个同宽列，再加上一列宽度为其两倍的列：

`ColumnDefinitions="*, *, 2*"`

使用完整展开的 XAML，效果等同于：

```xml
<Grid>
   <Grid.ColumnDefinitions>
       <ColumnDefinition Width="*"></ColumnDefinition>
       <ColumnDefinition Width="*"></ColumnDefinition>
       <ColumnDefinition Width="2*"></ColumnDefinition>
   </Grid.ColumnDefinitions>
</Grid>
```

:::tip
尺寸定义不支持百分比。一个常用技巧是让所有比例值相加为 100，例如 `<Grid ColumnDefinitions="25*, 25*, 50*">`，这样三列就会分别占剩余可用宽度的 25%、25% 和 50%。
:::

### 自动尺寸定义

若要让某行或某列自动适应其中最大的子控件，请使用 `Auto`。例如：

`RowDefinitions="Auto, Auto, Auto"`

使用完整展开的 XAML，效果等同于：

```xml
<Grid>
   <Grid.RowDefinitions>
       <RowDefinition Height="Auto"></RowDefinition>
       <RowDefinition Height="Auto"></RowDefinition>
       <RowDefinition Height="Auto"></RowDefinition>
   </Grid.RowDefinitions>
</Grid>
```

:::caution
如果某个子控件显式设置了自己的尺寸，那么在绘制时会遵循该尺寸。这意味着当它大于所在网格单元格时，可能会覆盖相邻单元格。
:::

### 混合尺寸定义

你可以在同一组尺寸定义中混合使用上述任意方式。例如：

`ColumnDefinitions="200, *, 2*"`

使用完整展开的 XAML，效果等同于：

```xml
<Grid>
   <Grid.ColumnDefinitions>
       <ColumnDefinition Width="200"></ColumnDefinition>
       <ColumnDefinition Width="*"></ColumnDefinition>
       <ColumnDefinition Width="2*"></ColumnDefinition>
   </Grid.ColumnDefinitions>
</Grid>
```

## 绘制规则

在计算尺寸时，比例列会使用在绝对尺寸和自动尺寸计算完成后剩余的空间。

自动尺寸的计算会基于子控件 margin 布局区域的外侧边界进行。

:::info
如需回顾控件布局区域的概念，请参阅 [布局区域](/docs/layout/#layout-zones)。
:::

子控件会按照它们在 XAML 中出现的顺序，绘制到各自分配的网格单元中。这条规则既决定了多个子控件位于同一单元格时的绘制顺序，也决定了子控件大于分配单元格时相互覆盖的方式。

当子控件拥有自己的尺寸，并且小于所分配的单元格时，它会根据自身的水平和垂直对齐属性在单元格中进行对齐绘制（两者默认都为居中）。

## 示例

本示例展示了：

- 如何使用列和行定义的简写语法。
- 如何混合使用绝对列宽和比例列宽。
- 如何为子控件指定所在单元格。
- 如何跨行和跨列。

下面是一个 `Grid` 示例：具有 3 个等高行和 3 个列，其中 1 列固定宽度，另 2 列按比例占用剩余空间：

在这里，先减去第 0 列固定的 100 宽度后，第 1 列会获得剩余宽度中的 1.5 份，第 2 列会获得 4 份。

按钮会从单元格（column 1，row 1）开始绘制，并额外跨越右侧一列和下方一行。效果如下：

<XamlPreview>

```xml
<Grid xmlns="https://github.com/avaloniaui"
      HorizontalAlignment="Center"
      VerticalAlignment="Center"
      RowSpacing="10"
      ColumnDefinitions="100,1.5*,4*" RowDefinitions="Auto,Auto,Auto"  Margin="4">
  <TextBlock Grid.Row="0" Grid.Column="0"
             Text="Col0Row0:" />
  <TextBlock Grid.Row="1" Grid.Column="0"
             Text="Col0Row1:" />
  <TextBlock Grid.Row="2" Grid.Column="0"
             Text="Col0Row2:" />
  <TextBlock Grid.Row="0" Grid.Column="2"
             Text="Col2Row0" />
  <Button Grid.Row="1" Grid.Column="1"
          Grid.RowSpan="2" Grid.ColumnSpan="2"
          HorizontalAlignment="Stretch"
          Content="SpansCol1-2Row1-2" />
</Grid>
```

</XamlPreview>

## SharedSizeGroup

`SharedSizeGroup` allows sharing size information for autosized row and column definitions across multiple `Grid` controls.

下面的示例演示了如何使用 `SharedSizeGroup`，让 `ListBox` 内外的列保持一致尺寸。

<Tabs>
<TabItem value="xml" label="XML" default>

```xml
<StackPanel Grid.IsSharedSizeScope="True">
  <StackPanel.Styles>
    <Style Selector="ListBoxItem">
      <Setter Property="Padding" Value="0" />
    </Style>
  </StackPanel.Styles>

  <ListBox ItemsSource="{Binding People}">
    <ListBox.ItemTemplate>
      <DataTemplate>
        <Grid Name="myGrid" RowDefinitions="auto, auto" ShowGridLines="True">
          <Grid.ColumnDefinitions>
            <ColumnDefinition SharedSizeGroup="A" />
            <ColumnDefinition SharedSizeGroup="B" />
            <ColumnDefinition Width="*" />
            <ColumnDefinition SharedSizeGroup="C" />
          </Grid.ColumnDefinitions>

          <TextBlock Grid.Column="0" Margin="6,0" Text="{Binding FirstName}" />
          <TextBlock Grid.Column="1" Margin="6,0" Text="{Binding LastName}" />
          <TextBlock Grid.Column="2" Margin="6,0" Text="{Binding Age}" />
          <TextBlock Grid.Column="3" Margin="6,0" Text="{Binding Occupation}" />
        </Grid>
      </DataTemplate>
    </ListBox.ItemTemplate>
  </ListBox>
    
  <!-- Controls may appear in-between Grids with SharedSizeGroups -->
  <Separator />

  <Grid>
    <Grid.ColumnDefinitions>
      <ColumnDefinition SharedSizeGroup="A" />
      <ColumnDefinition SharedSizeGroup="B" />
      <ColumnDefinition Width="*" />
      <ColumnDefinition SharedSizeGroup="C" />
    </Grid.ColumnDefinitions>

    <Button Content="This is the First Name" HorizontalAlignment="Stretch" Grid.Column="0" />
    <Button Content="Last" HorizontalAlignment="Stretch" Grid.Column="1" />
    <Button Content="Age" HorizontalAlignment="Stretch" Grid.Column="2" />
    <Button Content="Occupation" HorizontalAlignment="Stretch" Grid.Column="3" />
  </Grid>

</StackPanel>
```

</TabItem>
<TabItem value="example" label="C#">

```csharp
public record Person(string FirstName, string LastName, int Age, string Occupation);

public partial class MainWindowViewModel : ViewModelBase
{
    public ObservableCollection<Person> People { get; } = new()
    {
        new("Jim", "Smith", 35, "Printed Circuit Board Drafter"),
        new("Charlotte", "O'Shaughnessy-Alejandro", 30, "Librarian"),
        new("Ryan", "Cullen", 40, "Ceramics Instructor"),
        new("Valentina", "Levine", 38, "Oceanologist")
    };
}
```

</TabItem>
</Tabs>

<Image light={GridSharedSizeGroupScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

请注意各列的尺寸分配方式：第一列由 `Button` 决定宽度，第二列和第四列由 `ListBox` 内容决定宽度，第三列则占用剩余空间。

## 在代码中定义 Grid

下面的示例演示了如何构建一个类似于 Windows 开始菜单中“运行”对话框的 UI。

<Image light={GridSampleScreenshot} alt="Grid example app" position="center" maxWidth={400} cornerRadius="true" />

<Tabs
  defaultValue="xaml"
  values={[
      { label: 'XAML', value: 'xaml', },
      { label: 'C#', value: 'cs', },
  ]}
>
<TabItem value="xaml">

```xml
<Grid Background="Gainsboro" 
      HorizontalAlignment="Left" 
      VerticalAlignment="Top" 
      Width="425" 
      Height="165"
      ColumnDefinitions="Auto,*,*,*,*"
      RowDefinitions="Auto,Auto,*,Auto">
    
    <Image Grid.Row="0" Grid.Column="0" Source="{Binding runicon}" />
    
    <TextBlock Grid.Row="0" Grid.Column="1" Grid.ColumnSpan="4" 
               Text="Type the name of a program, folder, document, or Internet resource, and Windows will open it for you." 
               TextWrapping="Wrap" />
               
    <TextBlock Grid.Row="1" Grid.Column="0" Text="Open:" />
    
    <TextBox Grid.Row="1" Grid.Column="1" Grid.ColumnSpan="5" />
    
    <Button Grid.Row="3" Grid.Column="2" Content="OK" Margin="10,0,10,15" />
    
    <Button Grid.Row="3" Grid.Column="3" Content="Cancel" Margin="10,0,10,15" />
    
    <Button Grid.Row="3" Grid.Column="4" Content="Browse ..." Margin="10,0,10,15" />
</Grid>

```

</TabItem>
<TabItem value="cs">

```cs
// 创建 Grid。
grid1 = new Grid ();
grid1.Background = Brushes.Gainsboro;
grid1.HorizontalAlignment = HorizontalAlignment.Left;
grid1.VerticalAlignment = VerticalAlignment.Top;
grid1.ShowGridLines = true;
grid1.Width = 425;
grid1.Height = 165;

// 定义列。
colDef1 = new ColumnDefinition();
colDef1.Width = new GridLength(1, GridUnitType.Auto);
colDef2 = new ColumnDefinition();
colDef2.Width = new GridLength(1, GridUnitType.Star);
colDef3 = new ColumnDefinition();
colDef3.Width = new GridLength(1, GridUnitType.Star);
colDef4 = new ColumnDefinition();
colDef4.Width = new GridLength(1, GridUnitType.Star);
colDef5 = new ColumnDefinition();
colDef5.Width = new GridLength(1, GridUnitType.Star);
grid1.ColumnDefinitions.Add(colDef1);
grid1.ColumnDefinitions.Add(colDef2);
grid1.ColumnDefinitions.Add(colDef3);
grid1.ColumnDefinitions.Add(colDef4);
grid1.ColumnDefinitions.Add(colDef5);

// 定义行。
rowDef1 = new RowDefinition();
rowDef1.Height = new GridLength(1, GridUnitType.Auto);
rowDef2 = new RowDefinition();
rowDef2.Height = new GridLength(1, GridUnitType.Auto);
rowDef3 = new RowDefinition();
rowDef3.Height = new GridLength(1, GridUnitType.Star);
rowDef4 = new RowDefinition();
rowDef4.Height = new GridLength(1, GridUnitType.Auto);
grid1.RowDefinitions.Add(rowDef1);
grid1.RowDefinitions.Add(rowDef2);
grid1.RowDefinitions.Add(rowDef3);
grid1.RowDefinitions.Add(rowDef4);

// Add the Image.
img1 = new Image();
img1.Source = runicon;
Grid.SetRow(img1, 0);
Grid.SetColumn(img1, 0);

// Add the main application dialog.
txt1 = new TextBlock();
txt1.Text = "Type the name of a program, folder, document, or Internet resource, and Windows will open it for you.";
txt1.TextWrapping = TextWrapping.Wrap;
Grid.SetColumnSpan(txt1, 4);
Grid.SetRow(txt1, 0);
Grid.SetColumn(txt1, 1);

// Add the second text cell to the Grid.
txt2 = new TextBlock();
txt2.Text = "Open:";
Grid.SetRow(txt2, 1);
Grid.SetColumn(txt2, 0);

// Add the TextBox control.
tb1 = new TextBox();
Grid.SetRow(tb1, 1);
Grid.SetColumn(tb1, 1);
Grid.SetColumnSpan(tb1, 5);

// Add the buttons.
button1 = new Button();
button2 = new Button();
button3 = new Button();
button1.Content = "OK";
button2.Content = "Cancel";
button3.Content = "Browse ...";
Grid.SetRow(button1, 3);
Grid.SetColumn(button1, 2);
button1.Margin = new Thickness(10, 0, 10, 15);
button2.Margin = new Thickness(10, 0, 10, 15);
button3.Margin = new Thickness(10, 0, 10, 15);
Grid.SetRow(button2, 3);
Grid.SetColumn(button2, 3);
Grid.SetRow(button3, 3);
Grid.SetColumn(button3, 4);

grid1.Children.Add(img1);
grid1.Children.Add(txt1);
grid1.Children.Add(txt2);
grid1.Children.Add(tb1);
grid1.Children.Add(button1);
grid1.Children.Add(button2);
grid1.Children.Add(button3);
```
</TabItem>  

</Tabs>

## See also

- [Grid API reference](/api/avalonia/controls/grid)
- [`Grid.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Grid.cs)
- [GridSplitter](/controls/layout/panels/gridsplitter)
- [Canvas](/controls/layout/panels/canvas)
- [DockPanel](/controls/layout/panels/dockpanel)
- [Panel](/controls/layout/panels/panel)
- [RelativePanel](/controls/layout/panels/relativepanel)
- [StackPanel](/controls/layout/panels/stackpanel)
- [UniformGrid](/controls/layout/panels/uniformgrid)
- [WrapPanel](/controls/layout/panels/wrappanel)
