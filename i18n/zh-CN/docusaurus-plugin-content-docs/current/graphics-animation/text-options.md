---
id: text-options
title: 文本选项
description: 用于控制文本字形微调、像素对齐和渲染模式的 TextOptions 附加属性。
doc-type: reference
---

Avalonia 通过 `TextOptions` 附加属性为文本渲染提供细粒度控制。这些设置会影响控件及其后代中所有文本的字形微调、像素对齐和渲染模式。

## 属性

| 附加属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `TextOptions.TextRenderingMode` | `TextRenderingMode` | `Auto` | 控制文本使用抗锯齿、ClearType/子像素渲染，还是无抗锯齿渲染。 |
| `TextOptions.TextHintingMode` | `TextHintingMode` | `Full` | 控制字体字形微调的应用方式。字形微调会调整字形轮廓，使其对齐到像素网格，从而在小字号下获得更清晰的文本。 |
| `TextOptions.BaselinePixelAlignment` | `BaselinePixelAlignment` | `Unspecified` | 控制文本基线是否吸附到整数像素边界。 |

## TextRenderingMode

| 值 | 说明 |
|---|---|
| `Auto` | 由平台自动选择最佳渲染模式。 |
| `Alias` | 文本以无抗锯齿方式渲染。边缘会较锐利但也更锯齿化，适合像素风字体或非常小的文本。 |
| `Antialias` | 文本使用灰度抗锯齿渲染。 |
| `SubpixelAntialias` | 文本使用子像素抗锯齿渲染（例如 Windows 上的 ClearType）。在 LCD 显示器上可获得最锐利的文本效果。 |

```xml
<TextBlock Text="无抗锯齿文本"
           TextOptions.TextRenderingMode="Alias" />

<TextBlock Text="子像素文本"
           TextOptions.TextRenderingMode="SubpixelAntialias" />
```

## TextHintingMode

字体字形微调会调整字形轮廓，使其对齐到像素网格。这会提升小字号文本的可读性，但也可能改变字形形状。减弱或关闭字形微调可以更好地保留原始字型设计，因此更适合大字号文本或动画文本。

| 值 | 说明 |
|---|---|
| `None` | 不进行字形微调。字形保留原始轮廓。最适合大字号文本或动画文本。 |
| `Slight` | 最小程度的字形微调。只调整垂直度量，尽量保留字形的水平形状。 |
| `Normal` | 中等程度的字形微调。 |
| `Full` | 完整字形微调（默认）。最大化像素网格对齐，以获得最清晰的小字号文本。 |

```xml
<!-- 大标题关闭字形微调，以获得更平滑的轮廓 -->
<TextBlock Text="欢迎"
           FontSize="48"
           TextOptions.TextHintingMode="None" />

<!-- 正文文本使用完整字形微调，以获得最佳可读性 -->
<TextBlock Text="请阅读下面的详细信息。"
           FontSize="14"
           TextOptions.TextHintingMode="Full" />
```

## BaselinePixelAlignment

控制文本基线是否吸附到整数像素边界。像素对齐的基线会让静态布局中的文本更清晰；未对齐的基线则允许子像素定位，可避免文本在动画或平滑滚动时出现“跳动”。

| 值 | 说明 |
|---|---|
| `Unspecified` | 由平台决定（通常静态文本会对齐）。 |
| `Aligned` | 基线吸附到最近的像素。最适合静态 UI 文本。 |
| `Unaligned` | 基线使用子像素定位。最适合动画文本或平滑滚动文本。 |

```xml
<!-- 防止文本在 RenderTransform 动画期间发生像素吸附 -->
<TextBlock Text="滑动文本"
           TextOptions.BaselinePixelAlignment="Unaligned">
    <TextBlock.RenderTransform>
        <TranslateTransform />
    </TextBlock.RenderTransform>
</TextBlock>
```

## 应用于容器

与 `RenderOptions` 一样，`TextOptions` 也会被子控件继承。将它设置在容器上，可以影响其中的所有文本：

```xml
<StackPanel TextOptions.TextHintingMode="None"
            TextOptions.BaselinePixelAlignment="Unaligned">
    <TextBlock Text="此面板中的所有文本" />
    <TextBlock Text="都将关闭字形微调并使用子像素基线。" />
</StackPanel>
```

## 在代码中设置

```csharp
TextOptions.SetTextHintingMode(myControl, TextHintingMode.None);
TextOptions.SetBaselinePixelAlignment(myControl, BaselinePixelAlignment.Unaligned);
```

## 另请参阅

- [图像插值](/docs/graphics-animation/image-interpolation)：通过 `RenderOptions` 控制位图渲染质量。
- [自定义渲染](/docs/graphics-animation/custom-rendering)：使用 Avalonia 渲染 API 进行绘制。
