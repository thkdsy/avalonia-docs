---
id: architecture
title: Avalonia 架构
description: 了解控件如何进行测量、排列、渲染，以及如何连接到底层平台后端。
doc-type: explanation
---

本页介绍 Avalonia 的内部架构：控件如何被测量、排列和渲染，合成器如何调度帧，以及平台后端如何连接到渲染管线。

## 高层概览

Avalonia 采用分层构建方式。每一层只依赖其下方的层：

| 层级 | 职责 |
|---|---|
| **Controls** | 面向用户的控件（Button、TextBox、DataGrid）、数据绑定、模板、样式 |
| **Layout** | 测量/排列系统、面板逻辑、滚动 |
| **Visual** | 视觉树、渲染变换、不透明度、裁剪 |
| **Rendering** | 绘图原语（画刷、几何、文本）、场景图、合成 |
| **Platform Abstraction** | 窗口系统、输入、剪贴板、文件对话框、GPU 上下文 |
| **Platform Backends** | Win32、Cocoa、X11/Wayland、Android、iOS、Browser（WASM） |

应用程序代码主要与 Controls 和 Layout 两层交互。渲染层和平台层则在底层幕后运行。

## 渲染管线

Avalonia 使用保留模式渲染模型。控件不会直接自行绘制，而是通过模板声明自己的视觉结构，再由框架构建场景图，并在每一帧进行渲染。

### 帧生命周期

每一帧都会按照如下顺序执行：

1. **输入处理**：平台后端传递指针、键盘和触摸事件。事件会在视觉树中进行隧道和冒泡传播。
2. **属性变更**：输入处理器和定时器触发属性变化，这些变化可能使布局或渲染失效。
3. **布局阶段**：需要重新测量的控件会先执行 `Measure`，再执行 `Arrange`。布局从根节点开始向下遍历整棵树，为每个控件确定最终尺寸和位置。
4. **渲染阶段**：视觉已失效的控件会通过 `Render(DrawingContext)` 重建自身对应的场景图部分。框架会将新旧场景进行差异比较。
5. **合成阶段**：合成器接收更新后的场景图，并将绘制调用提交给 GPU 后端（Skia 或平台原生合成器）。

### 布局：Measure 与 Arrange

Avalonia 采用与 WPF 和 UWP 相同的两阶段布局模型：

- **Measure**：父元素告诉每个子元素有多少可用空间，子元素返回自己期望的尺寸。
- **Arrange**：父元素在可用区域内为每个子元素分配最终的位置和大小。

布局失效会沿树向上传播。当控件的 `Width`、`Margin` 或内容发生变化时，它会调用 `InvalidateMeasure()`，从而将自己及所有祖先标记为需要重新进行布局。框架会合并这些失效请求，以确保每帧只执行一次布局阶段。

```csharp
// Custom panel: override MeasureOverride and ArrangeOverride
protected override Size MeasureOverride(Size availableSize)
{
    foreach (var child in Children)
    {
        child.Measure(availableSize);
    }

    return new Size(200, 200); // desired size
}

protected override Size ArrangeOverride(Size finalSize)
{
    foreach (var child in Children)
    {
        child.Arrange(new Rect(child.DesiredSize));
    }

    return finalSize;
}
```

### 场景图与合成

布局完成后，渲染阶段会构建场景图：这是一个由绘图指令组成的轻量级树结构，例如填充矩形、绘制文本、应用裁剪等操作。合成器会在每一帧遍历该图，并通过当前激活的渲染后端发出 GPU 绘制调用。

Avalonia 支持两种合成模式：

- **软件合成**：场景图通过 Skia 在 CPU 上光栅化，然后复制到平台窗口中。这是大多数平台上的默认方式，并且具有最广泛的兼容性。
- **GPU 合成**：在平台支持的情况下（例如 Windows 上的 Direct3D、macOS 上的 Metal、Linux 上的 OpenGL/Vulkan），Avalonia 可以直接向 GPU 发出绘制调用，从而在复杂场景下获得更好的性能。

## 平台抽象

Avalonia 通过接口将所有平台相关代码隔离起来。核心抽象包括：

| 接口 | 用途 |
|---|---|
| `IWindowImpl` | 创建窗口、设置大小、定位、原生窗口装饰 |
| `ITopLevelImpl` | 渲染表面、输入传递、缩放 |
| `IClipboard` | 剪贴板读写 |
| `IStorageProvider` | 文件和文件夹选择对话框 |
| `ILauncher` | 使用系统处理程序打开 URI 和文件 |
| `IInsetsManager` | 安全区域边距（刘海、状态栏） |
| `IPlatformSettings` | 主题检测、强调色、动画偏好 |
| `IRenderTarget` | 用于渲染的 GPU 表面 |

每个平台后端（Win32、Cocoa、X11、Android、iOS、Browser）都会实现这些接口。应用程序在启动时通过 `AppBuilder` 选择后端：

```csharp
AppBuilder.Configure<App>()
    .UsePlatformDetect() // auto-select based on OS
    .StartWithClassicDesktopLifetime(args);
```

`UsePlatformDetect()` 会自动选择正确的后端。你也可以在测试或嵌入式场景下手动指定特定后端。

## 属性系统

Avalonia 拥有自己的属性系统，支持样式、动画、数据绑定和值继承。它包含三种属性类型：

- **StyledProperty**：默认属性类型。支持样式、动画、值继承和数据绑定。大多数控件属性都属于这一类。
- **DirectProperty**：由 CLR 字段支撑的轻量级属性。它比 StyledProperty 更快，但不支持样式或动画。通常用于频繁变化的属性（例如 `TextBox.Text`）。
- **AttachedProperty**：注册在其他所有者类型上的 StyledProperty。常用于像 `Grid.Row` 和 `DockPanel.Dock` 这样的布局属性。

属性值通过优先级系统进行解析。完整优先级顺序请参阅 [Property Value Precedence](/docs/properties/value-precedence)。

## 样式系统

Avalonia 使用的是受 CSS 启发的样式系统，而不是 WPF 那种基于资源查找的触发器系统。样式通过选择器声明，可按类型、类、伪类和名称来匹配控件：

```xml
<Style Selector="Button.primary:pointerover">
    <Setter Property="Background" Value="#818CF8" />
</Style>
```

样式系统会按照声明顺序计算选择器。当多个样式同时匹配时，后声明的样式会覆盖先声明样式中相同属性的值，这与 CSS 的优先规则类似，不过 Avalonia 使用的是更简单的基于顺序的模型。

控件主题是一类特殊样式，它为某种控件类型提供默认模板和属性值。主题通过 `Theme` 属性解析，因此你可以在单个控件级别或子树级别切换主题。

## 视觉树与逻辑树

每个 Avalonia UI 都由两棵并行的树结构表示：

- **逻辑树**：在 XAML 或代码中声明的控件树。`DataContext`、资源以及属性继承都沿这棵树传播。
- **视觉树**：实际参与渲染的元素树，其中包括模板展开后的内容。命中测试、渲染和事件路由都依赖这棵树。

详细说明请参阅 [Visual and Logical Trees](/docs/fundamentals/visual-and-logical-trees)。

## 线程模型

Avalonia 的 UI 操作是单线程的。所有属性变更、布局和渲染都发生在 UI 线程上。后台任务必须通过 `Dispatcher` 将结果切回 UI 线程：

```csharp
var data = await Task.Run(() => LoadData());
// Back on UI thread automatically with async/await
Items = new ObservableCollection<Item>(data);
```

完整线程模型请参阅 [Threading](/docs/app-development/threading)。

## 另请参阅

- [Cross-platform architecture](/docs/fundamentals/cross-platform-architecture)：解决方案结构与平台特定代码模式。
- [Visual and logical trees](/docs/fundamentals/visual-and-logical-trees)：两棵树如何工作，以及分别在何时使用。
- [Property system](/docs/properties)：StyledProperty、DirectProperty 和 AttachedProperty。
- [Threading](/docs/app-development/threading)：Dispatcher 与异步模式。
- [Performance optimization](/docs/app-development/performance)：布局、渲染与绑定性能优化建议。
