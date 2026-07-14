---
id: ui-testing-with-appium
title: 使用 Appium 做 UI 测试
---

Appium 是一个开源自动化框架，它会通过应用的无障碍树来驱动你的程序，模拟真实用户交互，例如点击按钮、输入文本以及验证控件状态。与 [无头测试](/docs/testing/setting-up-the-headless-platform) 不同，后者是在没有可见窗口的情况下以编程方式模拟输入，而 Appium 测试会在真实窗口中启动你编译后的应用，并以和用户相同的方式与其交互。

因此，Appium 非常适合用于端到端验证、无障碍检查以及平台特定行为测试。Avalonia 自身也在内部使用 Appium 来测试框架在 Windows 和 macOS 上的行为。

## 何时使用 Appium，何时使用无头测试

| 对比项 | 无头测试 | Appium |
|---|---|---|
| 速度 | 快（进程内、无 GUI） | 较慢（会启动真实应用） |
| 范围 | 单元测试与组件测试 | 端到端测试与集成测试 |
| 平台行为 | 模拟 | 真实（原生窗口系统、菜单、焦点） |
| 无障碍 | 不测试 | 会测试（通过无障碍树驱动） |
| CI/CD | 几乎可在任意环境运行 | 需要显示设备（Linux 上可使用虚拟显示） |

如果你需要快速验证控件逻辑和数据绑定，请使用无头测试；如果你需要验证整个应用是否正常工作，包括原生平台集成，请使用 Appium 测试。

## 前置条件

### Windows

安装 [WinAppDriver](https://github.com/microsoft/WinAppDriver/releases)。WinAppDriver 会在 Windows 上充当 Appium 服务器，并要求系统为 Windows 10 或更高版本。同时请在 Windows 设置中启用 **Developer Mode**（开发者模式）。

### macOS

安装 Appium 和 Mac2 驱动：

```bash
npm install -g appium
appium driver install mac2
```

你还需要为运行测试所用的终端或 IDE 授予无障碍权限。进入 **System Settings > Privacy & Security > Accessibility**，然后把你的终端应用加入其中。

## 项目配置

创建一个新的 xUnit 测试项目，并安装 Appium 客户端：

```bash
dotnet new xunit -n MyApp.UITests
cd MyApp.UITests
dotnet add package Appium.WebDriver
```

## 创建测试夹具

这个夹具负责管理 Appium driver 会话。它会启动应用、建立连接，并在测试结束后完成清理。

```csharp
using OpenQA.Selenium.Appium;
using OpenQA.Selenium.Appium.Windows;
using Xunit;

public class AppFixture : IDisposable
{
    public AppiumDriver Session { get; }

    public AppFixture()
    {
        if (OperatingSystem.IsWindows())
        {
            var options = new AppiumOptions();
            options.AddAdditionalAppiumOption("app", @"path\to\your\MyApp.exe");
            options.AddAdditionalAppiumOption("platformName", "Windows");
            options.AddAdditionalAppiumOption("deviceName", "WindowsPC");
            Session = new WindowsDriver(new Uri("http://127.0.0.1:4723"), options);
        }
        else if (OperatingSystem.IsMacOS())
        {
            var options = new AppiumOptions();
            options.AddAdditionalAppiumOption("platformName", "mac");
            options.AddAdditionalAppiumOption("automationName", "mac2");
            options.AddAdditionalAppiumOption("bundleId", "com.mycompany.myapp");
            Session = new AppiumDriver(new Uri("http://127.0.0.1:4723/wd/hub"), options);
        }
        else
        {
            throw new PlatformNotSupportedException();
        }
    }

    public void Dispose()
    {
        try { Session?.Quit(); } catch { }
    }
}

[CollectionDefinition("Default")]
public class DefaultCollection : ICollectionFixture<AppFixture> { }
```

:::tip
在 macOS 上，应使用 `bundleId` 而不是文件路径来标识应用。请先把应用构建为 `.app` 包。
:::

## 编写测试

测试会使用 `FindElementByAccessibilityId` 来定位控件。之所以可行，是因为 Avalonia 会通过平台无障碍 API 暴露 `AutomationProperties.AutomationId` 的值（或者控件的 `Name`）。

### 为控件设置 AutomationId

为控件设置 `AutomationId`，这样测试才能稳定地找到它们：

```xml
<Button AutomationProperties.AutomationId="SubmitButton" Content="Submit" />
<TextBox AutomationProperties.AutomationId="NameInput" />
<CheckBox AutomationProperties.AutomationId="AgreeCheckBox" Content="I agree" />
```

### 基础测试

```csharp
using OpenQA.Selenium.Appium;
using Xunit;

[Collection("Default")]
public class ButtonTests
{
    private readonly AppiumDriver _session;

    public ButtonTests(AppFixture fixture)
    {
        _session = fixture.Session;
    }

    [Fact]
    public void Click_Button_Updates_Text()
    {
        var button = _session.FindElement(MobileBy.AccessibilityId("SubmitButton"));
        var output = _session.FindElement(MobileBy.AccessibilityId("OutputText"));

        button.Click();

        Assert.Equal("Submitted", output.Text);
    }
}
```

### 测试复选框状态

```csharp
[Fact]
public void CheckBox_Toggles_On_Click()
{
    var checkBox = _session.FindElement(MobileBy.AccessibilityId("AgreeCheckBox"));

    // 通过无障碍属性读取初始状态
    var initialState = checkBox.GetAttribute("Toggle.ToggleState");
    Assert.Equal("0", initialState); // 0 = unchecked

    checkBox.Click();

    var newState = checkBox.GetAttribute("Toggle.ToggleState");
    Assert.Equal("1", newState); // 1 = checked
}
```

### 测试文本输入

```csharp
[Fact]
public void TextBox_Accepts_Input()
{
    var textBox = _session.FindElement(MobileBy.AccessibilityId("NameInput"));

    textBox.Clear();
    textBox.SendKeys("Avalonia");

    Assert.Equal("Avalonia", textBox.Text);
}
```

## 平台特定测试

有些测试只在特定平台上才有意义（例如 macOS 上的原生菜单测试）。你可以创建一个自定义特性，在不支持的平台上跳过这些测试：

```csharp
using System.Runtime.InteropServices;
using Xunit;

[Flags]
public enum TestPlatforms
{
    Windows = 0x01,
    MacOS = 0x02,
    Linux = 0x04,
    All = Windows | MacOS | Linux
}

public sealed class PlatformFactAttribute : FactAttribute
{
    public PlatformFactAttribute(TestPlatforms platforms)
    {
        if (!IsSupported(platforms))
        {
            Skip = $"Test is not supported on {RuntimeInformation.OSDescription}";
        }
    }

    private static bool IsSupported(TestPlatforms platforms)
    {
        if (OperatingSystem.IsWindows()) return platforms.HasFlag(TestPlatforms.Windows);
        if (OperatingSystem.IsMacOS()) return platforms.HasFlag(TestPlatforms.MacOS);
        if (OperatingSystem.IsLinux()) return platforms.HasFlag(TestPlatforms.Linux);
        return false;
    }
}
```

把它用于只针对特定平台的测试：

```csharp
[PlatformFact(TestPlatforms.MacOS)]
public void Native_Menu_Shows_App_Name()
{
    // 仅适用于 macOS 的测试
}
```

## 跨平台辅助方法

WinAppDriver 和 macOS 驱动在属性名称和元素查找方式上可能存在差异。借助辅助方法可以让测试代码保持整洁：

```csharp
public static class ElementExtensions
{
    public static string GetName(this AppiumElement element)
    {
        if (OperatingSystem.IsWindows())
            return element.GetAttribute("Name");
        return element.GetAttribute("title");
    }

    public static bool? GetIsChecked(this AppiumElement element)
    {
        var value = element.GetAttribute("Toggle.ToggleState")
            ?? element.GetAttribute("value");

        return value switch
        {
            "0" => false,
            "1" => true,
            _ => null // 不确定状态
        };
    }
}
```

## 运行测试

### Windows

先启动 WinAppDriver（它会作为本地服务器运行）：

```
"C:\Program Files (x86)\Windows Application Driver\WinAppDriver.exe"
```

然后运行测试：

```bash
dotnet test
```

### macOS

启动 Appium 服务器：

```bash
appium
```

然后在另一个终端中运行测试：

```bash
dotnet test
```

## CI/CD 注意事项

- **Windows**：在测试开始之前必须先启动 WinAppDriver。在 CI 中，请增加一个前置步骤来启动它。
- **macOS**：必须安装 Appium 和 mac2 驱动，并为 CI 代理授予无障碍权限。
- **Linux**：Appium 目前没有稳定的 Linux 桌面驱动。对于 Linux CI，请改用 [无头测试](/docs/testing/setting-up-the-headless-platform)。

## 另请参阅

- [结合 XUnit 的无头测试](/docs/testing/headless-xunit)：快速、进程内的测试方式。
- [结合 NUnit 的无头测试](/docs/testing/headless-nunit)：无头测试的 NUnit 集成。
- [无头平台配置](/docs/testing/setting-up-the-headless-platform)：输入模拟与帧捕获。
- [Avalonia 自身的 Appium 测试](https://github.com/AvaloniaUI/Avalonia/tree/master/tests/Avalonia.IntegrationTests.Appium)：Avalonia 内部使用的测试套件。
- [Appium 文档](https://appium.io/docs/en/latest/)：官方 Appium 指南。
