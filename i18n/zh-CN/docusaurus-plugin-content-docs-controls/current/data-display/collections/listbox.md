---
id: listbox
title: ListBox
---

import ListBoxStringScreenshot from '/img/controls/listbox/listbox-string.gif';
import ListBoxDataTemplateScreenshot from '/img/controls/listbox/listbox-datatemplate.gif';
import ListBoxDevToolsScreenshot from '/img/controls/listbox/listbox-devtools.png';
import ListBoxItemStyleScreenshot from '/img/controls/listbox/listbox-item-style.gif';

`ListBox` 会按多行方式显示来自某个项目源集合的项目，并支持单选或多选。

列表中的项目可以进行组合、绑定和模板化显示。

除非通过高度属性显式设置，或由外层控件（例如 [DockPanel](/controls/layout/panels/dockpanel)）限制，否则列表高度会自动扩展以容纳所有项目。

当高度受到限制，并且项目总高度超过该限制时，ListBox 内置的滚动视图就会显示垂直滚动条。

同样地，当某个项目的宽度超过了 ListBox 的宽度时，ListBox 内置的滚动视图也会显示水平滚动条（除非被禁止，见下文）。

## 常用属性

你最常使用的通常是这些属性：

<table>
  <thead>
    <tr><th width="289">属性</th>
    <th>说明</th></tr>
  </thead>
  <tbody>
    <tr>
      <td><code>Items</code></td>
      <td></td>
    </tr>
    <tr>
      <td><code>SelectedIndex</code></td>
      <td>选中项的索引（从零开始）；在多选情况下，则表示第一个被选中的项目。</td>
    </tr>
    <tr>
      <td><code>SelectedItem</code></td>
      <td>项目集合中当前选中的项目（对象）；在多选情况下，则表示第一个被选中的项目。</td>
    </tr>
    <tr>
      <td><code>SelectedItems</code></td>
      <td>列表中所有被选中的项目。</td>
    </tr>
    <tr>
      <td><code>Selection</code></td>
      <td>一个 <code>ISelectionModel</code> 对象，提供多种方法来跟踪多个被选中的项目。它针对大规模项目集合做了优化。</td>
    </tr>
    <tr>
      <td><code>SelectionMode</code></td>
      <td>选择模式，见下表。</td>
    </tr>
    <tr>
      <td><p><code>ScrollViewer.Horizontal</code></p><p><code>ScrollBarVisibility</code></p></td>
      <td>内置滚动视图的水平滚动条可见性。可选值为 `Disabled`（默认）、`Auto`、`Hidden` 和 `Visible`。禁用时，溢出内容会被隐藏。</td>
    </tr>
    <tr>
      <td><p><code>ScrollViewer.Vertical</code></p><p><code>ScrollBarVisibility</code></p></td>
      <td>内置滚动视图的垂直滚动条可见性。可选值为 `Disabled`、`Auto`（默认）、`Hidden` 和 `Visible`。禁用时，溢出内容会被隐藏。</td>
    </tr>
    <tr>
      <td><code>ItemsPanel</code></td>
      <td>用于放置项目的容器面板。若要自定义 ItemsPanel，请参阅[此页面](/docs/custom-controls/custom-itemspanel)。</td>
    </tr>
    <tr>
      <td><code>Styles</code></td>
      <td>应用到 `ItemsControl` 任意子元素上的样式。</td>
    </tr>
  </tbody>
</table>

:::info
当项目集合较大时，建议使用 `ISelectionModel` 以优化性能。
:::

## 选择模式

在触摸设备和手写笔设备上，选择会在指针释放时发生，而不是按下时。这使得用户可以在不改变选择状态的情况下，从某个项目上开始滑动或滚动手势。

ListBox 可使用以下选择模式：

<table><thead><tr><th width="237">选择模式</th><th>说明</th></tr></thead><tbody><tr><td><code>Single</code></td><td>只能选中单个项目（默认）。</td></tr><tr><td><code>Multiple</code></td><td>可以选中多个项目。</td></tr><tr><td><code>Toggle</code></td><td>可通过点击/空格键切换项目的选中状态。若未启用该模式，则需要使用 shift 或 ctrl 才能选择多个项目。</td></tr><tr><td><code>AlwaysSelected</code></td><td>只要列表中还有可选项目，就始终会有一个项目保持选中。</td></tr></tbody></table>

这些值可以组合使用，例如：

```xml
<ListBox SelectionMode="Multiple,Toggle">
```

## 示例

此示例在 C# 代码后置中将 `ItemsSource` 属性设置为一个数组。

```xml
<StackPanel Margin="20">
  <TextBlock Margin="0 5">Choose an animal:</TextBlock>
  <ListBox x:Name="animals"/>
</StackPanel>
```

```csharp title='C#'
using Avalonia.Controls;
using System.Linq;

namespace AvaloniaControls.Views
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
            animals.ItemsSource = new string[]
                {"cat", "camel", "cow", "chameleon", "mouse", "lion", "zebra" }
            .OrderBy(x => x);
        }
    }
}
```

<Image light={ListBoxStringScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 项目模板

你可以通过在列表框的 `ItemTemplate` 元素中使用 **数据模板**，来自定义项目的显示方式。

:::info
若要回顾 **数据模板** 的概念，请参阅 [数据模板简介](/docs/data-templates/introduction-to-data-templates)。
:::

这个示例会将每个项目显示在一个带有圆角的蓝色边框内。C# 代码后置与前一个示例相同：

```xml
<DockPanel Margin="20">
  <TextBlock Margin="0 5" DockPanel.Dock="Top">Choose an animal:</TextBlock>
  <ListBox x:Name="animals">
    <ListBox.ItemTemplate>
      <DataTemplate>
        <Border BorderBrush="Blue" BorderThickness="1" 
                CornerRadius="4" Padding="4">
          <TextBlock Text="{Binding}"/>
        </Border>
      </DataTemplate>
    </ListBox.ItemTemplate>
  </ListBox>
</DockPanel>
```

```csharp title='C#'
using Avalonia.Controls;
using System.Linq;

namespace AvaloniaControls.Views
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
            animals.ItemsSource = new string[]
                {"cat", "camel", "cow", "chameleon", "mouse", "lion", "zebra" }
            .OrderBy(x => x);
        }
    }
}
```

此处列表被放在 DockPanel 的填充区域，因此它的高度会占据剩余空间，这样就能显示出 ListBox 中的滚动条。

<Image light={ListBoxDataTemplateScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 项目样式

显示在 ListBox 中的每个项目，都会绘制在一个 [`ListBoxItem`](/api/avalonia/controls/listboxitem) 元素内。你可以通过 _Avalonia Dev Tools_（F12）中的 **Visual Tools** 选项卡查看这一点。例如：

<Image light={ListBoxDevToolsScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

`ListBoxItem` 元素作为 `ListBox.ItemTemplate` 中指定内容的容器；但它并不会在 XAML 中显式定义，而是由 _Avalonia_ 自动生成的。

这意味着你可以通过样式选择器来定制 ListBox 中的 `ListBoxItem` 元素。例如，下面让列表项具有固定宽度 200，并让它们右对齐：

```xml
<DockPanel Margin="20">
  <TextBlock Margin="0 5" DockPanel.Dock="Top">Choose an animal:</TextBlock>
  <ListBox x:Name="animals">
    <ListBox.Styles>
      <Style Selector="ListBoxItem">
        <Setter Property="Width" Value="200"/>
        <Setter Property="HorizontalAlignment" Value="Right"/>
      </Style>
    </ListBox.Styles>
  </ListBox>
</DockPanel>
```

```csharp title='C#'
using Avalonia.Controls;
using System.Linq;

namespace AvaloniaControls.Views
{
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
            animals.ItemsSource = new string[]
                {"cat", "camel", "cow", "chameleon", "mouse", "lion", "zebra" }
            .OrderBy(x => x);
        }
    }
}
```

<Image light={ListBoxItemStyleScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 另请参阅

- [ListBox API 参考](/api/avalonia/controls/listbox)
- [`ListBox.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/ListBox.cs)
