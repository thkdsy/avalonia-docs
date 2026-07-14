---
id: colorpicker
title: ColorPicker
---

import ColorPaletteFluent from '/img/controls/colorpicker/color-palette-fluent.png';
import ColorPaletteFlat from '/img/controls/colorpicker/color-palette-flat.png';
import ColorPaletteFlatHalf from '/img/controls/colorpicker/color-palette-flat-half.png';
import ColorPaletteMaterial from '/img/controls/colorpicker/color-palette-material.png';
import ColorPaletteMaterialHalf from '/img/controls/colorpicker/color-palette-material-half.png';
import ColorPaletteSixteen from '/img/controls/colorpicker/color-palette-sixteen.png';

[`ColorPicker`](/api/avalonia/controls/colorpicker) 是一个高度可自定义的通用控件，用户可以使用它在 RGB 或 HSV 色彩空间中选择颜色。这个实现不仅提供了一个开箱即用的取色器，也提供了一组基础控件，方便开发者构建自己的颜色选择器。

`ColorPicker` 包含一组相关控件（组件）：

 * [`ColorSpectrum`](/api/avalonia/controls/primitives/colorspectrum)（基础控件）：用于颜色选择的二维色谱。
 * [`ColorSlider`](/api/avalonia/controls/primitives/colorslider)（基础控件）：带有背景的滑块，用于表示单个颜色分量。
 * [`ColorPreviewer`](/api/avalonia/controls/primitives/colorpreviewer)（基础控件）：显示预览颜色，并可选择显示强调色。
 * [`ColorView`](/api/avalonia/controls/colorview)：使用色谱、调色板和分量滑块来呈现并编辑颜色。
 * `ColorPicker`：在下拉界面中使用色谱、调色板和分量滑块来呈现并编辑颜色。只有在打开下拉 Flyout 时才能编辑，否则只显示预览颜色。

每个基础组件都可以单独使用，也可以与其他组件自由组合。这种高度可组合性是许多其他颜色选择器实现所不具备的。例如，你可以快速将 `ColorSpectrum`、`ColorSlider` 和 `ColorPreviewer` 绑定在一起，构建一个拥有全新设计的自定义颜色选择器。

术语说明：“color picker” 通常指整组相关控件，而 `ColorPicker` 则特指该具体控件。

## 这是合适的控件吗？

这个控件适合直接用于颜色选择，同时兼顾用户友好性和开发者可定制性。你既可以使用画布式的 `ColorView` 控件，也可以使用紧凑型的 `ColorPicker` 下拉控件。

对于有更特殊需求的应用，你可以独立定制每个控件和基础组件，从而创建新的颜色选择器，而无需重新实现所有复杂的渲染与颜色逻辑。这对于匹配特定应用的设计和可用性需求非常有帮助。

使用这些控件的开发者可以：
 1. 直接在应用中原样使用 `ColorView` 或 `ColorPicker`
 2. 使用内置属性自定义 `ColorView` 或 `ColorPicker`。这些属性支持大量调整，例如禁用分量滑块、显示不同调色板，或隐藏除色谱标签外的其他区域。
 3. 使用现有基础组件创建一个满足特定应用设计和可用性要求的新颜色选择器。
 4. 重新为现有组件编写模板，创建一个全新的完全自定义颜色选择器。

## 在应用中使用

Avalonia 会运行在一些资源受限的环境中，例如嵌入式设备。因此，像 `ColorPicker` 这样较大的控件并未包含在主 Avalonia NuGet 包中。这意味着如果你想在应用中使用 `ColorPicker`，需要额外做一些配置：

 1. 将 `Avalonia.Controls.ColorPicker` NuGet 包添加到项目中。它**必须**与你项目中其他 Avalonia 包的版本保持一致。
 2. 在 `App.axaml` 中为所有颜色选择器控件添加主题和样式：
    * Fluent 主题使用 `<StyleInclude Source="avares://Avalonia.Controls.ColorPicker/Themes/Fluent/Fluent.xaml" />` **或者**
    * Simple 主题使用 `<StyleInclude Source="avares://Avalonia.Controls.ColorPicker/Themes/Simple/Simple.xaml" />`

:::note
对于某些默认已包含全部控件的主题包（例如 FluentAvalonia），则不需要执行这一步。
:::

## 背景

这个控件最初源于对 UWP（后来的 WinUI）中同类控件的一次重新设计，基础设计思路参考了 Windows Community Toolkit 的实现。WinUI 的 `ColorPicker` 对较小屏幕并不友好，而且其整体设计和可用性对用户与开发者来说都还有改进空间。

尽管 WinUI 控件具备不少功能，但它仍然没有达到理想状态。若要重新编写模板并进行自定义，需要付出很大代价（部分原因是各个组件之间耦合非常严重）。它还大量依赖模板部件和代码后置。Avalonia 版本的这个控件是一次完整重写，目标就是修复这些问题，并成为更优秀的 XAML 颜色选择器设计。

从 WinUI 版本吸取经验后，主要改进包括：
 * `ColorPicker` 被实现为下拉式控件（与其他“选择器”一致）。同时也提供 `ColorView` 控件，供需要画布式取色器的场景使用（类似 WinUI）。
 * Avalonia 控件尽可能将逻辑放在 XAML 控件主题中，并将代码后置减到最低。这显著提升了可组合性，也让应用开发者几乎可以自定义这些控件的每个部分（在大多数情况下甚至包括基础组件）。
 * `ColorSlider` 和 `ColorSpectrum` 等基础组件都是完全自包含的，可单独使用，从而方便开发者构建自定义颜色选择器实现。
 * Avalonia 核心中新增了 `HsvColor` 结构（与 `Color` 和 `HslColor` 并列），现在所有颜色选择器控件都使用它。这简化了代码后置，并使基础组件与控件之间的颜色属性绑定成为可能。颜色选择器控件内部以 HSV 色彩空间工作。
 * `HsvColor` 与 `ColorSlider` 的组合相比 WinUI 带来了更强的能力，同时也更容易重新编写模板。
 * 新增了大量属性（比 WinUI 更多）来控制 `ColorView` 可见性的各个方面。每个标签页以及大多数子区域都可以单独隐藏，这让你无需重写模板或使用复杂样式选择器，也能完成大量设计定制。
 * 通过 `IColorPalette` 接口添加了颜色调色板支持（与 Windows Community Toolkit 相同）。WinUI 版本的该控件则不支持颜色调色板。
 * 还新增了 `SelectedIndex`、`ColorModel` 等属性，用于自定义颜色选择器并让它进入预定义状态。例如，WinUI ColorPicker 总是默认使用 RGB，且无法通过代码或 XAML 修改；而此实现不存在这些限制。

## 控件与基础组件

| 控件 | 链接 |
|---------|------|
| `ColorPicker` | |
| `ColorView` | 请参阅专门的 [`ColorView`](/controls/input/selectors/colorview) 页面。 |
| `ColorSpectrum` | |
| `ColorSlider` | |
| `ColorPreviewer` | |

## 颜色调色板

这里提供了多个实现 `IColorPalette` 接口的预定义颜色调色板。你可以将这些调色板实例设置给 `ColorView` 或 `ColorPicker` 的 `Palette` 属性。

<table>
  <tr>
    <th>调色板</th>
    <th>说明</th>
  </tr>
  <tr>
    <td>
      <Image light={ColorPaletteFluent} alt="Fluent Color Palette" position="center" maxWidth={400} cornerRadius="true"/>
    </td>
    <td>包含 Windows 10 及更高版本中的 Fluent 调色板。这是默认调色板。</td>
  </tr>
  <tr>
    <td>
      <Image light={ColorPaletteFlat} alt="Flat UI Color Palette" position="center" maxWidth={400} cornerRadius="true"/>
    </td>
    <td>包含完整的 <a href="https://github.com/designmodo/Flat-UI">Flat UI 调色板</a>。</td>
  </tr>
  <tr>
    <td>
      <Image light={ColorPaletteFlatHalf} alt="Flat UI Half Color Palette" position="center" maxWidth={400} cornerRadius="true"/>
    </td>
    <td>包含一半的 <a href="https://github.com/designmodo/Flat-UI">Flat UI 调色板</a>，以提升可用性，尤其适合移动设备。</td>
  </tr>
  <tr>
    <td>
      <Image light={ColorPaletteMaterial} alt="Material Color Palette" position="center" maxWidth={400} cornerRadius="true"/>
    </td>
    <td>包含大部分 <a href="https://material.io/design/color/the-color-system.html#tools-for-picking-colors">Material Design 调色板</a>。为了让调色板保持统一且呈矩形布局，做了以下调整：1. 排除了每种颜色的 A100-A700 色阶，因为这些色阶并非所有颜色都具备（例如 Brown/Gray）。2. Black/White 作为独立颜色，也被排除在外。</td>
  </tr>
  <tr>
    <td>
      <Image light={ColorPaletteMaterialHalf} alt="Material Half Color Palette" position="center" maxWidth={400} cornerRadius="true"/>
    </td>
    <td>包含上面所示 <a href="https://material.io/design/color/the-color-system.html#tools-for-picking-colors">Material Design 调色板</a>的一半，以提升可用性，尤其适合移动设备。</td>
  </tr>
  <tr>
    <td>
      <Image light={ColorPaletteSixteen} alt="Sixteen Color Palette" position="center" maxWidth={400} cornerRadius="true"/>
    </td>
    <td>包含 HTML 4.01 规范中的标准 <a href="https://en.wikipedia.org/wiki/Web_colors#HTML_color_names">十六色调色板</a>。</td>
  </tr>
</table>

## 另请参阅

- [ColorPicker API 参考](/api/avalonia/controls/colorpicker)
- [`ColorPicker.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls.ColorPicker/ColorPicker/ColorPicker.cs)
