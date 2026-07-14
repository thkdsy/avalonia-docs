---
id: headless-testing
title: 无头测试
---

## 概述

在 WPF 中，单元测试一直是一个复杂的场景。最常见的解决方案是运行基于自动化的重型端到端（e2e）测试套件。这会使其更慢，并且将其限制为仅能在 Windows 上测试。

与其进行自动化测试，不如先尝试无头测试，因为它更快且更便于移植。
由于 XPF 基于与 Avalonia 相同的核心，无头测试也适用于 WPF 应用。

:::tip
提供了一个完整的包含无头测试的 [CalculatorDemo 示例](https://github.com/AvaloniaUIOU/CalculatorDemo)。如有需要，请联系支持团队以获取该仓库的访问权限。
:::

:::note
有关 Headless 平台和 Avalonia 扩展的更详细文档，请参阅 [使用 XUnit 进行无头测试](/docs/testing/headless-xunit) 和 [使用 NUnit 进行无头测试](/docs/testing/headless-nunit)。理解无头测试在 Avalonia 中的工作方式也有助于理解 XPF/WPF。
:::

## 配置测试项目

`XUnit`、`NUnit` 和 `MSTest` 都受 XPF/Avalonia 无头测试支持。
需要在测试项目中包含集成 nuget 包：

```xml
<ItemGroup>
    <PackageReference Include="Avalonia.Headless.XUnit" Version="$(XpfAvaloniaVersion)" />
    or
    <PackageReference Include="Avalonia.Headless.NUnit" Version="$(XpfAvaloniaVersion)" />
</ItemGroup>
```

`$(XpfAvaloniaVersion)` 是 `Xpf.Sdk` 中预定义的常量，也需要在测试项目中设置。如果你手动指定最新的 `PackageReference` 版本，也可以省略它。

`AvaloniaUI.Xpf.LicenseKey` 也需要用于测试项目，以通过运行时验证。如果你需要了解更多关于如何获取此密钥的信息，请参阅 [入门](/xpf/getting-started.md) 页面。

```xml
<ItemGroup>
    <RuntimeHostConfigurationOption Include="AvaloniaUI.Xpf.LicenseKey" Value="--在此插入你的密钥--"/>
</ItemGroup>
```

## （可选）配置测试应用程序

与 Avalonia 无头测试类似，你可以在项目中配置跨平台的 `AppBuilder` 来使用。
如果未定义，无头平台将使用默认参数，这可能会限制你的 XPF 测试体验。
注意，如果你已经为 XPF 应用重写了 AppBuilder（参见 [自定义初始化](/xpf/configuration/customizing-initialization) 文档），你可以复用相同的初始化代码，但在链的末尾添加 `.UseHeadless()`。

```csharp
[assembly: AvaloniaTestApplication(typeof(TestAppBuilder))]

public class TestAppBuilder
{
    // XPF specific: add .WithAvaloniaXpf() and use DefaultXpfAvaloniaApplication which has preconfigured default themes.
    public static AppBuilder BuildAvaloniaApp() => AppBuilder.Configure<DefaultXpfAvaloniaApplication>()
        .WithAvaloniaXpf()
        .UseSkia()
        .UseHeadless(new AvaloniaHeadlessPlatformOptions
        {
            // Set to false to enable capturing rendered frames
            UseHeadlessDrawing = false
        });
}
```

## 编写测试

你的测试运行在与 XPF 应用相同的进程中，这使得发送任何事件以及访问应用的任意输出都更容易。
与普通 WPF 应用一样，一切都必须从一个 Window 开始，它可以在同一个测试方法中创建，或者从设置方法中复用（NUnit 中的 `[SetUp]` 方法或 XUnit 中的构造函数）。

:::note
NUnit 的 [Test] 和 [Theory] 需要替换为 [AvaloniaTest] [AvaloniaTheory]，
而 XUnit 的 [Fact] 则需要替换为 [AvaloniaFact]。
:::

一个非常基础的 NUnit 测试如下所示：

```csharp
[AvaloniaTest]
public void Should_Be_Able_To_Raise_Event()
{
    var window = new MainWindow();
    window.Show();
    var button = window.ClickingButton;

    Assert.That(button.Content, Is.EqualTo("Click me"));

    button.RaiseEvent(new RoutedEventArgs(ButtonBase.ClickEvent, button));

    Assert.That(button.Content, Is.EqualTo("Click count: 1"));
}
```

按钮逻辑如下：

```csharp
private int _clickCount = 0;
private void ClickingButton_OnClick(object sender, RoutedEventArgs e)
{
    ClickingButton.Content = "Click count: " + (++_clickCount).ToString();
}
```

:::tip
要从测试项目访问 `ClickingButton`，你需要在 XAML 中将控件设置为 `x:FieldModifier="public"`，或者在主项目中添加 `[assembly: InternalsVisibleTo("YourTestProject")]` 特性。
:::

## 访问 Avalonia 无头扩展

Avalonia 提供了用于模拟点击和键盘输入的无头扩展，无需触发伪造的 WPF 事件。

这些扩展仅可用于 Avalonia Window，不能用于 WPF Window。
不过幸运的是，在无头测试中可以获取 Avalonia Window：

```csharp
// Get Avalonia window and send text input to currently focused control.
var avWindow = XpfWpfAbstraction.GetAvaloniaWindowForWindow(xpfWindow);
avWindow.KeyTextInput("Hello");
```

有关与 Avalonia 集成的更多细节，请参阅 [Avalonia 互操作](/xpf/interop/embedding-avalonia-in-xpf#accessing-avalonia-features)。

## （可选）在 WPF 应用/项目中使用 XPF 无头测试

只有启动项目需要使用 Xpf.Sdk 测试。
这也意味着你可以拥有一个普通的 "net8.0-windows" 项目来存放控件，并在 XPF 无头项目中引用它们。

如果你有一个共享控件库并想对其进行无头测试，这会很有用；或者如果你有一个普通的 Windows WPF 应用，并且需要无头测试而不想完全使用 XPF，这也同样适用。

所有使用步骤都相同，但你还需要将测试项目的 TargetFramework 设置为 `net8.0-windows`，并将 `EnableWindowsTargeting` 设置为 true（仅当你需要在 Linux/macOS 机器上运行它时）。

## MSTest 支持

对于 MSTest 项目，配置类似，但需要额外设置：

1. 在测试项目的 `.csproj` 中将 `DisableAutomaticXpfInit` 设为 `true`：
   ```xml
   <PropertyGroup>
       <DisableAutomaticXpfInit>true</DisableAutomaticXpfInit>
   </PropertyGroup>
   ```

2. 配置无头 AppBuilder，并使用 `[AvaloniaTestMethod]` 代替 `[TestMethod]`：
   ```csharp
   [assembly: AvaloniaTestApplication(typeof(TestAppBuilder))]

   public class TestAppBuilder
   {
       public static AppBuilder BuildAvaloniaApp() => AppBuilder
           .Configure<DefaultXpfAvaloniaApplication>()
           .WithAvaloniaXpf()
           .UseSkia()
           .UseHeadless(new AvaloniaHeadlessPlatformOptions
           {
               UseHeadlessDrawing = false
           });
   }
   ```

## 测试隔离

如果你遇到不稳定的测试（例如 `TaskScheduler` 错误，或测试之间状态不一致），请按程序集配置测试隔离：

```csharp
[assembly: AvaloniaTestApplication(typeof(TestAppBuilder), AvaloniaTestIsolationLevel.PerAssembly)]
```

这可确保 Avalonia 运行时每个测试程序集只初始化一次，而不是每个测试都初始化，从而避免测试清理与初始化之间的竞态条件。

## 在 CI 中运行测试

在 Linux 上的 CI 环境中运行 XPF 无头测试时：

- 确保已在测试项目中配置许可证密钥（参见 [入门](/xpf/getting-started#step-4-add-your-licence-key)）
- 使用无头模式时不需要显示服务器
- 如果出现 `XOpenDisplay failed` 错误，请确认 `DisableAutomaticXpfInit` 已设为 `true`，并且无头 AppBuilder 已正确配置

## 另见

- [使用 XUnit 进行无头测试](/docs/testing/headless-xunit)
- [使用 NUnit 进行无头测试](/docs/testing/headless-nunit)
- [设置无头平台](/docs/testing/setting-up-the-headless-platform)