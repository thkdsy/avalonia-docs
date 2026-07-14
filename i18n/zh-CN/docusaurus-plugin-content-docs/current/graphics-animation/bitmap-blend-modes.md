---
id: bitmap-blend-modes
title: 位图混合模式
description: 在 Avalonia 中通过位图混合模式控制渲染时像素如何组合。
doc-type: reference
---

import BlendModeCat from '/img/reference/animations-and-graphics/bitmap-blend-modes/Cat.jpg';
import BlendModeOverlayColor from '/img/reference/animations-and-graphics/bitmap-blend-modes/Overlay-Color.png';

import BlendModeOverlay from '/img/reference/animations-and-graphics/bitmap-blend-modes/Overlay.png';
import BlendModePlus from '/img/reference/animations-and-graphics/bitmap-blend-modes/Plus.png';
import BlendModeSaturation from '/img/reference/animations-and-graphics/bitmap-blend-modes/Saturation.png';
import BlendModeScreen from '/img/reference/animations-and-graphics/bitmap-blend-modes/Screen.png';
import BlendModeSoftLight from '/img/reference/animations-and-graphics/bitmap-blend-modes/SoftLight.png';
import BlendModeColor from '/img/reference/animations-and-graphics/bitmap-blend-modes/Color.png';
import BlendModeColorBurn from '/img/reference/animations-and-graphics/bitmap-blend-modes/ColorBurn.png';
import BlendModeColorDodge from '/img/reference/animations-and-graphics/bitmap-blend-modes/ColorDodge.png';
import BlendModeDarken from '/img/reference/animations-and-graphics/bitmap-blend-modes/Darken.png';
import BlendModeDifference from '/img/reference/animations-and-graphics/bitmap-blend-modes/Difference.png';
import BlendModeExclusion from '/img/reference/animations-and-graphics/bitmap-blend-modes/Exclusion.png';
import BlendModeHardLight from '/img/reference/animations-and-graphics/bitmap-blend-modes/HardLight.png';
import BlendModeHue from '/img/reference/animations-and-graphics/bitmap-blend-modes/Hue.png';
import BlendModeLighten from '/img/reference/animations-and-graphics/bitmap-blend-modes/Lighten.png';
import BlendModeLuminosity from '/img/reference/animations-and-graphics/bitmap-blend-modes/Luminosity.png';
import BlendModeMultiply from '/img/reference/animations-and-graphics/bitmap-blend-modes/Multiply.png';
import BlendModeNothing from '/img/reference/animations-and-graphics/bitmap-blend-modes/Nothing.png';

import BlendModeA from '/img/reference/animations-and-graphics/bitmap-blend-modes/A.png';
import BlendModeB from '/img/reference/animations-and-graphics/bitmap-blend-modes/B.png';

import BlendModeDestination from '/img/reference/animations-and-graphics/bitmap-blend-modes/Destination.png';
import BlendModeDestinationAtop from '/img/reference/animations-and-graphics/bitmap-blend-modes/DestinationAtop.png';
import BlendModeDestinationIn from '/img/reference/animations-and-graphics/bitmap-blend-modes/DestinationIn.png';
import BlendModeDestinationOut from '/img/reference/animations-and-graphics/bitmap-blend-modes/DestinationOut.png';
import BlendModeDestinationOver from '/img/reference/animations-and-graphics/bitmap-blend-modes/DestinationOver.png';
import BlendModeSource from '/img/reference/animations-and-graphics/bitmap-blend-modes/Source.png';
import BlendModeSourceAtop from '/img/reference/animations-and-graphics/bitmap-blend-modes/SourceAtop.png';
import BlendModeSourceIn from '/img/reference/animations-and-graphics/bitmap-blend-modes/SourceIn.png';
import BlendModeSourceOut from '/img/reference/animations-and-graphics/bitmap-blend-modes/SourceOut.png';
import BlendModeSourceOver from '/img/reference/animations-and-graphics/bitmap-blend-modes/SourceOver.png';
import BlendModeXor from '/img/reference/animations-and-graphics/bitmap-blend-modes/Xor.png';

在屏幕上渲染位图图形时，Avalonia 支持指定渲染所使用的混合模式。混合模式会改变“新像素（source）覆盖到已有像素（destination）上”时的计算方式。

目前，Avalonia 的 Composite 模式和 Pixel Blend 模式都统一放在同一个枚举中，名称为 `BitmapBlendingMode`。

Composite 模式主要描述新像素如何依据 alpha 通道与当前屏幕像素交互。它可用于实现例如“挖空效果”、排除区域或遮罩等效果。

而 Pixel Blend 模式则用于指定新颜色如何与当前颜色交互。这类模式可用于特殊效果、调整色相，或实现更复杂的图像合成。

如果你想了解混合模式的工作原理以及背后的数学计算，可以参阅 [Wikipedia page](https://en.wikipedia.org/wiki/Blend_modes)。

:::info
混合模式的支持情况取决于渲染后端。Skia 渲染器支持下面列出的所有混合模式。
:::

## 默认行为

默认混合模式是 `SourceOver`，也就是根据 alpha 通道，用新值覆盖像素值。这也是大多数应用叠加两张图像时的标准方式。

## 如何使用

在 XAML 中，你可以为 `Image` 控件指定渲染时所使用的混合模式。下面的示例会在一张很可爱的猫咪图片上叠加一个颜色覆盖层：

```xml
<Panel>
    <Image Source="./Cat.jpg"/>
    <Image Source="./Overlay-Color.png" BlendMode="Multiply"/>
</Panel>
```

如果你正在创建自定义用户控件，并希望在代码中使用其中某种模式来渲染位图，那么可以通过设置控件绘制上下文的 `BitmapBlendingMode` 实现：

``` csharp
// 在 “Render” 方法内部，像这样绘制位图：

using (context.PushRenderOptions(RenderOptions with { BitmapBlendingMode = BitmapBlendingMode.Multiply }))
{
    context.DrawImage(source, sourceRect, destRect);
}
```

## 位图混合模式图示

Avalonia 支持以下可用于渲染的位图混合模式：

### 像素混合模式

像素混合模式只影响颜色，不考虑 alpha 通道。

下面是示例中使用的图像：

| 猫咪底图（destination） | 色轮覆盖图（source） |
|:---:|:---:|
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Cat.jpg" alt="Cat photo used as destination image" width="180"/> | <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Overlay-Color.png" alt="Color wheel overlay used as source image" width="180"/> |

下面是 Avalonia 当前支持的全部取值。

| 预览 | 枚举值 | 说明 |
|---|---|---|
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Nothing.png" alt="Preview of Unspecified blend mode" width="180"/> | `Unspecified` | 或 `SourceOver`，默认行为。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Plus.png" alt="Preview of Plus blend mode" width="180"/> | `Plus` | 显示源图像与目标图像之和。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Screen.png" alt="Preview of Screen blend mode" width="180"/> | `Screen` | 将目标颜色和源颜色分别取补色后相乘，再对结果取补色。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Overlay.png" alt="Preview of Overlay blend mode" width="180"/> | `Overlay` | 根据目标颜色值决定执行乘法还是滤色。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Darken.png" alt="Preview of Darken blend mode" width="180"/> | `Darken` | 选择目标颜色与源颜色中较暗的那个。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/HardLight.png" alt="Preview of Lighten blend mode" width="180"/> | `Lighten` | 选择目标颜色与源颜色中较亮的那个。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/ColorDodge.png" alt="Preview of ColorDodge blend mode" width="180"/> | `ColorDodge` | 提亮目标颜色，以体现源颜色的效果。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/ColorBurn.png" alt="Preview of ColorBurn blend mode" width="180"/> | `ColorBurn` | 加深目标颜色，以体现源颜色的效果。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/HardLight.png" alt="Preview of HardLight blend mode" width="180"/> | `HardLight` | 根据源颜色值决定加深还是提亮颜色。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/SoftLight.png" alt="Preview of SoftLight blend mode" width="180"/> | `SoftLight` | 产生柔和光照效果，比 `HardLight` 更柔和。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Difference.png" alt="Preview of Difference blend mode" width="180"/> | `Difference` | 产生类似 Difference 模式的效果，但对比度更低。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Exclusion.png" alt="Preview of Exclusion blend mode" width="180"/> | `Exclusion` | 产生与 Difference 类似但更柔和的效果。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Multiply.png" alt="Preview of Multiply blend mode" width="180"/> | `Multiply` | 将源颜色与目标颜色相乘，得到更暗的结果。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Hue.png" alt="Preview of Hue blend mode" width="180"/> | `Hue` | 生成一种使用源颜色色相、并保留目标颜色饱和度和明度的颜色。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Saturation.png" alt="Preview of Saturation blend mode" width="180"/> | `Saturation` | 生成一种使用源颜色饱和度、并保留目标颜色色相和明度的颜色。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Color.png" alt="Preview of Color blend mode" width="180"/> | `Color` | 生成一种使用源颜色色相和饱和度、并保留目标颜色明度的颜色。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Luminosity.png" alt="Preview of Luminosity blend mode" width="180"/> | `Luminosity` | 生成一种使用源颜色明度、并保留目标颜色色相和饱和度的颜色。 |

### 合成混合模式

合成混合模式只影响 alpha 通道，而不会改变颜色本身。

下面是示例中使用的图像：

| “A” 基础图像（destination） | “B” 覆盖图像（source） |
|:---:|:---:|
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/A.png" alt="Image A used as destination for composition examples" width="180"/> | <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/B.png" alt="Image B used as source for composition examples" width="180"/> |

下面列出了 Avalonia 当前支持的所有取值。请注意，这个演示对 alpha 通道非常敏感，因此网页背景可能会透过图像显示出来。

| 预览 | 枚举值 | 说明 |
|---|---|---|
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Source.png" alt="Preview of Source composition mode" width="180"/> | `Source` | 只保留源图像。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/SourceOver.png" alt="Preview of SourceOver composition mode" width="180"/> | `SourceOver` | 或 `Unspecified`，默认行为：将源图像叠放在目标图像之上。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/SourceIn.png" alt="Preview of SourceIn composition mode" width="180"/> | `SourceIn` | 仅保留与目标图像重叠的源图像部分，并替换目标图像。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/SourceOut.png" alt="Preview of SourceOut composition mode" width="180"/> | `SourceOut` | 仅在源图像落在目标图像之外的位置显示源图像。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/SourceAtop.png" alt="Preview of SourceAtop composition mode" width="180"/> | `SourceAtop` | 与目标图像重叠的源图像部分会替换目标图像。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Xor.png" alt="Preview of Xor composition mode" width="180"/> | `Xor` | 组合源图像与目标图像中不重叠的区域。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/Destination.png" alt="Preview of Destination composition mode" width="180"/> | `Destination` | 只保留目标图像。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/DestinationOver.png" alt="Preview of DestinationOver composition mode" width="180"/> | `DestinationOver` | 将目标图像叠放在源图像之上。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/DestinationIn.png" alt="Preview of DestinationIn composition mode" width="180"/> | `DestinationIn` | 仅保留与源图像重叠的目标图像部分，并替换源图像。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/DestinationOut.png" alt="Preview of DestinationOut composition mode" width="180"/> | `DestinationOut` | 仅在目标图像落在源图像之外的位置显示目标图像。 |
| <img src="/img/reference/animations-and-graphics/bitmap-blend-modes/DestinationAtop.png" alt="Preview of DestinationAtop composition mode" width="180"/> | `DestinationAtop` | 与源图像重叠的目标图像部分会替换源图像。 |

## 另请参阅

- [画刷](/docs/graphics-animation/brushes)：用于填充和描边的画刷类型。
- [绘制图形](/docs/graphics-animation/drawing-graphics)：形状、几何图形与图形系统。
- [自定义渲染](/docs/graphics-animation/custom-rendering)：使用 `DrawingContext` 进行绘制。
