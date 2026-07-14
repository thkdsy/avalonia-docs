---
id: autocompletebox
title: AutoCompleteBox
---

import AutoCompleteBoxScreenshot from '/img/controls/autocompletebox/autocompletebox.gif';

`AutoCompleteBox` 提供一个用于用户输入的文本框，以及一个下拉列表，用于显示与当前输入文本匹配的项目源集合中的候选项。当用户开始输入时，下拉列表会显示出来，并会随着每次输入字符而更新匹配结果。用户可以从下拉列表中进行选择。

文本与项目源中候选项的匹配方式是可配置的。

## 常用属性

你最常使用的通常是这些属性：

<table><thead><tr><th width="233">属性</th><th>说明</th></tr></thead><tbody><tr><td><code>ItemsSource</code></td><td>用于匹配的项目列表。</td></tr><tr><td><code>FilterMode</code></td><td>控制匹配方式的选项。见下表。</td></tr><tr><td><code>AsyncPopulator</code></td><td>一个异步函数，可根据给定的（字符串）条件提供匹配结果列表。</td></tr><tr><td><code>MaxLength</code></td><td>用户最多可输入的字符数。0 表示不限制。</td></tr><tr><td><code>InnerLeftContent</code></td><td>显示在文本区域左侧内部的内容（例如搜索图标）。</td></tr><tr><td><code>InnerRightContent</code></td><td>显示在文本区域右侧内部的内容（例如清除按钮）。</td></tr></tbody></table>

以下是 `FilterMode` 属性的可选值：

<table><thead><tr><th width="350">筛选模式</th><th>说明</th></tr></thead><tbody><tr><td><code>StartsWith</code></td><td>区分区域性、不区分大小写；返回以指定文本开头的项目。</td></tr><tr><td><code>StartsWithCaseSensitive</code></td><td>区分区域性、区分大小写；返回以指定文本开头的项目。</td></tr><tr><td><code>StartsWithOrdinal</code></td><td>按序号比较、不区分大小写；返回以指定文本开头的项目。</td></tr><tr><td><code>StartsWithOrdinalCaseSensitive</code></td><td>按序号比较、区分大小写；返回以指定文本开头的项目。</td></tr><tr><td><code>Contains</code></td><td>区分区域性、不区分大小写；返回包含指定文本的项目。</td></tr><tr><td><code>ContainsCaseSensitive</code></td><td>区分区域性、区分大小写；返回包含指定文本的项目。</td></tr><tr><td><code>ContainsOrdinal</code></td><td>按序号比较、不区分大小写；返回包含指定文本的项目。</td></tr><tr><td><code>ContainsOrdinalCaseSensitive</code></td><td>按序号比较、区分大小写；返回包含指定文本的项目。</td></tr><tr><td><code>Equals</code></td><td>区分区域性、不区分大小写；返回与指定文本完全相等的项目。</td></tr><tr><td><code>EqualsCaseSensitive</code></td><td>区分区域性、区分大小写；返回与指定文本完全相等的项目。</td></tr><tr><td><code>EqualsOrdinal</code></td><td>按序号比较、不区分大小写；返回与指定文本完全相等的项目。</td></tr><tr><td><code>EqualsOrdinalCaseSensitive</code></td><td>按序号比较、区分大小写；返回与指定文本完全相等的项目。</td></tr></tbody></table>

:::info
在 **ordinal** 字符串比较中，每个字符都按照其简单字节值进行比较，而不考虑语言规则。
:::

:::info
**culture-sensitive** 指的是在设计和技术实现中考虑不同文化背景用户的需求。这包括依据语言采用不同的字符串处理和排序规则。例如，英语通常按 A-Z 字母顺序排序，中文可能按拼音或笔画排序，而其他语言也可能有各自不同的排序规则。
:::


## 示例

此示例使用一个固定项目源（数组），并在 C# 代码后置中进行设置。

```xml
<StackPanel Margin="20">
  <TextBlock Margin="0 5">Choose an animal:</TextBlock>
  <AutoCompleteBox x:Name="animals" FilterMode="StartsWith" />
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
            .OrderBy(x=>x);
        }
    }
}
```

<Image light={AutoCompleteBoxScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 将 AutoCompleteBox 与对象一起使用
当你处理的是复杂对象而不是简单字符串时，需要指定应显示哪个属性，以及控件应如何筛选底层数据。下面几节会介绍显示绑定、自定义筛选，以及显示文本格式化。

#### 使用 ValueMemberBinding 筛选对象
`ValueMemberBinding` 告诉控件，应显示对象的哪个属性到文本框中，并将其用于内置筛选。

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:vm="using:YourNamespace.ViewModels"
        x:Class="YourNamespace.MainWindow"
        x:DataType="vm:MainViewModel">

    <StackPanel Margin="20">
        <TextBlock Margin="0 5">Select a product:</TextBlock>

        <AutoCompleteBox ItemsSource="{Binding Products}"
                         SelectedItem="{Binding SelectedProduct}"
                         ValueMemberBinding="{Binding Name}"
                         FilterMode="Contains"
                         MinimumPrefixLength="0" />

        <TextBlock Margin="0 10"
                   Text="{Binding SelectedProduct.Price,
                                  StringFormat='Price: ${0:F2}'}" />
    </StackPanel>
</Window>
```

```csharp title='C#'

using System.Collections.ObjectModel;
using System.ComponentModel;

public class Product : INotifyPropertyChanged
{
    private int id;
    private string name = string.Empty;
    private decimal price;

    public int Id
    {
        get => id;
        set
        {
            if (id != value)
            {
                id = value;
                OnPropertyChanged(nameof(Id));
            }
        }
    }

    public string Name
    {
        get => name;
        set
        {
            if (name != value)
            {
                name = value;
                OnPropertyChanged(nameof(Name));
            }
        }
    }

    public decimal Price
    {
        get => price;
        set
        {
            if (price != value)
            {
                price = value;
                OnPropertyChanged(nameof(Price));
            }
        }
    }

    public event PropertyChangedEventHandler? PropertyChanged;
    private void OnPropertyChanged(string propertyName) =>
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
}

public class MainViewModel : INotifyPropertyChanged
{
    private Product? selectedProduct;

    public ObservableCollection<Product> Products { get; } = new()
    {
        new Product { Id = 1, Name = "Laptop", Price = 999.99m },
        new Product { Id = 2, Name = "Mouse", Price = 29.99m },
        new Product { Id = 3, Name = "Keyboard", Price = 79.99m },
        new Product { Id = 4, Name = "Monitor", Price = 299.99m },
        new Product { Id = 5, Name = "Headphones", Price = 149.99m }
    };

    public Product? SelectedProduct
    {
        get => selectedProduct;
        set
        {
            if (selectedProduct != value)
            {
                selectedProduct = value;
                OnPropertyChanged(nameof(SelectedProduct));
            }
        }
    }

    public event PropertyChangedEventHandler? PropertyChanged;
    private void OnPropertyChanged(string propertyName) =>
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
}

```

#### 使用 ItemFilter 实现自定义筛选
如果你需要跨多个属性进行搜索，例如同时搜索 Name 和 Id，就可以提供一个自定义筛选函数。

```xml
<AutoCompleteBox x:Name="ProductAutoComplete"
                 ItemsSource="{Binding Products}"
                 SelectedItem="{Binding SelectedProduct}"
                 ValueMemberBinding="{Binding Name}"
                 MinimumPrefixLength="1" />
```
```csharp title='C#'

public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        // 自定义筛选：同时搜索 Name 和 Id
        ProductAutoComplete.ItemFilter = (search, item) =>
        {
            if (item is Product product && !string.IsNullOrWhiteSpace(search))
            {
                return product.Name.Contains(search, StringComparison.OrdinalIgnoreCase)
                    || product.Id.ToString().Contains(search);
            }
            return true;
        };
    }
}

```

#### 使用值转换器自定义显示文本
你可以使用值转换器来控制下拉列表中显示的文本。当你希望显示多个字段的组合时，这会很有帮助，例如同时显示产品名称和价格。需要注意的是，这也会影响筛选行为：如果你同时显示名称和价格，控件就允许按其中任一值进行筛选。

如果你只是想更改项目的视觉显示方式，而不影响筛选逻辑，则应改用标准的 `ItemTemplate` 属性。

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:converters="using:YourNamespace.Converters"
        xmlns:vm="clr-namespace:YourNamespace.ViewModels;assembly=YourNamespace"
        x:DataType="vm:MainViewModel">

    <Window.Resources>
        <converters:ProductDisplayConverter x:Key="ProductDisplayConverter" />
    </Window.Resources>

    <StackPanel Margin="20">
        <TextBlock Margin="0 5">Select a product:</TextBlock>

        <AutoCompleteBox ItemsSource="{Binding Products}"
                         SelectedItem="{Binding SelectedProduct}"
                         ValueMemberBinding="{Binding ., Converter={StaticResource ProductDisplayConverter}}"
                         FilterMode="Contains" />
    </StackPanel>
</Window>

```
```csharp title='C#'

using Avalonia.Data.Converters;
using System;
using System.Globalization;

namespace YourNamespace.Converters
{
    public class ProductDisplayConverter : IValueConverter
    {
        public object? Convert(object? value, Type targetType,
                               object? parameter, CultureInfo culture)
        {
            if (value is Product product)
                return $"{product.Name} (${product.Price:F2})";

            return value?.ToString();
        }

        public object? ConvertBack(object? value, Type targetType,
                                   object? parameter, CultureInfo culture)
            => throw new NotSupportedException();
    }
}


```
## 另请参阅

- [AutoCompleteBox API 参考](/api/avalonia/controls/autocompletebox)
- [`AutoCompleteBox.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/AutoCompleteBox/AutoCompleteBox.cs)
