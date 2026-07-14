---
id: typography
title: 排版
description: 文本的字号、字重、字形、字宽、字间距、行高、文本装饰和对齐属性。
doc-type: reference
---

Avalonia 提供了一组属性，用于控制应用中文本的显示方式。这些属性定义在 [`TextElement`](/api/avalonia/controls/documents/textelement) 上，属于可继承的附加属性，因此你可以把它们设置在任意控件上，并通过 [property value inheritance](/docs/properties/property-value-inheritance) 影响其视觉树中的所有文本。

## TextElement 附加属性

以下属性定义在 `TextElement` 上，并会被后代控件继承。你既可以直接把它们设置到 `TextBlock` 这类文本控件上，也可以设置到容器控件上，以影响其中的所有文本。

| 附加属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `TextElement.FontFamily` | [`FontFamily`](/api/avalonia/media/fontfamily) | 平台默认值 | 用于渲染文本的字体家族。 |
| `TextElement.FontSize` | `double` | `12` | 以设备无关像素表示的文本大小。 |
| `TextElement.FontWeight` | [`FontWeight`](/api/avalonia/media/fontweight) | `Normal` | 字形笔画的粗细。 |
| `TextElement.FontStyle` | [`FontStyle`](/api/avalonia/media/fontstyle) | `Normal` | 文本是正常、斜体还是倾斜显示。 |
| `TextElement.FontStretch` | [`FontStretch`](/api/avalonia/media/fontstretch) | `Normal` | 字符相对于正常宽高比的宽度变化。 |
| `TextElement.FontFeatures` | `FontFeatureCollection` | `null` | 要启用或禁用的 OpenType 字体特性。 |
| `TextElement.Foreground` | `IBrush` | 继承 | 用于绘制文本的画刷。 |
| `TextElement.LetterSpacing` | `double` | `0` | 以设备无关像素表示的额外字间距。 |

当你直接在 `TextBlock` 上设置 `FontSize` 这类属性时，本质上等同于在该控件上设置 `TextElement.FontSize`。

```xml
<!-- Set font properties on a container to apply to all child text -->
<StackPanel TextElement.FontSize="16"
            TextElement.FontWeight="SemiBold"
            TextElement.LetterSpacing="0.5">
    <TextBlock Text="This inherits all three properties." />
    <TextBlock Text="So does this." />
    <TextBlock FontSize="24" Text="This overrides FontSize but inherits the rest." />
</StackPanel>
```

## 字号

`FontSize` 用于指定文本的高度，单位是设备无关像素。你可以把它设置在单个控件上，也可以设置到容器上以影响所有后代文本。

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui" Margin="20" Spacing="4">
    <TextBlock FontSize="12" Text="12px (default size)" />
    <TextBlock FontSize="16" Text="16px" />
    <TextBlock FontSize="24" Text="24px" />
    <TextBlock FontSize="36" Text="36px" />
</StackPanel>
```

</XamlPreview>

## 字重

`FontWeight` 控制字符笔画的粗细。你可以使用命名值，也可以直接使用 1 到 999 的数值。

| Named value | Numeric value | Aliases |
|---|---|---|
| `Thin` | 100 | |
| `ExtraLight` | 200 | `UltraLight` |
| `Light` | 300 | |
| `SemiLight` | 350 | |
| `Normal` | 400 | `Regular` |
| `Medium` | 500 | |
| `SemiBold` | 600 | `DemiBold` |
| `Bold` | 700 | |
| `ExtraBold` | 800 | `UltraBold` |
| `Black` | 900 | `Heavy` |
| `ExtraBlack` | 950 | `UltraBlack` |

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui" Margin="20" Spacing="4">
    <TextBlock FontWeight="Light" Text="Light (300)" />
    <TextBlock FontWeight="Normal" Text="Normal (400)" />
    <TextBlock FontWeight="Medium" Text="Medium (500)" />
    <TextBlock FontWeight="SemiBold" Text="SemiBold (600)" />
    <TextBlock FontWeight="Bold" Text="Bold (700)" />
    <TextBlock FontWeight="ExtraBold" Text="ExtraBold (800)" />
    <TextBlock FontWeight="Black" Text="Black (900)" />
</StackPanel>
```

</XamlPreview>

你也可以在 XAML 中直接使用数值，或在代码中将整数转换为对应类型：

```xml
<TextBlock FontWeight="550" Text="Custom weight 550" />
```

```csharp
myTextBlock.FontWeight = (FontWeight)550;
```

:::note
可用字重取决于具体字体。如果请求的字重不存在，Avalonia 会选择最接近的匹配项。有些字体只提供少数几种字重（例如 Normal 和 Bold），而有些字体则提供完整范围。
:::

## 字形

`FontStyle` 控制文本是以正常、斜体还是倾斜方式渲染。

| Value | Description |
|---|---|
| `Normal` | Upright text (default). |
| `Italic` | Uses the italic variant of the font, designed with modified letterforms. |
| `Oblique` | Slants the text algorithmically. Used when the font does not include a true italic variant. |

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui" Margin="20" Spacing="4">
    <TextBlock FontStyle="Normal" Text="Normal style" />
    <TextBlock FontStyle="Italic" Text="Italic style" />
    <TextBlock FontStyle="Oblique" Text="Oblique style" />
</StackPanel>
```

</XamlPreview>

## 字宽

`FontStretch` 控制字符相对于其正常宽高比的宽度变化。这个属性要求字体本身提供压缩版或扩展版字形。

| Value | Description |
|---|---|
| `UltraCondensed` | Narrowest character width. |
| `ExtraCondensed` | Narrower than `Condensed`. |
| `Condensed` | Narrower than `SemiCondensed`. |
| `SemiCondensed` | Slightly narrower than `Normal`. |
| `Normal` | Default character width. |
| `SemiExpanded` | Slightly wider than `Normal`. |
| `Expanded` | Wider than `SemiExpanded`. |
| `ExtraExpanded` | Wider than `Expanded`. |
| `UltraExpanded` | Widest character width. |

```xml
<TextBlock FontStretch="Condensed" Text="Condensed text" />
<TextBlock FontStretch="Normal" Text="Normal text" />
<TextBlock FontStretch="Expanded" Text="Expanded text" />
```

:::note
大多数字体只包含 `Normal` 宽度的字形。除非字体本身包含针对所请求字宽设计的字形，否则 `FontStretch` 不会产生可见效果。
:::

## 字间距

`LetterSpacing` 用于在字符之间增加额外间距，单位为设备无关像素。正值会增大间距，负值会减小间距。

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui" Margin="20" Spacing="4">
    <TextBlock LetterSpacing="-1" Text="Tighter spacing (-1px)" />
    <TextBlock LetterSpacing="0" Text="Default spacing (0px)" />
    <TextBlock LetterSpacing="2" Text="Wider spacing (2px)" />
    <TextBlock LetterSpacing="5" Text="Wide spacing (5px)" />
</StackPanel>
```

</XamlPreview>

由于 `LetterSpacing` 是定义在 `TextElement` 上的可继承附加属性，因此你可以将它设置在容器上，从而影响其中的所有文本：

```xml
<StackPanel TextElement.LetterSpacing="1.5">
    <TextBlock Text="All text in this panel" />
    <TextBlock Text="has 1.5px extra letter spacing." />
</StackPanel>
```

## 行高与行间距

`LineHeight` 和 `LineSpacing` 用于控制 `TextBlock` 中文本行之间的垂直距离。

| Property | Type | Default | Description |
|---|---|---|---|
| `LineHeight` | `double` | `NaN` | The total height of each line. When set to `NaN`, the font metrics determine line height automatically. |
| `LineSpacing` | `double` | `0` | Extra distance added between lines, in device-independent pixels. Added on top of the font's natural line height. |

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui" Margin="20" Spacing="16" Width="300">
    <TextBlock TextWrapping="Wrap"
               Text="Default line height: this text uses the line height determined by the font metrics. No manual adjustment." />

    <TextBlock TextWrapping="Wrap"
               LineHeight="32"
               Text="LineHeight=32: this text has a fixed line height of 32 pixels, creating consistent vertical spacing." />

    <TextBlock TextWrapping="Wrap"
               LineSpacing="8"
               Text="LineSpacing=8: this text adds 8 extra pixels between each line, on top of the font's natural height." />
</StackPanel>
```

</XamlPreview>

当你需要精确控制每一行的尺寸时，请使用 `LineHeight`。如果你只是想在不覆盖字体自然度量的前提下增加行与行之间的留白，则使用 `LineSpacing`。

## 文本对齐

`TextAlignment` 用于控制文本在容器中的水平位置。

| Value | Description |
|---|---|
| `Left` | Text aligns to the left edge. |
| `Center` | Text is centered horizontally. |
| `Right` | Text aligns to the right edge. |
| `Start` | Text aligns to the start edge, respecting `FlowDirection`. Equivalent to `Left` in left-to-right layouts. |
| `End` | Text aligns to the end edge, respecting `FlowDirection`. Equivalent to `Right` in left-to-right layouts. |
| `Justify` | Text is stretched so that each line (except the last) fills the full width. Requires `TextWrapping` to be enabled. |
| `DetectFromContent` | Alignment is inferred from the text content's Unicode directionality. |

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui" Margin="20" Width="300" Spacing="8">
    <TextBlock TextAlignment="Left" Text="Left-aligned text" />
    <TextBlock TextAlignment="Center" Text="Center-aligned text" />
    <TextBlock TextAlignment="Right" Text="Right-aligned text" />
    <TextBlock TextAlignment="Justify" TextWrapping="Wrap"
               Text="Justified text stretches words evenly across the full width when wrapping is enabled." />
</StackPanel>
```

</XamlPreview>

:::info
如果你的应用同时支持从左到右和从右到左布局，建议使用 `Start` 和 `End` 替代 `Left` 与 `Right`。这些值会根据当前的 `FlowDirection` 自动适配。
:::

## 文本装饰

文本装饰会在文本上方、下方或中间绘制线条。Avalonia 通过 [`TextDecorations`](/api/avalonia/media/textdecorations) 类提供了四种预设，也支持通过 [`TextDecoration`](/api/avalonia/media/textdecoration) 类进行完全自定义。

### 预设装饰

| Value | Description |
|---|---|
| `Underline` | A line below the text baseline. |
| `Strikethrough` | A line through the middle of the text. |
| `Overline` | A line above the text. |
| `Baseline` | A line at the text baseline. |

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui" Margin="20" Spacing="4">
    <TextBlock TextDecorations="Underline" Text="Underlined text" />
    <TextBlock TextDecorations="Strikethrough" Text="Struck-through text" />
    <TextBlock TextDecorations="Overline" Text="Overlined text" />
    <TextBlock TextDecorations="Baseline" Text="Baseline decoration" />
    <TextBlock TextDecorations="Underline Strikethrough" Text="Multiple decorations" />
</StackPanel>
```

</XamlPreview>

你也可以将装饰应用到 `TextBlock` 中的单个 `Run` 元素：

```xml
<TextBlock>
    <Run Text="Normal text, " />
    <Run TextDecorations="Underline" Text="underlined text, " />
    <Run TextDecorations="Strikethrough" Text="and struck-through text." />
</TextBlock>
```

### 自定义装饰

如果需要控制颜色、粗细、偏移量和虚线样式，可以直接定义 `TextDecoration`。

| Property | Type | Description |
|---|---|---|
| `Location` | [`TextDecorationLocation`](/api/avalonia/media/textdecorationlocation) | Where the line is drawn: `Underline`, `Strikethrough`, `Overline`, or `Baseline`. |
| `Stroke` | `IBrush` | The brush used to paint the decoration line. |
| `StrokeThickness` | `double` | The thickness of the decoration line. |
| `StrokeThicknessUnit` | [`TextDecorationUnit`](/api/avalonia/media/textdecorationunit) | The unit for thickness: `FontRecommended` (default), `FontRenderingEmSize`, or `Pixel`. |
| `StrokeOffset` | `double` | Vertical offset of the line from its default position. |
| `StrokeOffsetUnit` | `TextDecorationUnit` | The unit for offset. |
| `StrokeDashArray` | `AvaloniaList<double>` | A dash pattern for the decoration line. |
| `StrokeLineCap` | `PenLineCap` | The shape at the ends of dashes: `Flat`, `Round`, or `Square`. |

```xml
<TextBlock Text="Custom red dashed underline">
    <TextBlock.TextDecorations>
        <TextDecorationCollection>
            <TextDecoration Location="Underline"
                           Stroke="Red"
                           StrokeThickness="2"
                           StrokeDashArray="2,2" />
        </TextDecorationCollection>
    </TextBlock.TextDecorations>
</TextBlock>
```

## OpenType 字体特性

`FontFeatures` 属性用于启用或禁用 OpenType 特性，例如连字、等宽数字和小型大写字母。特性通过逗号分隔的标签来指定，语法遵循 HarfBuzz 约定。

```xml
<TextBlock Text="0123456789" FontFeatures="+tnum" />
<TextBlock Text="fi fl ffi" FontFeatures="-liga" />
<TextBlock Text="Small Caps" FontFeatures="+smcp" />
```

有关常见标签和用法示例的完整列表，请参阅 [Custom fonts: OpenType font features](/docs/styling/custom-fonts#opentype-font-features)。

## 使用样式类创建字号体系

Avalonia 不像 HTML 那样内置 `<h1>` 到 `<h6>` 这类标题样式。你可以使用 [style classes](/docs/styling/style-classes) 创建自己的字号体系，从而完全掌控符合应用设计风格的字号、字重和间距。

将这些样式定义在 `App.axaml`（或任意共享资源文件）中，这样它们就能在整个应用中使用：

```xml title="App.axaml"
<Application.Styles>
    <FluentTheme />

    <Style Selector="TextBlock.h1">
        <Setter Property="FontSize" Value="32" />
        <Setter Property="FontWeight" Value="Bold" />
        <Setter Property="LineHeight" Value="40" />
    </Style>
    <Style Selector="TextBlock.h2">
        <Setter Property="FontSize" Value="24" />
        <Setter Property="FontWeight" Value="SemiBold" />
        <Setter Property="LineHeight" Value="32" />
    </Style>
    <Style Selector="TextBlock.h3">
        <Setter Property="FontSize" Value="20" />
        <Setter Property="FontWeight" Value="SemiBold" />
        <Setter Property="LineHeight" Value="28" />
    </Style>
    <Style Selector="TextBlock.subtitle">
        <Setter Property="FontSize" Value="16" />
        <Setter Property="FontWeight" Value="Medium" />
        <Setter Property="Foreground" Value="{DynamicResource TextFillColorSecondaryBrush}" />
    </Style>
    <Style Selector="TextBlock.body">
        <Setter Property="FontSize" Value="14" />
        <Setter Property="LineHeight" Value="20" />
    </Style>
    <Style Selector="TextBlock.caption">
        <Setter Property="FontSize" Value="12" />
        <Setter Property="Foreground" Value="{DynamicResource TextFillColorTertiaryBrush}" />
    </Style>
</Application.Styles>
```

然后通过 `Classes` 属性应用这些样式：

```xml
<StackPanel Spacing="8">
    <TextBlock Classes="h1" Text="Page title" />
    <TextBlock Classes="subtitle" Text="A short description of the page" />
    <TextBlock Classes="h2" Text="Section heading" />
    <TextBlock Classes="body" TextWrapping="Wrap"
               Text="Body text for the main content of the section." />
    <TextBlock Classes="caption" Text="Last updated March 2026" />
</StackPanel>
```

如果某个特定实例需要单独调整，你也可以将样式类与局部覆盖搭配使用：

```xml
<TextBlock Classes="h1" Foreground="DodgerBlue" Text="Colored heading" />
```

这种方式非常适合配合 [sharing styles](/docs/styling/sharing-styles) 在整个应用中复用。只需定义一次字号体系，就可以在各处保持一致使用。

## 在代码中设置排版

所有 `TextElement` 附加属性都提供了静态 `Get` 和 `Set` 方法，可在代码后置逻辑中使用：

```csharp
TextElement.SetFontSize(myPanel, 18);
TextElement.SetFontWeight(myPanel, FontWeight.Bold);
TextElement.SetLetterSpacing(myPanel, 1.5);
TextElement.SetFontStyle(myPanel, FontStyle.Italic);
```

你也可以直接在文本控件上设置这些属性：

```csharp
myTextBlock.FontSize = 24;
myTextBlock.FontWeight = FontWeight.SemiBold;
myTextBlock.LineHeight = 32;
myTextBlock.TextDecorations = TextDecorations.Underline;
```

## 另请参阅

- [Custom fonts](/docs/styling/custom-fonts): Embedding and loading custom font files.
- [Text options](/docs/graphics-animation/text-options): Controlling text rendering, hinting, and baseline alignment.
- [TextBlock](/controls/data-display/text-display/textblock): The primary control for displaying formatted text.
- [TextTrimming](/controls/data-display/text-display/texttrimming): How text is truncated when it overflows.
- [Property value inheritance](/docs/properties/property-value-inheritance): How font properties propagate through the visual tree.
- [`TextElement` API reference](/api/avalonia/controls/documents/textelement)
- [Style classes](/docs/styling/style-classes): Applying named style classes to controls.
- [`FontWeight` API reference](/api/avalonia/media/fontweight)
