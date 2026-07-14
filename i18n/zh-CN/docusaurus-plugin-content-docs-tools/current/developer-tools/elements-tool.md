---
id: elements-tool
title: 元素工具
doc-type: reference
---

Elements Tree 提供了一个统一视图，将可视层级与逻辑层级结合在一起。为了优化性能，它只加载可见元素，同时以逻辑树为基础组织整体结构。模板内容会折叠在 `/template/` 节点之下。

![Elements Tool](/img/tools/dev-tools/elements-tool.png)

## 检查模式

Elements Tool 提供了多种方式，让你可以直接从正在运行的应用程序中定位并选择特定 UI 元素：

- **焦点跟踪** - 启用后，该功能会在 Elements 树中自动选中应用里当前拥有焦点的元素。你可以使用 <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>K</kbd>（macOS 上为 <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>K</kbd>）切换此模式，以便在与应用交互时自动跟踪焦点变化。

- **检查元素** - 该模式会将你的光标变成元素选择器。启用方式为 <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>C</kbd>（macOS 上为 <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>C</kbd>）。启用后，点击应用中的任意元素，就会立即在 Elements 树中定位并选中它。这在你所看到的界面与其底层结构之间建立了直接关联。

这些检查模式能帮助你定位元素，而无需手动在元素层级中逐层查找。

## 上下文菜单

上下文菜单提供了浏览和操作元素树所需的核心操作：

右键单击某个元素时，你可以使用多种展开选项，以不同细节层级浏览其层次结构。`Expand Children` 会显示直接子元素，而 `Recursively` 与 `Recursively with templates` 则可进行更深入的递归展开。

其他操作还包括折叠节点、复制元素或其选择器、让元素获得焦点、滚动到可视区域内，以及使视觉对象失效。窗口元素还支持显示诸如 `FPS` 之类的调试叠加层。

整个树结构都支持搜索功能，你可以按名称或类型快速定位特定元素。

![Elements Tree Context Menu](/img/tools/dev-tools/elements-context-menu.png)

## 伪类选择器

对于每个元素，该工具都会显示其上定义的伪类。这个功能对于测试元素在不同状态下的响应尤其有价值，因为你不必再通过真实用户交互手动触发这些状态。

在开发带有伪类的自定义控件时，添加 `[PseudoClassesAttribute]` 能改善其与 Developer Tools 的集成效果，同时也能增强 IDE 的自动补全支持。

## 元素属性

Properties 面板会显示 Elements 树中当前选中元素的详细信息，包括影响该元素的全部属性、样式和值。

![Properties list](/img/tools/dev-tools/properties-list.png)

该面板会显示分配给该元素的全部 Avalonia 属性。开发者可以：
- 按名称筛选属性
- 按字母顺序或按值排序属性
- 按类别对属性分组
- 使用专用编辑器编辑值（ColorPicker、BrushPicker、Image/Geometry 预览）
- 点击带有嵌套网格的属性，以预览 `DataContext` 或 `Image.Source` 这类属性

### 属性详情

当开发者选中某个属性后，可以通过两个专门的选项卡查看更多详细信息。

#### 样式和值

Avalonia 属性采用基于优先级的系统，同一个属性可以被赋予多个值。Properties 面板会展示这种分层机制：

![Styles setters](/img/tools/dev-tools/properties-style-setters.png)

每个属性都可能拥有多个具有不同优先级和条件的 setter。例如，一个按钮在正常状态、悬停状态和按下状态下，可能分别定义了不同的背景色。DevTools 会显示所有这些 setter，并默认展开当前生效的那一个。

未激活的 setter（即当前条件不满足的那些）会以折叠且灰显的方式显示。这种可视层级有助于开发者理解当前到底应用了哪个样式，以及为什么会应用它，从而更容易排查样式问题。

#### 绑定表达式

绑定表达式选项卡会展示属性是如何与数据源连接的：

![Binding Expressions](/img/tools/dev-tools/properties-bindings.png)

当某个属性使用数据绑定时，此选项卡会显示绑定关系中的关键信息：
- 绑定的 Source 和 Path
- 如果绑定失败，则显示验证错误
- 诸如 Mode、Converter 和 FallbackValue 等附加绑定参数
对于存在验证错误的属性，面板会显示异常类型和异常消息，并包含可能为调试提供额外上下文的内部异常。
某些属性会使用组合多个数据源的 MultiBinding 表达式：
![MultiBinding Expressions](/img/tools/dev-tools/properties-multi-bindings.png)
## 元素 3D 查看器
3D Viewer 为应用程序的可视树提供三维可视化视图，使你可以在空间上下文中查看 UI 元素的层叠关系与层级结构。

![3D Viewer Tab](/img/tools/dev-tools/3d-viewer-mini-demo.gif)

### 打开 3D Viewer

你可以在 Developer Tools 面板中，通过切换 Properties 视图工具栏上的 “3D Viewer” 按钮来打开它。
也可以在 Elements Tree 的上下文菜单中选择 “Open 3D Viewer”。

任意可视元素子树都可以查看，但模板和根级 Application 不支持。

:::note

此功能要求 Avalonia 11.2.0 或更高版本。

:::

### 功能

3D Viewer 会将可视树的每一层渲染为三维空间中的独立平面。

元素会根据它们的 Z-index 和渲染顺序进行定位，这使你可以轻松识别彼此重叠的元素及其堆叠上下文。

#### 导航控制

你可以在 3D 空间中导航，从不同角度观察 UI：

- **Rotate**：点击并拖动以旋转视图
- **Pan**：右键拖动以移动摄像机位置
- **Zoom**：使用鼠标滚轮进行缩放
- **Reset**：双击将视图重置为默认位置

#### 可视化设置

你可以自定义 3D 视图渲染元素的方式：

- **Draw as Gradient**：切换后使用渐变着色显示元素，以获得更好的深度感知
- **Draw Borders**：启用或禁用元素边框渲染，以获得更简洁的视觉效果
- **Layer Distance**：调整可视树各层之间的间距
- **Layer Range**：设置最小和最大层索引，以聚焦于可视树中的特定深度范围

### 3D Viewer 的使用场景

- **调试 Z-Index 问题**：识别并解决元素堆叠问题
- **理解复杂布局**：可视化嵌套面板与控件之间的关系
- **优化可视树**：识别不必要的嵌套或冗余容器
- **讲解 UI 架构**：将其作为教学工具来展示可视树概念

## 应用内叠加层

Avalonia Developer Tools 提供直接显示在运行中应用程序上的可视化叠加层，帮助你在不修改代码的前提下可视化并调试 UI 组件。

### 启用叠加层

你可以通过两种方式启用叠加层：

#### 1. 通过 Elements Tree

当你在 Developer Tools 树中将鼠标悬停在元素上时，叠加层会自动显示。

![Trigger overlays from the Elements Tree](/img/tools/dev-tools/overlay-tree-inspect.png)

#### 2. 通过 “Highlight Elements” 模式快捷键

直接在应用程序中进入检查模式：
- 按下 <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>H</kbd>（Windows/Linux）或 <kbd>⌘</kbd> <kbd>⇧</kbd> <kbd>H</kbd>（macOS）
- 将鼠标悬停在任意元素上即可查看其叠加层
- 如有需要，可再次按下相同快捷键关闭该模式

![In-App Overlays inspect via Shortcut](/img/tools/dev-tools/overlay-shortcut-inspect.png)

### 可用叠加层

#### 信息提示框

悬停时显示详细元素信息：

- **基本信息**：元素类型、名称和样式类
- **布局属性**：尺寸、边距、内边距、约束和 Z-Index
- **视觉属性**：边框与背景细节、颜色和不透明度
- **文本属性**：前景色、字体设置
- **控件特定属性**：选择画刷、图像细节

![Info Tooltip](/img/tools/dev-tools/overlay-info-tooltip.png)

#### 布局叠加层

通过颜色高亮来可视化 UI 结构：

- **Margin**：以半透明方式高亮边距区域
- **Padding**：以半透明方式高亮内边距区域
- **Bounds**：用实线边框标出控件实际边界

![Margin Padding layout overlay](/img/tools/dev-tools/overlay-margin-padding.png)

#### 标尺叠加层

提供测量参考：

- 沿窗口边缘显示水平和垂直标尺
- 使用辅助线将内容边界连接到标尺

## 另请参阅

- [事件工具](/tools/developer-tools/events-tool)
- [Developer tools 快捷键](/tools/developer-tools/shortcuts)
