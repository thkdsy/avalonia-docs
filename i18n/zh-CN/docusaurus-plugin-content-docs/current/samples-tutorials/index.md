---
id: index
title: 示例与教程
description: 浏览 Avalonia 教程、示例应用和快速指南，加快你的学习进度。
doc-type: landing
hide_table_of_contents: true
---

import DocsCard from '@site/src/components/global/DocsCard';
import DocsCards from '@site/src/components/global/DocsCards';

<head>
  <title>Avalonia 文档：示例与教程</title>
  <meta
    name="description"
    content="浏览 Avalonia 教程、示例应用和快速指南，加快你的学习进度。"
  />
  <style>{`
    :root {
      --doc-item-container-width: 60rem;
    }
  `}</style>
</head>

浏览教程、示例应用和快速指南，加快你的 Avalonia 学习进度。

## 教程

<DocsCards>

  <DocsCard header="入门教程" href="/docs/get-started/starter-tutorial">
    <p>一步一步构建你的第一个 Avalonia 应用，从控件和布局到事件与数据转换。</p>
  </DocsCard>

  <DocsCard header="待办列表应用" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/CompleteApps/SimpleToDoList">
    <p>使用 MVVM 模式、绑定、命令、样式和基础 I/O 构建一个待办列表应用。</p>
  </DocsCard>

  <DocsCard header="音乐商店应用" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/CompleteApps/Avalonia.MusicStore">
    <p>构建一个带有对话框、图像、集合和数据持久化功能的图形化音乐商店应用。</p>
  </DocsCard>

</DocsCards>

## MVVM 示例

<DocsCards>

  <DocsCard header="基础 MVVM" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/MVVM/BasicMvvmSample">
    <p>使用 MVVM 模式接收并处理用户输入的文本。</p>
  </DocsCard>

  <DocsCard header="绑定与转换器" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/BindingsAndConverters">
    <p>将日期转换为字符串值，并据此计算一个人的年龄。</p>
  </DocsCard>

  <DocsCard header="值转换器" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/MVVM/ValueConversionSample">
    <p>在绑定中加入转换器，为视图计算新的值。</p>
  </DocsCard>

  <DocsCard header="命令" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/MVVM/CommandSample">
    <p>学习如何通过命令从用户界面调用 ViewModel 中的方法。</p>
  </DocsCard>

  <DocsCard header="数据验证" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/MVVM/ValidationSample">
    <p>验证属性值，并在值无效时显示错误信息。</p>
  </DocsCard>

  <DocsCard header="对话框管理器" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/ViewInteraction/DialogManagerSample">
    <p>创建一个对话框管理服务，帮助你在应用中显示对话框。</p>
  </DocsCard>

</DocsCards>

## 数据模板示例

<DocsCards>

  <DocsCard header="基础数据模板" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/DataTemplates/BasicDataTemplateSample">
    <p>使用 DataTemplate 控制数据的显示方式。</p>
  </DocsCard>

  <DocsCard header="FuncDataTemplate" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/DataTemplates/FuncDataTemplateSample">
    <p>在代码中使用 FuncDataTemplate 创建高级数据模板。</p>
  </DocsCard>

  <DocsCard header="IDataTemplate 接口" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/DataTemplates/IDataTemplateSample">
    <p>在你自己的类中实现 IDataTemplate，以完全掌控数据模板行为。</p>
  </DocsCard>

</DocsCards>

## 样式与绘图示例

<DocsCards>

  <DocsCard header="按钮自定义" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/ButtonCustomize">
    <p>通过创建可复用样式来自定义按钮外观。</p>
  </DocsCard>

  <DocsCard header="创建列表" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/MakingLists">
    <p>使用绑定和 ListBox 控件创建数据列表。</p>
  </DocsCard>

  <DocsCard header="原生菜单" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/NativeMenuOps">
    <p>在 macOS 和 Linux 上于 Avalonia 应用中使用原生菜单。</p>
  </DocsCard>

  <DocsCard header="启动画面" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/SplashScreen">
    <p>构建一个会在 MainWindow 之前显示的自定义启动画面。</p>
  </DocsCard>

  <DocsCard header="矩形画板" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/Drawing/RectPainter">
    <p>创建一个可与鼠标交互的自定义渲染控件，构成简单的绘图应用。</p>
  </DocsCard>

  <DocsCard header="加载图像" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/LoadingImages">
    <p>了解如何通过 XAML、绑定和网络来源加载图像。</p>
  </DocsCard>

  <DocsCard header="使用字体" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/GoogleFonts">
    <p>在应用中使用自定义字体，例如 Google Fonts。</p>
  </DocsCard>

  <DocsCard header="Battle City" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/Drawing/BattleCity">
    <p>一个使用 Avalonia 编写、且无需手写渲染代码的 2D 游戏示例。</p>
  </DocsCard>

</DocsCards>

## 自定义控件示例

<DocsCards>

  <DocsCard header="评分控件" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/CustomControls/RatingControlSample">
    <p>创建一个评分控件，让用户通过点击多颗星星进行打分。</p>
  </DocsCard>

  <DocsCard header="雪花控件" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/CustomControls/SnowflakesControlSample">
    <p>创建一个重写 OnRender 的自定义控件，以使用更高级的渲染方式。</p>
  </DocsCard>

</DocsCards>

## 测试示例

<DocsCards>

  <DocsCard header="使用 XUnit 进行无头测试" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/Testing/TestableApp.Headless.XUnit">
    <p>借助 Avalonia 的无头平台和 XUnit，在没有可见图形界面的情况下测试你的应用。</p>
  </DocsCard>

  <DocsCard header="使用 NUnit 进行无头测试" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/Testing/TestableApp.Headless.NUnit">
    <p>借助 Avalonia 的无头平台和 NUnit，在没有可见图形界面的情况下测试你的应用。</p>
  </DocsCard>

  <DocsCard header="使用 Appium 测试" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/Testing/TestableApp.Appium">
    <p>为按钮点击、文本输入、界面导航等 UI 交互编写自动化测试。</p>
  </DocsCard>

</DocsCards>

## 其他示例

<DocsCards>

  <DocsCard header="剪贴板操作" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/ClipboardOps">
    <p>与设备剪贴板交互，实现文本复制和粘贴。</p>
  </DocsCard>

  <DocsCard header="拖放操作" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/DragDropOps">
    <p>在 Avalonia 应用中实现拖放功能。</p>
  </DocsCard>

  <DocsCard header="原生文件操作" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/FileOps">
    <p>使用原生“另存为”和“打开文件”对话框。</p>
  </DocsCard>

  <DocsCard header="IoC 文件操作" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/IoCFileOps">
    <p>结合控制反转（IoC）使用原生“另存为”和“打开文件”对话框。</p>
  </DocsCard>

  <DocsCard header="本地化" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/Localization">
    <p>演示如何为 Avalonia 应用做本地化处理。</p>
  </DocsCard>

  <DocsCard header="基础视图定位器" href="https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/Routing/BasicViewLocatorSample">
    <p>使用视图定位器切换 UI 内容。</p>
  </DocsCard>

  <DocsCard header="原生 AOT" href="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/NativeAot">
    <p>将你的应用配置为使用原生预先编译（AOT）构建。</p>
  </DocsCard>

</DocsCards>

## 另请参阅

- [快速开始](/docs/get-started/create-your-first-project)：创建你的第一个 Avalonia 应用。
- [控件参考](/controls)：完整的 Avalonia 控件文档。
