---
id: accessibility
title: 无障碍支持
description: 通过 AutomationProperties、键盘导航和自定义 peer 来实现可访问的 Avalonia 应用。
doc-type: overview
---

Avalonia 通过自动化 peer 内置了无障碍支持，它们会将你的 UI 暴露给屏幕阅读器等辅助技术。本页介绍如何让你的 Avalonia 应用对所有用户都更易访问。

## Avalonia 中的无障碍如何工作

Avalonia 的无障碍模型使用 **automation peer**，这与 WPF 和 UWP 类似。每个控件都关联一个 `AutomationPeer`，它会向平台的无障碍 API 描述控件的角色、状态和内容（Windows 上是 UI Automation，macOS 上是 NSAccessibility，Linux 上是 AT-SPI）。

大多数内置控件都会自动创建自己的 automation peer。比如 `Button` 会把自己报告为按钮，`TextBox` 会报告为可编辑文本框，`CheckBox` 会报告为复选框，等等。

## AutomationProperties

`AutomationProperties` 类提供了一组附加属性，用来为控件提供无障碍元数据。这些属性不会影响 UI 的视觉外观，只会被辅助技术读取和使用。

### Name

这是最重要的无障碍属性。它提供了当控件获得焦点时，屏幕阅读器要朗读的文本：

```xml
<Button AutomationProperties.Name="Submit order"
        Content="{Binding SubmitIcon}" />
```

如果控件本身显示文本内容（例如 `Content` 是字符串的 `Button`，或者 `TextBlock`），automation peer 会自动使用该文本。以下情况则应显式设置 `Name`：
- 内容是没有文字的图片或图标
- 可见文本脱离上下文后含义不明确（例如多个“Edit”按钮）
- 控件本身没有可见内容

### HelpText

提供额外的描述性文本，通常会在控件名称之后被朗读：

```xml
<TextBox AutomationProperties.Name="Email"
         AutomationProperties.HelpText="Enter your email address to receive notifications" />
```

### LabeledBy

将控件与一个标签控件关联起来，并使用该标签文本作为可访问名称：

```xml
<TextBlock x:Name="NameLabel" Text="Full Name:" />
<TextBox AutomationProperties.LabeledBy="{Binding #NameLabel}" />
```

### AutomationId

这是一个稳定的 UI 自动化测试标识符。与 `Name` 不同，这个值不会被本地化，也不会随着界面语言变化而变化：

```xml
<Button AutomationProperties.AutomationId="SubmitOrderButton"
        Content="Submit" />
```

### AcceleratorKey and AccessKey

向辅助技术传达键盘快捷键：

```xml
<Button AutomationProperties.AcceleratorKey="Ctrl+S"
        Content="Save" />

<Button AutomationProperties.AccessKey="Alt+S"
        Content="_Save" />
```

### ControlTypeOverride

覆盖向辅助技术报告的控件类型。当某个自定义控件应被朗读成某种标准控件类型时，可以使用它：

```xml
<Border AutomationProperties.ControlTypeOverride="Button"
        AutomationProperties.Name="Custom button"
        PointerPressed="OnBorderClick">
    <TextBlock Text="Click me" />
</Border>
```

### LandmarkType

将 UI 的某个区域标识为可导航的地标。屏幕阅读器通常允许用户在这些地标之间快速跳转：

```xml
<StackPanel AutomationProperties.LandmarkType="Navigation">
    <!-- Navigation menu -->
</StackPanel>

<ScrollViewer AutomationProperties.LandmarkType="Main">
    <!-- Main content -->
</ScrollViewer>
```

可用的地标类型包括：`Banner`、`Complementary`、`ContentInfo`、`Region`、`Form`、`Main`、`Navigation` 和 `Search`。

:::note
在 Windows 上，必须将 `AccessibilityView` 至少设置为 `Control`，Narrator 才能识别这些地标并启用相关快捷导航（Narrator+N）。
:::

### HeadingLevel

将某个控件标记为特定级别的标题。屏幕阅读器会利用标题进行文档导航。为了获得更好的跨平台兼容性，建议使用 1 到 6 的值（macOS 支持 0-6，Windows 支持 1-9）：

```xml
<TextBlock AutomationProperties.HeadingLevel="1"
           Text="Settings" FontSize="24" />

<TextBlock AutomationProperties.HeadingLevel="2"
           Text="Appearance" FontSize="18" />
```

### LiveSetting

控制动态内容变化应如何被屏幕阅读器朗读：

```xml
<!-- Polite：在屏幕阅读器空闲时朗读 -->
<TextBlock AutomationProperties.LiveSetting="Polite"
           Text="{Binding StatusMessage}" />

<!-- Assertive：立即朗读，并打断当前语音 -->
<TextBlock AutomationProperties.LiveSetting="Assertive"
           Text="{Binding ErrorMessage}" />
```

### ItemStatus

描述元素当前的状态。屏幕阅读器会朗读这段文本，以补充说明该元素所处的状态：

```xml
<ListBoxItem AutomationProperties.ItemStatus="Downloading (45%)"
             Content="{Binding FileName}" />
```

### ItemType

以对用户更有意义的方式描述元素类型。它会在控件类型之外，补充应用特定的上下文信息：

```xml
<ListBoxItem AutomationProperties.ItemType="PDF Document"
             Content="{Binding FileName}" />
```

### AccessibilityView

控制某个控件是否出现在自动化树中：

```xml
<!-- 将装饰性元素从无障碍树中移除 -->
<Image Source="decorative-line.png"
       AutomationProperties.AccessibilityView="Raw" />
```

| 值 | 含义 |
|---|---|
| `Default` | 由控件的 automation peer 决定 |
| `Raw` | 仅包含在原始（未过滤）树中 |
| `Control` | 包含在控件视图中 |
| `Content` | 包含在内容视图中 |

## 键盘无障碍

可访问的应用必须能够仅通过键盘完整操作：

### Tab navigation

请确保所有交互控件都能通过 Tab 键访问。对需要参与 Tab 导航的控件设置 `IsTabStop="True"`，并通过 `TabIndex` 控制顺序：

```xml
<TextBox TabIndex="1" />
<TextBox TabIndex="2" />
<Button TabIndex="3" Content="Submit" />
```

完整的键盘导航模型，包括 `KeyboardNavigation.TabNavigation` 模式与 `XYFocus` 方向导航，请参阅 [Focus](/docs/input-interaction/focus)。

### Keyboard shortcuts

对于只能通过指针完成的交互，请使用 `HotKey` 或 `KeyBinding` 提供对应的键盘操作方式：

```xml
<Button Content="_Save" HotKey="Ctrl+S" Command="{Binding SaveCommand}" />
```

下划线前缀会创建访问键（在 Windows/Linux 上通常是 Alt+S）。更多细节请参阅 [Keyboard and Hotkeys](/docs/input-interaction/keyboard-and-hotkeys)。

### Focus indicators

请确保焦点是可见的。当 `:focus-visible` 激活时，Avalonia 默认会显示一个 `FocusAdorner`（即围绕焦点控件的一圈边框）。如果你创建了自定义控件模板，请确认焦点仍然清晰可见：

```xml
<Style Selector="Button.custom:focus-visible">
    <Setter Property="BorderBrush" Value="{DynamicResource SystemAccentColor}" />
    <Setter Property="BorderThickness" Value="2" />
</Style>
```

## 创建自定义 automation peer

当你构建自定义控件时，Avalonia 可能没有足够的信息把它正确描述给辅助技术。这时可以重写 `OnCreateAutomationPeer` 来提供自定义 peer：

```csharp
public class RatingControl : Control
{
    public static readonly StyledProperty<int> ValueProperty =
        AvaloniaProperty.Register<RatingControl, int>(nameof(Value));

    public int Value
    {
        get => GetValue(ValueProperty);
        set => SetValue(ValueProperty, value);
    }

    protected override AutomationPeer OnCreateAutomationPeer()
    {
        return new RatingControlAutomationPeer(this);
    }
}

public class RatingControlAutomationPeer : ControlAutomationPeer
{
    public RatingControlAutomationPeer(RatingControl owner) : base(owner) { }

    protected override AutomationControlType GetAutomationControlTypeCore()
        => AutomationControlType.Slider;

    protected override string? GetNameCore()
        => $"Rating: {((RatingControl)Owner).Value} out of 5";
}
```

### 需要重点重写的方法

| 方法 | 作用 |
|---|---|
| `GetAutomationControlTypeCore()` | 控件类型（如 Button、TextBox、Slider 等） |
| `GetNameCore()` | 屏幕阅读器朗读的可访问名称 |
| `GetHelpTextCore()` | 额外的描述性文本 |
| `GetAutomationIdCore()` | 用于测试的稳定标识符 |
| `IsContentElementCore()` | 控件是否出现在内容视图中 |
| `IsControlElementCore()` | 控件是否出现在控件视图中 |

## 数据验证错误

验证错误会自动暴露给辅助技术。当某个控件（例如 `TextBox`）存在验证错误时（这些错误可能来自数据注解、`INotifyDataErrorInfo` 或异常），`DataValidationErrors` 控件会通过它的 automation peer 将这些错误作为帮助文本报告出去。屏幕阅读器会在控件获得焦点时朗读这些错误，并且验证错误文本的优先级高于 tooltip 文本。

这不需要额外配置。只要你的控件使用了 Avalonia 的 [data validation](/docs/data-binding/binding-validation) 系统，这些错误默认就是可访问的。

## 无障碍检查清单

在检查应用时，可以参考下面这份清单：

- 所有交互控件都能通过键盘访问（Tab/Shift+Tab）
- 所有可聚焦控件的焦点指示都清晰可见
- 图片和图标都通过 `AutomationProperties.Name` 提供了可访问名称
- 表单字段都有标签（通过 `AutomationProperties.Name`、`AutomationProperties.LabeledBy` 或文本内容）
- 动态内容通过 `AutomationProperties.LiveSetting` 提供朗读提示
- 颜色不是传递信息的唯一方式（也可以同时使用形状、文本或图标）
- 文本满足最小对比度要求（普通文本 4.5:1，大号文本 3:1）
- 自定义控件拥有合适的 automation peer
- 装饰性元素已从自动化树中排除

## 平台支持

Avalonia 的无障碍支持会因平台而异：

| 平台 | 无障碍 API | 支持状态 |
|---|---|---|
| Windows | UI Automation (UIA) | 完整支持 |
| macOS | NSAccessibility | 完整支持 |
| Linux | [AT-SPI2](/docs/platform-specific-guides/linux#accessibility) | 完整支持 |
| iOS | UIAccessibility | 支持 |
| Android | AccessibilityNodeInfo | 支持 |
| Browser (WASM) | ARIA attributes | 部分支持 |

在 Linux 上，只要 AT-SPI2 可用，Avalonia 就会自动通过 D-Bus 暴露无障碍树。像 Orca 这样的屏幕阅读器可以发现并操作所有标准 Avalonia 控件。安装与测试说明请参阅 [Linux platform guide](/docs/platform-specific-guides/linux#accessibility)。

## 另请参阅

- [Focus](/docs/input-interaction/focus)：键盘焦点导航与 Tab 顺序。
- [Keyboard and Hotkeys](/docs/input-interaction/keyboard-and-hotkeys)：键盘快捷键与访问键。
- [Custom Controls](/docs/custom-controls)：构建具备正确无障碍支持的自定义控件。
- [Desktop Linux](/docs/platform-specific-guides/linux#accessibility)：在 Linux 上使用 Orca 和 Accerciser 测试无障碍能力。
