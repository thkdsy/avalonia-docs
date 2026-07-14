---
id: combobox
title: ComboBox
description: 一个下拉选择控件，允许用户从列表中选择单个项目，并支持可编辑文本输入和占位提示。
doc-type: reference
---

import ComboBoxDataTemplateScreenshot from '/img/controls/combobox/combobox-data-template.gif';

`ComboBox` 会显示当前选中的项目，并带有一个可展开选项列表的下拉按钮。除非你显式指定，否则组合框的宽度和高度会由当前选中项决定。

你可以对列表中的项目进行组合、绑定和模板化显示。

:::info
若要回顾 **数据模板** 的概念，请参阅 [Introduction to data templates](/docs/data-templates/introduction-to-data-templates)。
:::

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 说明 |
| -------------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------ |
| `Items` | `ItemCollection` | 列表项集合。 |
| `SelectedIndex` | `int` | 选中项的索引（从 0 开始）。 |
| `SelectedItem` | `object?` | 当前选中的项目本身。 |
| `SelectedValue` | `object?` | 选中项的值，由 `SelectedValueBinding` 决定。 |
| `IsEditable` | `bool` | 启用文本编辑，使你可以在组合框中输入内容以筛选项目或输入自定义值。 |
| `Text` | `string?` | 当 `IsEditable` 为 `true` 时，获取或设置文本值。 |
| `PlaceholderText` | `string?` | 未选择任何项目时显示的提示文本。 |
| `AutoScrollToSelectedItem` | `bool` | 指示是否自动滚动到新选中的项目。 |
| `IsDropDownOpen` | `bool` | 指示下拉列表当前是否已打开。 |
| `MaxDropDownHeight` | `double` | 下拉列表的最大高度。这表示列表部分的实际高度，而不是可显示的项目数量。 |
| `ItemsPanel` | `ITemplate<Panel>` | 用于放置项目的容器面板。默认是 `StackPanel`。若要自定义 `ItemsPanel`，请参阅[此页面](/docs/custom-controls/custom-itemspanel)。 |

## 实用说明

- 当你希望控件在加载时就显示一个选项时，请始终设置 `SelectedIndex` 或 `SelectedItem` 的初始值。如果两者都未设置，且你也没有提供 `PlaceholderText`，控件将显示为空白。
- 当你将 `ItemsSource` 绑定到复杂对象集合时，请提供 `ItemTemplate`，以便控件知道如何渲染每个项目。否则，控件会对每个对象调用 `ToString()`。
- 当尚未选择任何项时，可使用 `PlaceholderText` 为用户提供提示（例如“请选择一个类别...”）。
- 如果你需要以编程方式清除选择，可以将 `SelectedIndex` 设为 `-1`，或将 `SelectedItem` 设为 `null`。
- 每当选中项发生变化时，`SelectionChanged` 事件都会触发，这对于在视图模型之外执行副作用逻辑很有用。

## 示例

这个基础示例使用了文本项目，并对下拉列表高度设置了限制。

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui">
  <StackPanel Margin="20">
    <ComboBox SelectedIndex="0" MaxDropDownHeight="100">
      <ComboBoxItem>Text Item 1</ComboBoxItem>
      <ComboBoxItem>Text Item 2</ComboBoxItem>
      <ComboBoxItem>Text Item 3</ComboBoxItem>
      <ComboBoxItem>Text Item 4</ComboBoxItem>
      <ComboBoxItem>Text Item 5</ComboBoxItem>
      <ComboBoxItem>Text Item 6</ComboBoxItem>
      <ComboBoxItem>Text Item 7</ComboBoxItem>
      <ComboBoxItem>Text Item 8</ComboBoxItem>
      <ComboBoxItem>Text Item 9</ComboBoxItem>
    </ComboBox>
  </StackPanel>
</UserControl>
```

</XamlPreview>

此示例为每个项目使用了组合视图：

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui">
  <StackPanel Margin="20">
    <ComboBox SelectedIndex="0">
      <ComboBoxItem>
        <Panel>
          <Ellipse Width="50" Height="50" Fill="Red"/>
          <TextBlock VerticalAlignment="Center"
                     HorizontalAlignment="Center">Red</TextBlock>
        </Panel>
      </ComboBoxItem>
      <ComboBoxItem>
          <Panel>
            <Ellipse Width="50" Height="50" Fill="Orange"/>
            <TextBlock VerticalAlignment="Center"
                       HorizontalAlignment="Center">Amber</TextBlock>
          </Panel>
      </ComboBoxItem>
      <ComboBoxItem>
        <Panel>
          <Ellipse Width="50" Height="50" Fill="Green"/>
          <TextBlock VerticalAlignment="Center"
                     HorizontalAlignment="Center">Green</TextBlock>
        </Panel>
      </ComboBoxItem>
    </ComboBox>
  </StackPanel>
</UserControl>
```

</XamlPreview>

此示例使用数据模板来绑定组合框中的项目。C# code-behind 会加载已安装的字体族名称，并将它们绑定到 `ItemsSource` 属性。

```xml
<StackPanel Margin="20">
  <ComboBox x:Name="fontComboBox" SelectedIndex="0"
            Width="200" MaxDropDownHeight="300"
            ItemsSource="{Binding FontFamilies}"
            SelectedValue="{Binding SelectedFont}">
    <ComboBox.ItemTemplate>
      <DataTemplate>
        <TextBlock Text="{Binding Name}" FontFamily="{Binding}" />
      </DataTemplate>
    </ComboBox.ItemTemplate>
  </ComboBox>
</StackPanel>
```

```csharp title='C#'
using Avalonia.Controls;
using Avalonia.Media;
using Avalonia.Media.Fonts;
using System.Collections.Generic;
using System.Linq;

namespace TmpAvaloniaApp;

public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();
        IFontCollection fontCollection = FontManager.Current.SystemFonts;
        FontFamilies = new List<FontFamily>(fontCollection).OrderBy(x=>x.Name).ToList();
        DataContext = this;
    }

    public FontFamily? SelectedFont { get; set; }

    public List<FontFamily> FontFamilies { get; set; }
}
```

<Image light={ComboBoxDataTemplateScreenshot} alt="ComboBox with data template showing font families" position="center" maxWidth={400} cornerRadius="true"/>

## 绑定到视图模型

绑定 `ItemsSource`、`SelectedItem`，并使用 `ItemTemplate`：

```csharp
public partial class MainViewModel : ObservableObject
{
    public ObservableCollection<string> Categories { get; } = new()
    {
        "Electronics", "Clothing", "Books", "Food"
    };

    [ObservableProperty]
    private string? _selectedCategory;
}
```

```xml
<ComboBox ItemsSource="{Binding Categories}"
          SelectedItem="{Binding SelectedCategory}"
          PlaceholderText="Select a category" />
```

## 可编辑 ComboBox

将 `IsEditable` 设置为 `true`，即可直接在组合框中输入文本。随着输入，控件会在项目中查找匹配项，并相应更新 `SelectedItem`。`Text` 属性则保存当前输入的文本值。

```xml
<ComboBox IsEditable="True"
          Text="{Binding SearchText}"
          ItemsSource="{Binding Countries}"
          SelectedItem="{Binding SelectedCountry}"
          PlaceholderText="Type a country..." />
```

### `TextSearch.TextBinding`

当项目是复杂对象时，可使用 `TextSearch.TextBinding` 来指定可编辑文本应匹配哪个属性：

```xml
<ComboBox IsEditable="True"
          ItemsSource="{Binding People}"
          SelectedItem="{Binding SelectedPerson}"
          TextSearch.TextBinding="{Binding FullName}">
    <ComboBox.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding FullName}" />
        </DataTemplate>
    </ComboBox.ItemTemplate>
</ComboBox>
```

## 占位文本

在未选择任何项目时显示占位提示文本：

```xml
<ComboBox PlaceholderText="Choose an option..."
          ItemsSource="{Binding Options}"
          SelectedItem="{Binding SelectedOption}" />
```

## 另请参阅

- [ListBox](/controls/data-display/collections/listbox)
- [AutoCompleteBox](/controls/input/text-input/autocompletebox)
- [RadioButton](/controls/input/buttons/radiobutton)
- [Data templates](/docs/data-templates/introduction-to-data-templates)
- [ComboBox API reference](/api/avalonia/controls/combobox)
- [`ComboBox.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/ComboBox.cs)
