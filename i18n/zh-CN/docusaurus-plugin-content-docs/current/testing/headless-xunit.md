---
id: headless-xunit
title: 结合 XUnit 的无头测试
description: 了解如何使用 XUnit 测试框架为 Avalonia 应用配置并运行无头 UI 测试。
doc-type: guide
---

## 准备工作

本页假定你已经创建好了 XUnit 项目。
如果还没有，请先参考 XUnit 的 “Getting Started” 与 “Installation” 文档：https://xunit.net/docs/getting-started/netfx/visual-studio。

## 安装包

除了 XUnit 本身的包之外，你还需要再安装两个包：
- [Avalonia.Headless.XUnit](https://www.nuget.org/packages/Avalonia.Headless.XUnit)，它也会一并引入 Avalonia。
- [Avalonia.Themes.Fluent](https://www.nuget.org/packages/Avalonia.Themes.Fluent)，因为即使是无头控件也需要主题。

:::tip
无头平台并不强制要求某个特定主题，你也可以把 FluentTheme 替换成其他主题。
:::

## 配置应用程序

和其他 Avalonia 应用一样，你需要创建一个 `Application` 实例并应用主题。使用无头平台时，这部分配置与普通 Avalonia 应用并没有太大区别，大多数代码都可以复用。

```xml title=App.axaml
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="Tests.App">
  <Application.Styles>
    <FluentTheme />
  </Application.Styles>
</Application>
```

对应的代码如下：

```csharp title=App.axaml.cs
using Avalonia;
using Avalonia.Headless;

public class App : Application
{
    public override void Initialize()
    {
        AvaloniaXamlLoader.Load(this);
    }
}
```

:::note
通常情况下，`BuildAvaloniaApp` 方法会定义在 Program.cs 中，但 NUnit/XUnit 测试项目中通常没有这个入口，因此这里把它定义在 `App` 文件中。
:::

## 初始化 XUnit 测试

`[AvaloniaTestApplication]` 特性会把当前项目中的测试与指定应用关联起来。它每个项目只需要定义一次，可以放在任意文件中。

```csharp
[assembly: AvaloniaTestApplication(typeof(TestAppBuilder))]

public class TestAppBuilder
{
    public static AppBuilder BuildAvaloniaApp() => AppBuilder.Configure<App>()
        .UseHeadless(new AvaloniaHeadlessPlatformOptions());
}
```

## 测试隔离级别

默认情况下，每个测试都会重新创建 `Application` 和 `Dispatcher`（即 `PerTest` 隔离）。对于大型测试集，这可能会比较慢。如果想在同一个程序集中的所有测试之间复用一个 `Application` 实例，请添加 `[AvaloniaTestIsolation]` 特性：

```csharp
[assembly: AvaloniaTestApplication(typeof(TestAppBuilder))]
[assembly: AvaloniaTestIsolation(AvaloniaTestIsolationLevel.PerAssembly)]
```

| 级别 | 行为 |
|---|---|
| `PerTest` | 为每个测试重新创建 `Application` 和 `Dispatcher`（默认）。测试完全隔离。 |
| `PerAssembly` | 为程序集中的所有测试复用同一个 `Application` 和 `Dispatcher`。速度更快，但测试之间会共享状态。 |

:::caution
在 `PerAssembly` 隔离模式下，测试会共享 `Application` 状态。请在测试之间清理全局状态（样式、资源、静态属性等），以避免互相干扰。此模式不支持并发执行测试。
:::

## 示例

```csharp
[AvaloniaFact]
public void Should_Type_Text_Into_TextBox()
{
    // 创建控件：
    var textBox = new TextBox();
    var window = new Window { Content = textBox };

    // 打开窗口：
    window.Show();

    // 聚焦文本框：
    textBox.Focus();

    // 模拟文本输入：
    window.KeyTextInput("Hello World");

    // 断言：
    Assert.Equal("Hello World", textBox.Text);
}
```

不要使用常规的 `[Fact]`，而应使用 `[AvaloniaFact]`，因为它会负责初始化 UI 线程。类似地，`[Theory]` 也有对应的 `[AvaloniaTheory]` 可用。

## 另请参阅

- [适用于 XUnit 的可测试示例应用](https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/Testing/TestableApp.Headless.XUnit)
- [无头测试平台](/docs/testing/setting-up-the-headless-platform)
- [结合 NUnit 的无头测试](/docs/testing/headless-nunit)
- [使用 Appium 做 UI 测试](/docs/testing/ui-testing-with-appium)
