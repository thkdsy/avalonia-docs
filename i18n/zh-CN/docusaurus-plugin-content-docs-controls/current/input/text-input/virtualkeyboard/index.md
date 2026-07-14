---
id: index
title: 虚拟键盘概述
tags:
  - avalonia pro
  - avalonia enterprise
---

虚拟键盘组件为 Avalonia 应用程序提供了一个屏幕键盘。它专为触摸或自助服务终端场景设计，在这些场景中物理键盘可能不可用，从而通过触摸屏或鼠标点击实现文本输入。

:::info
此组件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

虚拟键盘组件包括以下类：

- [`VirtualKeyboardScope`](/controls/input/text-input/virtualkeyboard/virtualkeyboardscope)：管理键盘可见性和输入方法的容器控件。
- [`VirtualKeyboard`](/controls/input/text-input/virtualkeyboard/virtualkeyboard-control)：可以手动放置的实际键盘控件。
- `VirtualKeyboardInputMethod`：表示特定的输入方法或键盘布局。

## 入门指南

1. 通过运行 `dotnet add package` 安装 `Avalonia.Controls.VirtualKeyboard` NuGet 包。

```bash
dotnet add package Avalonia.Controls.VirtualKeyboard
```

2. 在可执行项目文件（`.csproj`）中包含您的 Avalonia 许可证密钥。您可以从 [Avalonia 门户](https://portal.avaloniaui.net) 获取您的许可证密钥。

```xml
<ItemGroup>
  <AvaloniaUILicenseKey Include="您的许可证密钥" />
</ItemGroup>
```

:::tip
对于多项目解决方案，您可以将许可证密钥存储在[环境变量](https://learn.microsoft.com/en-us/visualstudio/msbuild/how-to-use-environment-variables-in-a-build)或[共享属性文件](https://learn.microsoft.com/en-us/visualstudio/msbuild/customize-by-directory?view=vs-2022#directorybuildprops-example)中，以避免重复。
:::

3. 在您的 `App.axaml` 文件中通过 `StyleInclude` 引用 `VirtualKeyboard` 的流畅主题。这将添加正确设置虚拟键盘样式所需的资源。

```xml
<Application.Styles>
   <StyleInclude Source="avares://Avalonia.Controls.VirtualKeyboard/Themes/Fluent.axaml"/>
   <!-- 其他样式 -->
</Application.Styles>
```

有关安装 Avalonia Pro 控件的更多信息，请参阅[安装 Avalonia Pro](/tools/installing-avalonia-pro)。

## 基本用法

### 使用 VirtualKeyboardScope（推荐）

向应用程序添加虚拟键盘的最简单方法是使用 `VirtualKeyboardScope` 控件，当文本输入控件获得焦点时，它会自动显示和隐藏键盘：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d" d:DesignWidth="600" d:DesignHeight="600"
        Width="600" Height="600"
        x:Class="YourNamespace.YourWindow"
        Title="虚拟键盘示例">
    <VirtualKeyboardScope InputMethods="en-US:kbd:standard, de:kbd:standard, ja:ime:kana">
        <StackPanel>
            <TextBlock>Hello world!</TextBlock>
            <TextBox PlaceholderText="在此输入"/>
        </StackPanel>
    </VirtualKeyboardScope>
</Window>
```

### 直接使用 VirtualKeyboard

为了获得更多控制，您可以直接使用 `VirtualKeyboard` 控件并指定目标输入元素。

无论当前输入焦点如何，输入都将被定向到指定的 `Target` 元素。

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="虚拟键盘示例"
        Width="400" Height="600">
    <StackPanel>
        <TextBox x:Name="InputBox" Width="300" Margin="10"/>
        <VirtualKeyboard Target="{Binding ElementName=InputBox}"
                           InputMethods="en-US:kbd:standard, de:kbd:standard, ja:ime:kana"
                           Margin="10"/>
    </StackPanel>
</Window>
```

## 管理输入方法

虚拟键盘支持多种输入方法和键盘布局。以下是指定要包含哪些输入方法的说明：

### XAML

```xml
<VirtualKeyboardScope InputMethods="en-US:kbd:standard, de:kbd:standard, ja:ime:kana">
    <!-- 您的内容放在这里 -->
</VirtualKeyboardScope>
```

### C#

```csharp
// 获取特定语言的所有可用输入方法
var inputMethods = new List<VirtualKeyboardInputMethod>
{
    VirtualKeyboardInputMethod.GetInputMethodsForLanguage("en-US").First(),
    VirtualKeyboardInputMethod.GetInputMethodsForLanguage("de").First(),
    VirtualKeyboardInputMethod.GetInputMethodsForLanguage("ja").First()
};

// 分配给 VirtualKeyboardScope
myKeyboardScope.InputMethods = inputMethods;
```

### 检索输入方法

```csharp
// 获取所有支持的语言
var languages = VirtualKeyboardInputMethod.GetSupportedLanguages();

// 获取特定语言的输入方法
var englishInputMethods = VirtualKeyboardInputMethod.GetInputMethodsForLanguage("en-US");

// 通过 ID 获取特定输入方法
var japaneseKana = VirtualKeyboardInputMethod.GetInputMethodById("ja:ime:kana");
```

## 输入方法标识符

请参阅[虚拟键盘控件](/controls/input/text-input/virtualkeyboard/virtualkeyboard-control#input-methods)页面，了解支持的输入方法及其标识符的表格。

## RIME 输入法引擎

虚拟键盘支持 [RIME](https://rime.im/) 输入法引擎用于中文文本输入。RIME 支持作为单独的插件包提供。

请参阅 [`VirtualKeyboard` 控件参考](/controls/input/text-input/virtualkeyboard/virtualkeyboard-control#rime-input-method-engine)了解安装和配置说明。

## 文本输入选项

您可以使用 `TextInputOptions` 附加属性自定义虚拟键盘在不同输入字段中的行为：

```xml
<TextBox TextInputOptions.ContentType="Email"
         TextInputOptions.ReturnKeyType="Search" />
```

可用的 `ContentType` 值：
- `Normal`
- `Email`
- `Url`
- `Digits`

可用的 `ReturnKeyType` 值：
- `Default`
- `Done`
- `Go`
- `Next`
- `Previous`
- `Return`
- `Search`
- `Send`

## 另请参阅

- [VirtualKeyboard 控件](/controls/input/text-input/virtualkeyboard)
- [VirtualKeyboardScope 控件](/controls/input/text-input/virtualkeyboard/virtualkeyboardscope)
