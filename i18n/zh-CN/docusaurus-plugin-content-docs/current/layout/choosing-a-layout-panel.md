---
id: choosing-a-layout-panel
title: 选择布局面板
description: 对比 Avalonia 的面板控件，并为你的布局策略选择合适的面板。
doc-type: how-to
---

Avalonia 提供了多种可满足不同 UI 角色的布局面板。本指南将帮助你根据目标布局策略选择最合适的面板控件。

## 决策流程

1. 你是否需要具有不同尺寸的行和列？**使用 [Grid](#grid)**。
2. 你是否需要围绕内容区布置页眉、页脚或侧边栏？**使用 [DockPanel](#dockpanel)**。
3. 你是否需要让元素沿单一方向堆叠？**使用 [StackPanel](#stackpanel)**。
4. 你是否需要在空间不足时让元素自动换到下一行？**使用 [WrapPanel](#wrappanel)**。
5. 你是否需要由等尺寸单元格组成的均匀网格？**使用 [UniformGrid](#uniformgrid)**。
6. 你是否需要让元素彼此相对定位？**使用 [RelativePanel](#relativepanel)**。
7. 你是否需要像素级精确的绝对定位？**使用 [Canvas](#canvas)**。
8. 你是否需要让子元素彼此叠放？**使用 [Panel](#panel)**。

## 快速对比

| 面板 | 排列方式 | 是否适应窗口大小 | 最适合的场景 |
|---|---|---|---|
| [Grid](/controls/layout/panels/grid) | 行与列 | 是 | 通用布局、表单、仪表盘 |
| [DockPanel](/controls/layout/panels/dockpanel) | 边缘停靠（上、下、左、右）+ 填充 | 是 | 带页眉、侧边栏和内容区的应用框架 |
| [StackPanel](/controls/layout/panels/stackpanel) | 单线排列（垂直或水平） | 部分支持（垂直方向拉伸，主方向可滚动） | 工具栏、菜单、简单控件序列 |
| [WrapPanel](/controls/layout/panels/wrappanel) | 顺序排列并自动换行 | 是 | 标签云、图标网格、响应式项目集合 |
| [UniformGrid](/controls/layout/panels/uniformgrid) | 等尺寸单元格 | 是 | 计算器键盘、图片画廊、等尺寸卡片仪表盘 |
| [RelativePanel](/controls/layout/panels/relativepanel) | 相对于兄弟元素或面板边缘定位 | 是 | 基于可用空间重新排布的自适应布局 |
| [Canvas](/controls/layout/panels/canvas) | 绝对坐标 | 否 | 绘图表面、示意图、自定义叠加层 |
| [Panel](/controls/layout/panels/panel) | 子元素相互叠放 | 是 | 叠加层、同位置视觉堆叠 |

## Grid

一种适用于许多常见布局的通用面板。你可以定义固定尺寸、比例尺寸（`*`）或自动尺寸（`Auto`）的行和列，并将子元素放入指定的目标单元格中。

<XamlPreview>

```xml
<Grid xmlns="https://github.com/avaloniaui"
      ColumnDefinitions="100,*"
      RowDefinitions="Auto,*"
      ShowGridLines="true">
    <TextBlock Grid.Row="0" Grid.Column="0" Text="Label" />
    <TextBox Grid.Row="0" Grid.Column="1" />
    <Button Grid.Row="1" Grid.Column="0" Grid.ColumnSpan="2" HorizontalAlignment="Center">
      Button spanning two columns
    </Button>
</Grid>
```

</XamlPreview>

**适用场景：** 你需要由不同行列尺寸组成的结构化布局，例如表单、仪表盘，或任何同时包含固定区域与弹性区域的界面。

**不适用场景：** 如果你只需要让所有子元素沿单一方向堆叠，请改用 [`StackPanel`](#stackpanel)。如果你需要所有单元格尺寸完全一致，请改用 [`UniformGrid`](#uniformgrid)。此外，`Grid` 比更简单的面板更重，因此在不需要其复杂能力时，优先选择更轻量的面板类型。

更多信息请参阅 [Grid](/controls/layout/panels/grid) 页面。

## DockPanel

将子元素停靠到面板边缘，最后一个子元素会填充剩余空间。

<XamlPreview>

```xml
<DockPanel xmlns="https://github.com/avaloniaui">

    <Menu DockPanel.Dock="Top" Background="Gray">
      <MenuItem Header="_File" />
      <MenuItem Header="_Edit" />
    </Menu>

    <TextBlock DockPanel.Dock="Bottom" Background="Lime" Text="Ready" />

    <StackPanel DockPanel.Dock="Right" Width="100">
      <Button Content="Zoom in" />
      <Button Content="Zoom out" />
    </StackPanel>

    <ContentControl Background="Beige" />  <!-- fills remaining space -->

</DockPanel>
```

</XamlPreview>

**适用场景：** 你正在构建应用外壳，需要围绕中心内容区域放置固定的页眉、页脚和/或侧边栏。

**不适用场景：** 如果你需要让子元素按比例共享空间，请改用 [`Grid`](#grid)。`DockPanel` 会优先处理更早声明的子元素。

更多信息请参阅 [DockPanel](/controls/layout/panels/dockpanel) 页面。

## StackPanel

将子元素排列成单行，可以是垂直方向（默认）或水平方向。子元素会在垂直于主方向的轴上拉伸填充。

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui"
            Spacing="8">
    <TextBlock Text="Name" />
    <TextBox />
    <TextBlock Text="Email" />
    <TextBox />
    <Button Content="Submit" HorizontalAlignment="Right" />
</StackPanel>
```

</XamlPreview>

**适用场景：** 你想要一个简短的线性控件序列，例如工具栏、设置表单或菜单。

**不适用场景：** 如果项目序列很长，可能超出可用空间，则不适合单独使用 `StackPanel`。因为它会给子元素提供无限空间，并不会自动触发滚动。此时可以考虑包裹在 `ScrollViewer` 中，或改用支持虚拟化的 `ItemsControl`。

更多信息请参阅 [StackPanel](/controls/layout/panels/stackpanel) 页面。

## WrapPanel

将子元素从左到右（或从上到下）排列，在到达边缘时自动换到下一行。

<XamlPreview>

```xml
<WrapPanel xmlns="https://github.com/avaloniaui" 
           ItemSpacing="8" LineSpacing="8">
    <Button Content="One" />
    <Button Content="Two" />
    <Button Content="Three" />
    <Button Content="Four" />
    <Button Content="Five" />
</WrapPanel>
```

</XamlPreview>

**适用场景：** 你希望项目根据可用空间自然流动排列，例如标签、缩略图或响应式按钮栏。

**不适用场景：** 如果你需要项目按严格规则对齐，则不适合使用 `WrapPanel`。因为不同行上的项目间距彼此独立，通常不会形成整齐的列。

更多信息请参阅 [WrapPanel](/controls/layout/panels/wrappanel) 页面。

## UniformGrid

将可用空间划分为大小相等的单元格，子元素按顺序填入这些单元格。

<XamlPreview>

```xml
<UniformGrid xmlns="https://github.com/avaloniaui" 
             Columns="3">
    <Button Content="7" />
    <Button Content="8" />
    <Button Content="9" />
    <Button Content="4" />
    <Button Content="5" />
    <Button Content="6" />
    <Button Content="1" />
    <Button Content="2" />
    <Button Content="3" />
</UniformGrid>
```

</XamlPreview>

**适用场景：** 每个项目都应具有相同尺寸，例如计算器键盘、调色板或由等尺寸卡片组成的仪表盘。

**不适用场景：** 如果你需要不同尺寸的项目，请改用 [`Grid`](#grid)。

更多信息请参阅 [UniformGrid](/controls/layout/panels/uniformgrid) 页面。

## RelativePanel

使用附加属性，让子元素相对于兄弟控件或面板边缘进行定位。

<XamlPreview>

```xml
<RelativePanel xmlns="https://github.com/avaloniaui">
    <TextBlock Name="TitleText" Text="Title"
               RelativePanel.AlignTopWithPanel="True"
               RelativePanel.AlignLeftWithPanel="True" />
    <TextBox Name="SearchBox"
             RelativePanel.Below="TitleText"
             RelativePanel.AlignLeftWith="TitleText"
             RelativePanel.AlignRightWithPanel="True"
             Margin="0,8,0,0" />
    <Button Content="Search"
            RelativePanel.Below="SearchBox"
            RelativePanel.AlignRightWithPanel="True"
            Margin="0,8,0,0" />
</RelativePanel>
```

</XamlPreview>

**适用场景：** 你需要描述控件之间的布局关系，例如“把这个按钮放在那个文本框下方，并与右边缘对齐”。这对可根据可用空间调整关系的自适应布局尤其有用。

**不适用场景：** 如果更简单的面板已经足以实现布局，就不必使用 `RelativePanel`。它虽然灵活，但写法也更冗长；如果布局能用 [`Grid`](#grid) 或 [`StackPanel`](#stackpanel) 表达，优先使用它们。

更多信息请参阅 [RelativePanel](/controls/layout/panels/relativepanel) 页面。

## Canvas

使用 `Canvas.Left`、`Canvas.Top`、`Canvas.Right` 和 `Canvas.Bottom` 将子元素放置在精确的像素坐标上。

<XamlPreview>

```xml
<Canvas xmlns="https://github.com/avaloniaui">
    <Ellipse Canvas.Left="50" Canvas.Top="30"
             Width="80" Height="80" Fill="Blue" />
    <Rectangle Canvas.Left="150" Canvas.Top="60"
               Width="100" Height="50" Fill="Red" />
</Canvas>
```

</XamlPreview>

**适用场景：** 你需要为绘图表面或示意图进行绝对定位，或者构建一个可将对象放置到特定坐标的拖放界面。

**不适用场景：** 如果你正在构建标准应用界面，通常不适合使用 `Canvas`。它不会随窗口大小变化自动适配，因此窗口调整后内容可能被裁切，或留下空白区域。

更多信息请参阅 [Canvas](/controls/layout/panels/canvas) 页面。

## Panel

一种最简容器，会按声明顺序将子元素层叠放置。可使用 `ZIndex` 控制哪个子元素显示在最上层。

<XamlPreview>

```xml
<Panel xmlns="https://github.com/avaloniaui">
    <ContentControl Name="BG" Background="Gray" />
    <TextBlock Text="Overlay text"
                HorizontalAlignment="Center"
                VerticalAlignment="Center" />
</Panel>
```

</XamlPreview>

**适用场景：** 你想将一个控件叠加在另一个控件之上，例如给图片加水印，或在内容上方显示加载指示器。

**不适用场景：** 如果你想让子元素并排显示或按顺序排列，则不应使用它。

更多信息请参阅 [Panel](/controls/layout/panels/panel) 页面。

## 嵌套面板

对于复杂布局，你可以组合多个面板。将面板彼此嵌套，并在每一层尽量使用最简单、最合适的面板。

<XamlPreview>

```xml
<DockPanel xmlns="https://github.com/avaloniaui">

    <!-- App shell: header + sidebar + content -->
    <Menu DockPanel.Dock="Top"
          Background="Gray">
      <MenuItem Header="_File" />
      <MenuItem Header="_Edit" />
    </Menu>

    <StackPanel DockPanel.Dock="Left"
                Width="100"
                Spacing="4"
                Background="Navy">
        <!-- Sidebar: vertical list of navigation items -->
        <Button Content="Home" />
        <Button Content="Settings" />
    </StackPanel>

    <Grid RowDefinitions="*,Auto">
        <!-- Content area: main content + status bar -->
        <ContentControl Grid.Row="0" />
        <TextBlock Grid.Row="1" Background="Lime" Text="Ready" />
    </Grid>

</DockPanel>
```

</XamlPreview>

### 性能提示

- 尽可能优先使用更简单的面板。[`StackPanel`](#stackpanel) 和 [`Panel`](#panel) 通常比 [`Grid`](#grid) 更轻量。
- 避免过深的面板嵌套。如果你发现布局已经嵌套超过三层，可以考虑是否能用一个定义好行列的 [`Grid`](#grid) 取代整棵结构树。
- 对于大型可滚动列表，优先使用带虚拟化能力的 [`ItemsRepeater`](/controls/data-display/collections/itemsrepeater) 或 [`ListBox`](/controls/data-display/collections/listbox)，而不是在 `ScrollViewer` 内的 `StackPanel` 中放入数百个控件。

## 另请参阅

- [布局](/docs/layout)：了解测量与排列系统的工作方式。
- [控件定位](/docs/layout/positioning-controls)：了解对齐、边距和内边距的用法。
- [响应式布局操作指南](/docs/how-to/responsive-layout-how-to)：了解自适应布局技巧。
- 各个面板的参考页面：
  - [Grid](/controls/layout/panels/grid)
  - [DockPanel](/controls/layout/panels/dockpanel)
  - [StackPanel](/controls/layout/panels/stackpanel)
  - [WrapPanel](/controls/layout/panels/wrappanel)
  - [Canvas](/controls/layout/panels/canvas)
  - [RelativePanel](/controls/layout/panels/relativepanel)
  - [UniformGrid](/controls/layout/panels/uniformgrid)
  - [Panel](/controls/layout/panels/panel).
