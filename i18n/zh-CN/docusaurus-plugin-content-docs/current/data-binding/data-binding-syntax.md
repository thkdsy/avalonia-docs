---
id: data-binding-syntax
title: 数据绑定语法
description: Avalonia 数据绑定标记语法参考，包括路径、模式、转换器和回退值。
doc-type: reference
---

import DataBindingModeDiagram from '/img/concepts/data-concepts/data-binding-syntax/data-binding-mode.png';

Avalonia 支持在 XAML 和代码中创建数据绑定。在 XAML 中，数据绑定通常通过本文介绍的 `Binding` [`MarkupExtension`](/api/avalonia/markup/xaml/markupextension) 来创建。若要在代码中创建数据绑定，请参阅 [How to bind from code](/docs/data-binding/binding-from-code)。

## 数据绑定 `MarkupExtension`

`Binding` `MarkupExtension` 使用关键字 `Binding`，并结合一组可选参数来定义数据源和其他选项，如下例所示：

```xml
<SomeControl SomeProperty="{Binding Path, Mode=ModeValue, StringFormat=Pattern}" />
```

| 参数 | 说明 |
|-----------------------|-----------------------------------------------------------------------------------|
| `Path` | 要绑定到的源属性名称。 |
| `Mode` | 绑定的数据同步方向。 |
| `Priority` | 属性设置器的优先级。 |
| `Source` | 包含 `Path` 所指定属性的对象。 |
| `ElementName` | 使用一个已命名的 `Control` 作为 `Source`。 |
| `RelativeSource` | 使用视觉树层级中的相对 `Control` 作为 `Source`。 |
| `StringFormat` | 用于把属性值格式化为字符串的模式。 |
| `Converter` | 用于在源值与目标值之间双向转换的 `IValueConverter`。 |
| `ConverterParameter` | 传递给 `Converter` 的参数。 |
| `FallbackValue` | 当绑定无法创建或无法产生值时所设置的值。 |
| `TargetNullValue` | 当源属性值为 `null` 时所设置的值。 |
| [`UpdateSourceTrigger`](/api/avalonia/data/updatesourcetrigger) | 当满足预定义条件时触发源属性更新。 |
| `Delay` | 在源值变化后，延迟一段时间再更新绑定目标。 |

这些参数必须在创建绑定时一并设置。它们是 CLR 属性，不能再通过额外绑定进行设置或更新。

## 数据绑定路径

第一个参数通常就是 `Path`。它表示 `Source` 中某个属性的名称（默认情况下 `Source` 是 `DataContext`），Avalonia 会在创建绑定时查找它。

如果它是第一个参数，就可以省略 `Path=`。下面两种绑定写法是等价的：

```xml
<TextBlock Text="{Binding Name}"/>
<TextBlock Text="{Binding Path=Name}"/>
```

绑定路径既可以是单个属性，也可以是一个子属性链。例如，如果数据源有一个 `Student` 属性，而该属性返回的对象又有一个 `Name` 属性，那么你就可以用下面这样的语法绑定到学生姓名：

```xml
<TextBlock Text="{Binding Student.Name}"/>
```

如果数据源支持索引访问（例如数组或列表），那么你也可以像下面这样在绑定路径中加入索引：

```xml
<TextBlock Text="{Binding Students[0].Name}"/>
```

### 空条件运算符

在绑定路径中可以使用 `?.` 运算符，安全地访问那些可能为 `null` 的属性。如果路径中的任意一段为 `null`，绑定就会产生一个 `null` 值，而不会抛出异常：

```xml
<TextBlock Text="{Binding SelectedStudent?.Address?.City}"/>
```

这与 C# 的空条件运算符作用相同。如果不使用 `?.`，那么 `SelectedStudent` 为 `null` 时就会产生绑定错误。

## 空绑定路径

你也可以在不写 `Path` 的情况下声明数据绑定。此时绑定的就是控件自身的 `DataContext`（也就是定义该绑定的位置）。下面两种语法是等价的：

```xml
<TextBlock Text="{Binding}" />
<TextBlock Text="{Binding .}" />
```

## 数据绑定模式

你可以通过设置 `Mode`，改变数据同步的方向。

<Image light={DataBindingModeDiagram} alt="Diagram showing data binding mode directions between source and target" position="center" maxWidth={400} cornerRadius="true"/>
<br/><br/>

例如：

```xml
<TextBlock Text="{Binding Name, Mode=OneTime}" />
```

可用的绑定模式如下：

| 模式 | 说明 |
|------------------|---------------------------------------------------------------------------------------------------------------------------|
| `OneWay` | 数据源中的变化会传播到绑定目标。 |
| `TwoWay` | 数据源中的变化会传播到绑定目标，反过来也一样。 |
| `OneTime` | 数据源中的值只会向绑定目标传播一次。如果 `DataContext` 变化，绑定会重新求值；但同一个源对象后续的属性变化会被忽略。 |
| `OneWayToSource` | 绑定目标中的变化会传播到数据源，但不会反向传播。 |
| `Default` | 绑定模式由该属性在代码中定义的默认模式决定，详见下文。 |

如果没有显式指定 `Mode`，就会使用 `Default`。对于那些不会因用户交互而改变值的控件属性，默认模式通常是 `OneWay`；而对于会因为用户输入而改变值的控件属性，默认模式通常是 `TwoWay`。

例如，`TextBlock.Text` 的默认模式是 `OneWay`，而 `TextBox.Text` 的默认模式则是 `TwoWay`。

## 数据绑定源

`Source` 指定了 `Path` 所相对的根对象实例。默认情况下，这个根对象就是包含当前绑定的 `Control` 的 `DataContext`。最常见的场景，是使用 `ElementName` 或 `RelativeSource` 参数绑定到另一个控件，或者直接在 `Path` 中使用它们的简写语法（分别是 `#controlName` 和 `$parent[ControlType]`）。

```xml
<TextBox Name="input" />
<TextBlock Text="{Binding Text, ElementName=input}" />
<TextBlock Text="{Binding #input.Text}" />

<TextBlock Text="{Binding Title, 
    RelativeSource={RelativeSource FindAncestor, AncestorType=Window}}" />
<TextBlock Text="{Binding $parent[Window].Title}" />
```

:::info
有关如何绑定到控件的更多说明，请参阅 [How To Bind to a Control](/docs/data-binding/binding-to-controls)
:::

## 转换绑定值

绑定提供了多种方式，用于把数据绑定提供的值转换或替换为更适合目标属性的类型或内容。

### 字符串格式化

你可以通过 `StringFormat` 参数为 `OneWay` 绑定指定格式化模式，把绑定源属性格式化为文本。其内部使用的是 `string.Format`。

格式字符串中的占位符索引从 0 开始，并且必须写在花括号中。如果花括号位于模式开头，那么即使整个模式外面套了单引号，也依然需要进行转义。转义方式可以是在模式最前面加一对空花括号，或者对每个花括号使用反斜杠转义。

```xml
<TextBlock Text="{Binding FloatProperty, StringFormat={}{0:0.0}}" />
```

此外，你也可以用反斜杠来转义格式模式中的花括号。例如：

```xml
<TextBlock Text="{Binding FloatProperty, StringFormat=\{0:0.0\}}" />
```

不过，如果你的模式不是以 `{0` 这种形式开头，就不需要转义。另外，如果模式中包含空格，那么你必须用单引号把它包起来。例如：

```xml
<TextBlock Text="{Binding Animals.Count, StringFormat='I have {0} animals.'}" />
```

这也意味着：如果你的模式一开始就是绑定值本身，那么就确实需要转义。例如：

```xml
<TextBlock Text="{Binding Animals.Count, 
    StringFormat='{}{0} animals live in the farm.'}" />
```

### 带多个参数的字符串格式化

你可以使用 `MultiBinding` 来格式化需要多个绑定参数的字符串。下面的示例把多个数值输入格式化为一个用于显示的字符串。

```xml
<StackPanel Spacing="8">
  <NumericUpDown x:Name="red" Minimum="0" Maximum="255" Value="0" FormatString="{}{0:0.}" Foreground="Red" />
  <NumericUpDown x:Name="green" Minimum="0" Maximum="255" Value="0" FormatString="{}{0:0.}" Foreground="Green" />
  <NumericUpDown x:Name="blue" Minimum="0" Maximum="255" Value="0" FormatString="{}{0:0.}" Foreground="Blue" />

  <TextBlock>
    <TextBlock.Text>
      <MultiBinding StringFormat="(r: {0:0.}, g: {1:0.}, b: {2:0.})">
        <Binding Path="Value" ElementName="red" />
        <Binding Path="Value" ElementName="green" />
        <Binding Path="Value" ElementName="blue" />
      </MultiBinding>
    </TextBlock.Text>
  </TextBlock>
</StackPanel>
```

`FormatString` 是 `NumericUpDown` 内部用来控制值显示方式的属性。这里由于 RGB 颜色值是整数，因此不应显示小数部分，所以使用了 .NET 能识别的自定义数字格式说明符 `0.`。

如果输入值分别是 `red = 100`、`green = 80` 和 `blue = 255`，那么最终显示的文本将是 `(r: 100, g: 80, b: 255)`。

:::tip
另一种做法是使用 `Run` 元素组成的 `InlineCollection`，并让每个 `Run` 自己带一个单参数绑定。这样可以分别对每一段文本做视觉定制。
:::

### 内置转换

Avalonia 提供了一系列内置的数据绑定转换器，包括：

* 空值测试转换器
* 布尔运算转换器

:::info
有关 Avalonia 内置数据绑定转换器的完整列表，请参阅 [built-in data binding converters reference](/docs/data-binding/built-in-data-binding-converters)。
:::

### 自定义转换

如果内置转换器无法满足你的需求，那么你可以通过实现 `IValueConverter` 来创建自定义转换器。

:::info
有关如何创建自定义转换器的指导，请参阅 [How to create a custom data binding converter](/docs/data-binding/how-to-create-a-custom-data-binding-converter)。
:::

### FallbackValue

当属性绑定无法建立，或者转换器返回 `AvaloniaProperty.UnsetValue` 时，就会使用 `FallbackValue`。

一个常见场景是：子属性绑定中的父属性为 `null`。在下面的示例中，如果 `Student` 为 `null`，就会使用 `FallbackValue`：

```xml
<TextBlock Text="{Binding Student.Name, FallbackValue=Cannot find name}"/>
```

:::tip
`ReflectionBinding` 可以绑定任意类型，而不受编译期安全检查限制。当绑定无法建立时，`FallbackValue` 可以作为一个很有用的替代值。
:::

### `TargetNullValue`

当某个属性的绑定成功建立、但属性值本身是 `null` 时，你可以使用 `TargetNullValue` 提供一个特定值。

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui"
            xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
            Margin="10">
    <NumericUpDown x:Name="number" Value="200" />
    <TextBlock Text="{Binding #number.Value, TargetNullValue=Value is null}" />
</StackPanel>
```

</XamlPreview>

## `UpdateSourceTrigger`

像 `TextBox` 这样的控件，默认会在每次按键输入后把 `Text` 绑定同步回源属性。在某些场景下，这可能会触发耗时任务或不希望过早发生的验证。`UpdateSourceTrigger` 可以让你指定同步发生的时机。

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui">
    <TextBlock Width="200">PropertyChanged</TextBlock>
    <TextBox Text="{Binding #propertyChanged.Text}"
             Width="200" />
    <TextBlock Name="propertyChanged"
               Width="200" />

    <TextBlock Width="200">LostFocus</TextBlock>
    <TextBox Text="{Binding #lostFocus.Text, UpdateSourceTrigger=LostFocus}"
             Width="200" />
    <TextBlock Name="lostFocus"
               Width="200" />
</StackPanel>
```

</XamlPreview>

| UpdateSourceTrigger | Description                                                                                      |
|---------------------|--------------------------------------------------------------------------------------------------|
| `Default`           | This currently defaults to `PropertyChanged`.                                                    |
| `PropertyChanged`   | Updates the binding source immediately whenever the binding target property changes.             |
| `LostFocus`         | Updates the binding source whenever the binding target element loses focus.                      |
| `Explicit`          | Updates the binding source only when you call the `BindingExpressionBase.UpdateSource()` method. |

## `Delay`

_Available since Avalonia 11.3_

The `Delay` parameter specifies a time (in milliseconds) to wait before the binding target is updated after the source value changes. Each time the source value changes, the delay timer resets. The target is only updated once the specified time has elapsed since the last change. This is commonly known as "debouncing."

This is particularly useful for search-as-you-type scenarios, where you want to avoid triggering expensive operations (such as filtering or querying a service) on every keystroke.

### XAML usage

```xml
<TextBox Text="{Binding SearchText, Delay=300}" />
```

In this example, the `SearchText` property on your view model is only updated 300 milliseconds after the user stops typing.

### Code usage

When creating bindings in code, set the `Delay` property to a `TimeSpan`:

```csharp
var binding = new Binding("SearchText")
{
    Delay = TimeSpan.FromMilliseconds(300)
};
myTextBox.Bind(TextBox.TextProperty, binding);
```

### Practical example

The following example shows a search TextBox that waits 300ms after the user stops typing before updating the bound property. This prevents a search operation from running on every keystroke.

```xml
<StackPanel Spacing="8">
    <TextBox PlaceholderText="Search..."
             Text="{Binding SearchText, Delay=300}" />
    <TextBlock Text="{Binding SearchResults}" />
</StackPanel>
```

```csharp
public class SearchViewModel : ObservableObject
{
    private string _searchText = string.Empty;
    private string _searchResults = string.Empty;

    public string SearchText
    {
        get => _searchText;
        set
        {
            if (SetProperty(ref _searchText, value))
            {
                // This will only be called 300ms after the user stops typing
                PerformSearch(value);
            }
        }
    }

    public string SearchResults
    {
        get => _searchResults;
        set => SetProperty(ref _searchResults, value);
    }

    private void PerformSearch(string query)
    {
        SearchResults = string.IsNullOrEmpty(query)
            ? string.Empty
            : $"Searching for: {query}";
    }
}
```

## See also

- [Data context](/docs/data-binding/data-context): Where the data binder gets the data object from.
- [Compiled bindings](/docs/data-binding/compiled-bindings): Compile-time binding validation.
- [How to create a custom data binding converter](/docs/data-binding/how-to-create-a-custom-data-binding-converter): Custom value converters.
