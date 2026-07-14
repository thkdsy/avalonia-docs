---
id: style-selector-syntax
title: 样式选择器语法
---

本页列出了样式选择器在 XAML 中的写法，以及与之对应、可执行同样选择逻辑的 C# 代码方法。

## 按控件类选择

```xml
<Style Selector="Button">
<Style Selector="local|Button">
```

```csharp title='C#'
new Style(x => x.OfType<Button>());
new Style(x => x.OfType(typeof(Button)));
```

按控件类名选择控件。

上面的第一个示例选择的是 `Avalonia.Controls.Button` 类。如果你需要在类型中包含 XAML 命名空间，请使用 `|` 字符分隔命名空间和类型名。

:::caution
这个选择器不会匹配派生类型。如果你需要匹配派生类，请使用 [`:is` selector](#include-derived-classes)。
:::

:::info
对象的类型是通过查看它的 `StyleKey` 属性来决定的。默认情况下，这个属性会返回当前实例的类型；但如果你的控件继承自 `Button`，而你希望它按 `Button` 的方式参与样式匹配，那么你可以重写类中的 `StyleKeyOverride` 属性，并让它返回 `typeof(Button)`。
:::

## 按名称选择

```xml
<Style Selector="#myButton">
<Style Selector="Button#myButton">
```

```csharp title='C#'
new Style(x => x.Name("myButton"));
new Style(x => x.OfType<Button>().Name("myButton"));
```

通过控件的 `Name` 属性来选择，并在前面加上 `#` 前缀。

## 按样式类选择

```xml
<Style Selector="Button.large">
<Style Selector="Button.large.red">
```

```csharp title='C#'
new Style(x => x.OfType<Button>().Class("large"));
new Style(x => x.OfType<Button>().Class("large").Class("red"));
```

选择带有指定样式类的控件。多个类之间用句点分隔。如果选择器中指定了多个类，那么控件必须同时拥有这些类，才会被匹配到。

## 按伪类选择

```xml
<Style Selector="Button:focus">
<Style Selector="Button:focus:pointerover">
<Style Selector="Button.large:focus">
```

```csharp title='C#'
new Style(x => x.OfType<Button>().Class(":focus"));
new Style(x => x.OfType<Button>().Class(":focus").Class(":pointerover"));
new Style(x => x.OfType<Button>().Class("large").Class(":focus"));
```

根据控件当前的伪类状态进行选择。选择器中的冒号表示伪类名称的起始位置。一个控件可以同时具有多个伪类。

:::info
有关伪类的更多细节，请参阅 [Pseudoclasses](/docs/styling/pseudoclasses)。
:::

## 包含派生类

```xml
<Style Selector=":is(Button)">
<Style Selector=":is(local|Button)">
```

```csharp title='C#'
new Style(x => x.Is<Button>());
new Style(x => x.Is(typeof(Button)));
```

这种写法与按类选择非常相似，但它还会匹配派生类型。

:::info
在匹配过程中，Avalonia 会通过检查控件的 `StyleKey` 属性来确定其类型。
:::

这使你能够编写非常通用的基于类的选择器。由于所有控件都继承自 `Control` 类，因此如果你想只根据样式类 `margin2` 来选择控件，可以这样写：

```xml
<Style Selector=":is(Control).margin2">
<Style Selector=":is(local|Control).margin2">
```

```csharp title='C#'
new Style(x => x.Is<Control>().Class("margin2"));
new Style(x => x.Is(typeof(Control)).Class("margin2"));
```

## 子级运算符

```xml
<Style Selector="StackPanel > Button">
```

```csharp title='C#'
new Style(x => x.OfType<StackPanel>().Child().OfType<Button>());
```

子级选择器通过 `>` 字符把两个选择器连接起来。它只会匹配**逻辑控件树**中的直接子元素。

:::info
有关逻辑控件树的概念，请参阅 [Control trees](/docs/custom-controls/control-trees)。
:::

For example, applying the above selector to this XAML:

```xml
<StackPanel>
   <Button>Save</Button>
   <DockPanel Width="300" Height="300">
       <Button DockPanel.Dock="Top">Top</Button>
       <TextBlock>Some text</TextBlock>
   </DockPanel>
</StackPanel>
```

这个选择器会匹配第一个按钮，但不会匹配第二个按钮。因为第二个按钮并不是 `StackPanel` 的直接子元素，它还被包裹在 `DockPanel` 内部。

## 任意后代运算符

```xml
<Style Selector="StackPanel Button">
```

```csharp title='C#'
new Style(x => x.OfType<StackPanel>().Descendant().OfType<Button>());
```

当两个选择器之间用空格分隔时，它会匹配逻辑树中的任意后代元素。左边表示祖先，右边表示后代。

因此，如果把上面的选择器应用到前面的 XAML 示例中，那么两个按钮都会被选中。

## 按属性匹配

```xml
<Style Selector="Button[IsDefault=true]">
```

```csharp title='C#'
new Style(x => x.OfType<Button>().PropertyEquals(Button.IsDefaultProperty, true));
```

你可以通过属性值进一步细化选择器。属性=value 对写在方括号中。它会匹配所有指定属性值等于给定值的控件。

```xml
<StackPanel Orientation="Horizontal">
   <Button IsDefault="True">Save</Button>
   <Button>Cancel</Button>   
</StackPanel>
```

例如，在上面的 XAML 中，第一个按钮会被选中，而第二个不会。

:::info
注意：当你使用附加属性进行属性匹配时，属性名必须用括号包起来。例如：

```xml
<Style Selector="TextBlock[(Grid.Row)=0]">
```
:::

:::info
当你使用属性匹配时，该属性类型必须支持组件模型中的 `TypeConverter`。更多说明请参阅 [Microsoft TypeConverter documentation](https://learn.microsoft.com/dotnet/api/system.componentmodel.typeconverter)。
:::

## 按模板内部选择

```xml
<Style Selector="Button /template/ ContentPresenter">
```

```csharp title='C#'
new Style(x => x.OfType<Button>().Template().OfType<ContentPresenter>());
```

上面的语法用于选择控件模板内部的元素。在这个例子里，选择器匹配的是位于 `Button` 模板内部的 [`ContentPresenter`](/api/avalonia/controls/presenters/contentpresenter) 控件。

这个选择器比较特殊，因为它可以深入模板内部进行选择，而不是像本页其他选择器一样只作用于逻辑树。

## Not 函数

```xml
<Style Selector="TextBlock:not(.h1)">
```

```csharp title='C#'
new Style(x => x.OfType<TextBlock>().Not(y => y.Class("h1")));
```

这个函数会对括号中的选择结果取反。在上面的例子里，所有**不**带 `h1` 类的文本块都会被匹配到。

## 按列表选择

```xml
<Style Selector="TextBlock, Button">
```

```csharp title='C#'
new Style(x => Selectors.Or(x.OfType<TextBlock>(), x.OfType<Button>()))
```

你可以使用逗号分隔的一组选择器，来匹配满足其中任意一个条件的元素。该样式中的 setter 必须作用于所有这些元素共有的属性。

## 按子元素位置公式选择

```xml
<Style Selector="TextBlock:nth-child(2n+3)">
```

```csharp title='C#'
new Style(x => x.OfType<TextBlock>().NthChild(2, 3));
```

你可以根据元素在同级兄弟中的位置进行匹配，而不依赖父级（容器）控件的具体类型。

选择规则基于样式中的简单公式 `An + B`：其中 **`A`** 控制步长，**`B`** 控制起始偏移。在上面的 `nth-child` 公式中，**`n`** 会依次取 0 及所有从 0 开始的正整数，最终再与子元素的从 1 开始的位置进行比较，以决定是否匹配。

因此，对于上面的选择器：

<table><thead><tr><th width="175">Child = 1</th><th width="184">Child = 2</th><th width="201">Child = 3</th><th>Child = 4</th></tr></thead><tbody><tr><td>n=0, n=1</td><td>n=0, n=1</td><td>n=0, n=1</td><td>n=0, n=1</td></tr><tr><td>3, 5</td><td>3, 5</td><td><strong>3</strong>, 5</td><td>3, 5</td></tr><tr><td>No Match</td><td>No Match</td><td>Match</td><td>No Match</td></tr></tbody></table>

如果公式计算结果小于 1，那么它会被忽略，因为不存在索引小于 1 的子元素。

还有一个对应的选择器，它会使用从分组末尾开始计数的公式：

```xml
<Style Selector="TextBlock:nth-last-child(2n+3)">
```

```csharp title='C#'
new Style(x => x.OfType<TextBlock>().NthLastChild(2, 3));
```

### 单个子元素位置

你可以在 XAML 公式中省略 **A** 和 **n**，只指定一个固定位置。例如，下面的写法只会选择第 3 个子元素：

```xml
<Style Selector="TextBlock:nth-child(3)">
```

```csharp title='C#'
new Style(x => x.OfType<TextBlock>().NthChild(0, 3));
```

### 关键字写法

你也可以使用关键字来替代公式：`odd` 或 `even`。因此下面这些选择器是等价的：

```xml
<Style Selector="TextBlock:nth-child(2n)">
<Style Selector="TextBlock:nth-child(even)">
```

```xml
<Style Selector="TextBlock:nth-child(2n+1)">
<Style Selector="TextBlock:nth-child(odd)">
```

### 其他公式示例

下表列出了一些按子元素位置进行选择的示例：

| 公式示例 | 含义 |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `:nth-child(odd)` | 奇数位置元素：**1**、**3**、**5**，依此类推。 |
| `:nth-child(even)` | 偶数位置元素：**2**、**4**、**6**，依此类推。 |
| `:nth-child(2n+1)` | 奇数位置元素：**1**_(2×0+1)_、**3**_(2×1+1)_、**5**_(2×2+1)_，依此类推。等价于 `:nth-child(odd)`。 |
| `:nth-child(2n)` | 偶数位置元素：**2**_(2×1)_、**4**_(2×2)_、**6**_(2×3)_，依此类推。等价于 `:nth-child(even)`。注意 **0**_(2×0)_ 虽然公式合法，但由于索引从 1 开始，因此不会匹配任何元素。 |
| `:nth-child(7)` | 第 7 个元素。 |
| `:nth-child(n+7)` | 从第 7 个开始的所有元素：**7**_(0+7)_、**8**_(1+7)_、**9**_(2+7)_，依此类推。 |
| `:nth-child(3n+4)` | 从第 4 个开始，每隔 3 个选择一个元素：**4**_(3×0+4)_、**7**_(3×1+4)_、**10**_(3×2+4)_、**13**_(3×3+4)_，依此类推。 |
| `:nth-child(-n+3)` | 前 3 个元素：**3**_(-1×0+3)_、**2**_(-1×1+3)_、**1**_(-1×2+3)_。后续索引都小于 1，因此不会再匹配任何元素。 |

### 在线子元素位置测试器

虽然这是一个 CSS 网站，但它同样适用于 Avalonia 的子元素位置选择器，因为两者规则相同。

:::info
你可以使用这个网站来测试子元素位置选择器： \
[https://css-tricks.com/examples/nth-child-tester/](https://css-tricks.com/examples/nth-child-tester/)
:::

## 嵌套

```xml
<Style Selector="TextBlock">
    <Setter Property="FontSize" Value="24"/>
    
    <!-- Effectively "TextBlock:pointerover" -->
    <Style Selector="^:pointerover">
        <Setter Property="FontWeight" Value="Bold"/>
    </Style>
</Style>
```

```csharp title='C#'
new Style(x => x.OfType<TextBlock>())
{
    Setters = { new Setter(TextBlock.FontSizeProperty, 24d) },
    Children =
    {
        new Style(x => x.Nesting().Class(":pointerover"))
        {
            Setters = { new Setter(TextBlock.FontWeightProperty, FontWeight.Bold) }
        }
    }
};
```

## 另请参阅

- [Style selectors](/docs/styling/style-selectors)
- [Pseudoclasses](/docs/styling/pseudoclasses)
- [Styles](/docs/styling/styles)
