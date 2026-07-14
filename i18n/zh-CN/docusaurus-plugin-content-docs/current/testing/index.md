---
id: index
title: 测试
---

Avalonia 支持多种测试策略，每一种都适用于不同目标。你可以将它们组合起来，构建一个分层测试体系，覆盖视图模型逻辑、控件行为、视觉输出以及端到端用户流程。

## 测试策略

| 策略 | 测试内容 | 速度 | 是否需要 UI |
|---|---|---|---|
| **单元测试** | 视图模型逻辑、服务、转换器 | 快 | 否 |
| **无头测试** | 控件、布局、数据绑定、输入 | 快 | 否（进程内） |
| **视觉回归测试** | 渲染出的像素结果 | 中 | 否（无头 + Skia） |
| **Appium UI 测试** | 完整应用、平台集成、无障碍 | 慢 | 是（真实窗口） |

### 单元测试

标准的 .NET 单元测试适用于那些不依赖 Avalonia 控件的代码。视图模型、值转换器、服务和业务逻辑都可以直接使用 xUnit、NUnit 或 MSTest 测试，而无需任何 Avalonia 专属配置。

```csharp
[Fact]
public void ViewModel_Increments_Count()
{
    var vm = new MainViewModel();
    vm.IncrementCommand.Execute(null);
    Assert.Equal(1, vm.Count);
}
```

### 无头测试

[无头平台](/docs/testing/setting-up-the-headless-platform) 会在内存中运行 Avalonia 完整的控件树、布局引擎、样式系统和数据绑定，而无需真正打开窗口。你可以通过辅助方法模拟键盘和鼠标输入。这非常适合测试控件行为、数据绑定、焦点管理以及命令执行。

Avalonia 还为 [xUnit](/docs/testing/headless-xunit) 和 [NUnit](/docs/testing/headless-nunit) 提供了集成包，可自动完成相关配置。

```csharp
[AvaloniaFact]
public void TextBox_Accepts_Input()
{
    var textBox = new TextBox();
    var window = new Window { Content = textBox };
    window.Show();

    textBox.Focus();
    window.KeyTextInput("Hello");

    Assert.Equal("Hello", textBox.Text);
}
```

### 视觉回归测试

启用无头模式下的 Skia 渲染器后，你可以把渲染帧捕获成位图，并与基线图像进行比较。这能帮助你发现控件渲染、主题和布局中那些非预期的视觉变化。有关具体配置方式，请参阅 [捕获最后渲染帧](/docs/testing/setting-up-the-headless-platform#visual-regression-testing)。

### Appium UI 测试

[Appium](/docs/testing/ui-testing-with-appium) 会在真实窗口中启动你编译好的应用，并通过平台无障碍树来驱动它。这种方式会测试完整技术栈：原生窗口系统、菜单、焦点、平台特定行为以及无障碍支持。Appium 测试更慢，但能验证你的应用是否真的像用户体验到的那样正常工作。

Avalonia 自身也在内部使用 Appium，来测试框架在 Windows 和 macOS 上的行为。

## 选择合适的方法

- **先从单元测试开始**，覆盖视图模型和服务。这类测试速度快、稳定，也不需要特殊配置。
- **补充无头测试**，用于那些依赖 Avalonia 属性系统、布局或输入处理的控件行为。
- **如果你的应用有自定义控件或主题，且像素级正确性很重要**，就加入视觉回归测试。
- **对关键用户流程、平台特性和无障碍验证**，再补充 Appium 测试。

## 另请参阅

- [无头测试平台](/docs/testing/setting-up-the-headless-platform)：输入模拟、帧捕获和异步处理。
- [结合 XUnit 的无头测试](/docs/testing/headless-xunit)：XUnit 集成配置。
- [结合 NUnit 的无头测试](/docs/testing/headless-nunit)：NUnit 集成配置。
- [使用 Appium 做 UI 测试](/docs/testing/ui-testing-with-appium)：基于真实窗口的端到端测试。
