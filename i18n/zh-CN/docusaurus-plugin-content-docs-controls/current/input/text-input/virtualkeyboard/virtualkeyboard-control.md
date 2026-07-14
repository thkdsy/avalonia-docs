---
id: virtualkeyboard-control
title: VirtualKeyboard 控件参考
tags:
  - avalonia pro
  - avalonia enterprise
---

import VirtualKeyboardStyles from '/img/avalonia-pro/virtual-keyboard/styles.png';

`VirtualKeyboard` 是一个独立的控件，提供屏幕键盘。此控件可以手动放置在应用程序的布局中。与 `VirtualKeyboardScope`（它根据焦点自动管理键盘可见性）不同，`VirtualKeyboard` 被明确指向特定的目标输入元素。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 概述

`VirtualKeyboard` 让您直接控制键盘的放置和行为。无论哪个控件具有输入焦点，它都会将输入直接发送到其指定的目标。这使其适用于基于焦点的自动键盘显示不合适的专用输入场景。

## 属性

| 属性 | 类型 | 描述 |
|----------|------|-------------|
| `Target` | `IInputElement` | 获取或设置接收键盘击键的输入元素。 |
| `InputMethods` | `IEnumerable<VirtualKeyboardInputMethod>` | 获取或设置可供用户使用的输入方法集合。 |

## 使用示例

### 最小实现

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <StackPanel>
        <TextBox x:Name="InputField" />
        <VirtualKeyboard Target="{Binding ElementName=InputField}"
                         InputMethods="en-US:kbd:standard" />
    </StackPanel>
</Window>
```

### 多种输入方法

```xml
<StackPanel>
    <TextBox x:Name="EmailField" PlaceholderText="电子邮件地址" />
    <VirtualKeyboard Target="{Binding ElementName=EmailField}"
                     InputMethods="en-US:kbd:standard, de:kbd:standard, ja:ime:kana" />
</StackPanel>
```

### 代码后置配置

```csharp
// 使用 SelectMany + ToList 获取特定语言的输入方法
var inputMethods = new[] { "en-US", "ja", "de" }
    .SelectMany(VirtualKeyboardInputMethod.GetInputMethodsForLanguage)
    .ToList();

// 创建并配置 VirtualKeyboard
var keyboard = new VirtualKeyboard
{
    Target = myTextBox,
    InputMethods = inputMethods
};

// 将其添加到可视化树
myContainer.Children.Add(keyboard);
```

## 使用 `TextInputOptions`

`TextInputOptions` 附加属性可以应用于目标元素，以自定义键盘行为：

```xml
<StackPanel>
    <TextBox x:Name="EmailField"
             TextInputOptions.ContentType="Email"
             TextInputOptions.ReturnKeyType="Next" />

    <VirtualKeyboard Target="{Binding ElementName=EmailField}"
                     InputMethods="en-US:kbd:standard" />
</StackPanel>
```

## 何时使用 `VirtualKeyboard` vs. `VirtualKeyboardScope`

### 选择 VirtualKeyboard 当：

- **固定目标**：您需要键盘始终针对特定的输入控件，无论焦点如何。
- **专用输入**：您正在构建自定义输入体验，其中焦点不驱动键盘目标。

### 选择 VirtualKeyboardScope 当：

- **标准输入**：您希望键盘自动跟随焦点。
- **更简单的集成**：您更喜欢基于容器的方法，配置选项更少。
- **自动可见性**：您希望根据焦点变化自动显示/隐藏行为。

## 最佳实践

1. **设置有效的目标。**
   - 始终将 `Target` 属性设置为可以接收击键的有效输入元素。
   - 如果目标无效，键盘输入将无处可去。

2. **小心放置键盘。**
   - 将键盘放置在不会遮挡重要内容的位置，通常位于屏幕底部。
   - `VirtualKeyboard` 不会自动管理内容滚动，这与 `VirtualKeyboardScope` 不同，因此您可能需要自己处理滚动或布局调整。

3. **选择相关的输入方法。**
   - 选择适合目标受众的输入方法。
   - 对于国际化应用程序，包括所有支持区域的布局。

4. **最小化内存使用。**
   - 如果动态创建键盘，请记得在不再需要时将其从可视化树中移除。

5. **设计响应式布局。**
   - 规划布局以容纳键盘的空间需求。
   - 考虑使用带有行定义的 [`Grid`](/controls/layout/panels/grid) 来为键盘分配空间。

## 样式设置

`VirtualKeyboard` 通过命名资源进行样式设置。您可以在应用程序中覆盖这些资源以自定义键盘元素的外观。

### 可自定义的资源

<Image light={VirtualKeyboardStyles} maxWidth={400} alignment="center" />

以下是在主题或资源字典中可覆盖的资源列表：

| 键 | 类型 | 默认值 |
|---|---|---|
| `KeyboardActionButtonBackground` | Brush | `Goldenrod` |
| `KeyboardActionButtonBackgroundPressed` | Brush | `PaleGoldenrod` |
| `KeyboardButtonBackground` | Brush | `GhostWhite` |
| `KeyboardButtonBackgroundPressed` | Brush | `FloralWhite` |
| `KeyboardButtonBorderBrush` | Brush | `Black` |
| `KeyboardButtonFontSize` | Double | `24` |
| `KeyboardButtonForeground` | Brush | `Black` |
| `KeyboardFunctionalButtonBackground` | Brush | `LightSteelBlue` |
| `KeyboardFunctionalButtonBackgroundPressed` | Brush | `LightBlue` |
| `KeyboardPaneBackground` | Brush | `DarkGray` |
| `KeyboardPanePadding` | Thickness |  `4` |
| `KeyboardPopupKeySelectedBackground` | Brush | `PaleTurquoise` |

### 如何覆盖

要自定义，请在您的应用程序主题或资源字典中定义这些资源。例如：

```xml
<SolidColorBrush x:Key="KeyboardButtonForeground" Color="#FF0000" />
```

### 示例：自定义主题

```xml
<ResourceDictionary>
  <SolidColorBrush x:Key="KeyboardPaneBackground" Color="#FFD700" />
  <system:Double x:Key="KeyboardButtonFontSize">36</system:Double>
  <!-- 根据需要添加更多覆盖 -->
</ResourceDictionary>
```

## 输入方法

| 标识符 | 描述 | 说明 |
| --- | --- | --- |
|`af:kbd:standard` | 南非荷兰语 | |
|`ar:kbd:standard` | 阿拉伯语 | |
|`hy-AM:kbd:standard` | 亚美尼亚语（亚美尼亚）音标 | |
|`az-AZ:kbd:standard` | 阿塞拜疆语（阿塞拜疆） | |
|`eu-ES:kbd:standard` | 巴斯克语（西班牙） | |
|`be-BY:kbd:standard` | 白俄罗斯语（白俄罗斯） | |
|`bn-BD:kbd:standard` | 孟加拉语（孟加拉国） | |
|`bn-IN:kbd:standard` | 孟加拉语（印度） | |
|`bg:kbd:standard` | 保加利亚语 | |
|`bg:kbd:bds` | 保加利亚语（BDS） | |
|`ca:kbd:standard` | 加泰罗尼亚语 | |
|`hr:kbd:standard` | 克罗地亚语 | |
|`cs:kbd:standard` | 捷克语 | |
|`da:kbd:standard` | 丹麦语 | |
|`nl:kbd:standard` | 荷兰语 | |
|`nl-BE:kbd:standard` | 荷兰语（比利时） | |
|`en-GB:kbd:standard` | 英语（英国） | |
|`en-IN:kbd:standard` | 英语（印度） | |
|`en-US:kbd:standard` | 英语（美国） | |
|`eo:kbd:standard` | 世界语 | |
|`et-EE:kbd:standard` | 爱沙尼亚语（爱沙尼亚） | |
|`fi:kbd:standard` | 芬兰语 | |
|`fr:kbd:standard` | 法语 | |
|`fr-CA:kbd:standard` | 法语（加拿大） | |
|`fr-CH:kbd:standard` | 法语（瑞士） | |
|`gl-ES:kbd:standard` | 加利西亚语（西班牙） | |
|`ka-GE:kbd:standard` | 格鲁吉亚语（格鲁吉亚） | |
|`de:kbd:standard` | 德语 | |
|`de-CH:kbd:standard` | 德语（瑞士） | |
|`el:kbd:standard` | 希腊语 | |
|`hi:kbd:standard` | 印地语 | |
|`hi:kbd:compact` | 印地语（紧凑型） | |
|`hu:kbd:standard` | 匈牙利语 | |
|`is:kbd:standard` | 冰岛语 | |
|`it:kbd:standard` | 意大利语 | |
|`it-CH:kbd:standard` | 意大利语（瑞士） | |
|`ja:ime:kana` | 日语（假名） | |
|`ko:ime:hangul` | 韩语（韩文） | |
|`kn-IN:kbd:standard` | 卡纳达语（印度） | |
|`kk:kbd:standard` | 哈萨克语 | |
|`km-KH:kbd:standard` | 高棉语（柬埔寨） | |
|`ky:kbd:standard` | 吉尔吉斯语 | |
|`lo-LA:kbd:standard` | 老挝语（老挝） | |
|`lv:kbd:standard` | 拉脱维亚语 | |
|`lt:kbd:standard` | 立陶宛语 | |
|`mk:kbd:standard` | 马其顿语 | |
|`ml-IN:kbd:standard` | 马拉雅拉姆语（印度） | |
|`mr-IN:kbd:standard` | 马拉地语（印度） | |
|`mn-MN:kbd:standard` | 蒙古语（蒙古） | |
|`ne-NP:kbd:romanized` | 尼泊尔语（拉丁化） | |
|`ne-NP:kbd:traditional` | 尼泊尔语（传统） | |
|`nb:kbd:standard` | 挪威博克马尔语 | |
|`fa:kbd:standard` | 波斯语 | |
|`pl:kbd:standard` | 波兰语 | |
|`pt-BR:kbd:standard` | 葡萄牙语（巴西） | |
|`pt-PT:kbd:standard` | 葡萄牙语（葡萄牙） | |
|`ro:kbd:standard` | 罗马尼亚语 | |
|`ru:kbd:standard` | 俄语 | |
|`sr-Cyrl:kbd:standard` | 塞尔维亚语（西里尔字母） | |
|`sr-Latn:kbd:standard` | 塞尔维亚语（拉丁字母） | |
|`sk:kbd:standard` | 斯洛伐克语 | |
|`sl:kbd:standard` | 斯洛文尼亚语 | |
|`es:kbd:standard` | 西班牙语 | |
|`es-419:kbd:standard` | 西班牙语（拉丁美洲） | |
|`es-US:kbd:standard` | 西班牙语（美国） | |
|`sw:kbd:standard` | 斯瓦希里语 | |
|`sv:kbd:standard` | 瑞典语 | |
|`tl:kbd:standard` | 他加禄语 | |
|`ta-IN:kbd:standard` | 泰米尔语（印度） | |
|`ta-SG:kbd:standard` | 泰米尔语（新加坡） | |
|`te-IN:kbd:standard` | 泰卢固语（印度） | |
|`th:kbd:standard` | 泰语 | |
|`tr:kbd:standard` | 土耳其语 | |
|`uk:kbd:standard` | 乌克兰语 | |
|`uz-UZ:kbd:standard` | 乌兹别克语（乌兹别克斯坦） | |
|`vi:kbd:standard` | 越南语 | |
|`zu:kbd:standard` | 祖鲁语 | |
|`zh:ime:rime` | 中文（RIME） | 需要 `Avalonia.Controls.VirtualKeyboard.Ime.Rime` 插件包。 |

## RIME 输入法引擎

[RIME](https://rime.im/) 输入法引擎提供中文文本输入（拼音等）。它作为单独的插件包分发，包含原生二进制文件和方案数据。

### 安装 RIME

```bash
dotnet add package Avalonia.Controls.VirtualKeyboard.Ime.Rime
```

此包包含所有支持平台的 RIME 原生二进制文件和 Luna 拼音方案数据。

### 启用 RIME

在您的 `AppBuilder` 上调用 `WithVirtualKeyboardRimePlugin()`，并确保在 `VirtualKeyboardOptions` 中设置了 `DataPath`：

```csharp
using Avalonia.Controls;

public static AppBuilder BuildAvaloniaApp()
    => AppBuilder.Configure<App>()
        .UsePlatformDetect()
        .WithVirtualKeyboardOptions(new VirtualKeyboardOptions
        {
            DataPath = Path.Combine(
                Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData),
                "MyApp", "keyboard-data")
        })
        .WithVirtualKeyboardRimePlugin()
        .LogToTrace();
```

:::warning
使用 RIME 时 **必须** 设置 `VirtualKeyboardOptions.DataPath`。此路径指定一个可写目录，RIME 在其中存储用户词典和编译的方案数据。如果未设置 `DataPath`，应用程序将在启动时抛出 `InvalidOperationException`。
:::

### 在 XAML 中使用 RIME

```xml
<VirtualKeyboardScope InputMethods="en-US:kbd:standard, zh:ime:rime">
    <StackPanel>
        <TextBox PlaceholderText="用英语或中文输入"/>
    </StackPanel>
</VirtualKeyboardScope>
```

## 另请参阅

- [VirtualKeyboardScope](/controls/input/text-input/virtualkeyboard/virtualkeyboardscope)：自动管理键盘可见性的容器控件。
