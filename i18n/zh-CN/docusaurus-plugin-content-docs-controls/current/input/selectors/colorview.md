---
id: colorview
title: ColorView
---

使用色谱、调色板和颜色分量滑块来呈现并编辑颜色。

## 常用属性

| 属性 | 说明 |
|----------|-------------|
| `Color` | 获取或设置当前在 RGB 色彩模型中的选中颜色。对于控件作者，建议改用 `HsvColor`，以避免精度损失和颜色漂移。 |
| `ColorModel` | 获取或设置滑块当前使用的颜色模型。此属性仅适用于颜色分量标签页；色谱标签页始终使用 HSV，而调色板标签页只包含预定义颜色。 |
| `ColorSpectrumComponents` | 获取或设置色谱中显示的两个 HSV 颜色分量。 |
| `ColorSpectrumShape` | 获取或设置色谱的显示形状。 |
| `HexInputAlphaPosition` | 获取或设置十六进制输入框中 alpha 分量相对于其他颜色分量的位置。 |
| `HsvColor` | 获取或设置当前在 HSV 色彩模型中的选中颜色。所有场景下都建议优先使用它，而不是 `Color` 属性。由于 `ColorSpectrum` 内部使用 HSV 色彩模型，使用该属性可避免精度损失和颜色漂移。 |
| `IsAccentColorsVisible` | 获取或设置是否在预览颜色旁显示强调色。 |
| `IsAlphaEnabled` | 获取或设置 alpha 分量是否启用。禁用后（设为 false），alpha 分量会固定为最大值，相关编辑控件也会被禁用。 |
| `IsAlphaVisible` | 获取或设置 alpha 分量的编辑控件（滑块和 TextBox）是否可见。隐藏时，现有 alpha 值会被保留。注意，`IsComponentTextInputVisible` 也会影响 alpha 分量 TextBox 的可见性。 |
| `IsColorComponentsVisible` | 获取或设置颜色分量标签页/面板/页面（子视图）是否可见。 |
| `IsColorModelVisible` | 获取或设置当前颜色模型指示器/选择器是否可见。 |
| `IsColorPaletteVisible` | 获取或设置颜色调色板标签页/面板/页面（子视图）是否可见。 |
| `IsColorPreviewVisible` | 获取或设置颜色预览是否可见。注意，强调色的可见性由 `IsAccentColorsVisible` 单独控制。 |
| `IsColorSpectrumVisible` | 获取或设置颜色色谱标签页/面板/页面（子视图）是否可见。 |
| `IsColorSpectrumSliderVisible` | 获取或设置颜色色谱中的第三个分量滑块是否可见。 |
| `IsComponentSliderVisible` | 获取或设置颜色分量滑块是否可见。该属性控制所有颜色分量，但 alpha 也可通过 `IsAlphaVisible` 单独控制。 |
| `IsComponentTextInputVisible` | 获取或设置颜色分量文本输入框是否可见。该属性控制所有颜色分量，但 alpha 也可通过 `IsAlphaVisible` 单独控制。 |
| `IsHexInputVisible` | 获取或设置十六进制颜色值文本输入框是否可见。 |
| `MaxHue` | 获取或设置 Hue 分量的最大值，范围为 0..359。该属性必须大于 `MinHue`。 |
| `MaxSaturation` | 获取或设置 Saturation 分量的最大值，范围为 0..100。该属性必须大于 `MinSaturation`。 |
| `MaxValue` | 获取或设置 Value 分量的最大值，范围为 0..100。该属性必须大于 `MinValue`。 |
| `MinHue` | 获取或设置 Hue 分量的最小值，范围为 0..359。该属性必须小于 `MaxHue`。 |
| `MinSaturation` | 获取或设置 Saturation 分量的最小值，范围为 0..100。该属性必须小于 `MaxSaturation`。 |
| `MinValue` | 获取或设置 Value 分量的最小值，范围为 0..100。该属性必须小于 `MaxValue`。 |
| `PaletteColors` | 获取或设置调色板中的单个颜色集合。通常不需要手动设置，而应通过向 `Palette` 属性提供 `IColorPalette` 来自动生成。 |
| `PaletteColumnCount` | 获取或设置调色板每一行（或每个区段）中的颜色数量。在标准调色板中，行表示色阶，列表示颜色。通常不需要手动设置，而应通过向 `Palette` 属性提供 `IColorPalette` 来自动生成。 |
| `Palette` | 获取或设置颜色调色板。设置后会自动更新 `PaletteColors` 和 `PaletteColumnCount`，并覆盖现有值。 |
| `SelectedIndex` | 获取或设置当前选中标签页/面板/页面（子视图）的索引。使用默认控件主题时，该属性设计为与 `ColorViewTab` 枚举配合使用。`ColorViewTab` 枚举定义了三个标准标签页的索引值，例如 `SelectedIndex = (int)ColorViewTab.Palette`。 |

:::note
用于控制可见性的属性采用 `IsThingVisible` 命名模式，而不是 `ShowThing`，因为界面中的某些元素既可以单独控制启用状态，也可以单独控制可见状态。这种命名方式也与 `Control` 的风格保持一致。
:::

## 伪类

无

## 模板部件

| 名称 | 类型 | 说明 |
|------|----- |-------------|
| `PART_HexTextBox` | TextBox | 提供一个十六进制颜色表示的输入/输出框，控件可对其内容进行解析。 |
| `PART_TabControl` | TabControl | 用于在色谱、调色板和颜色分量标签页/面板/页面（子视图）之间导航的主控件。该模板部件是可选的，仅在 `SelectedIndex` 的某些验证场景中才需要。 |

## 另请参阅

- [ColorView API 参考](/api/avalonia/controls/colorview)
- [`ColorView.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls.ColorPicker/ColorView/ColorView.cs)
