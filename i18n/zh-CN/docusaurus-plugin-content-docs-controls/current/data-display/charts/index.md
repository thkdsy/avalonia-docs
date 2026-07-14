---
id: index
title: 图表控件
description: Avalonia 图表是一个包含超过 70 种数据可视化控件和模式的库，适用于仪表板、金融、分析等领域。
doc-type: overview
tags:
  - avalonia pro
  - avalonia charts
---

图表提供了数据可视化控件和文档化的组合模式库，用于构建仪表板、分析工具、金融工具、科学报告等。通用图表功能，如工具提示、图例和交互性，开箱即用，并与 Avalonia 的主题系统集成。

:::info
图表随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

## 入门指南

1. 通过运行 `dotnet add package` 安装 `Avalonia.Controls.Charts` NuGet 包。

```bash
dotnet add package Avalonia.Controls.Charts
```

2. 在可执行项目文件（`.csproj`）中包含您的 Avalonia 许可证密钥。您的许可证密钥可从 [Avalonia 门户](https://portal.avaloniaui.net) 获取。

```xml
<ItemGroup>
  <AvaloniaUILicenseKey Include="YOUR_LICENSE_KEY" />
</ItemGroup>
```

:::tip
对于多项目解决方案，您可以将许可证密钥存储在[环境变量](https://learn.microsoft.com/en-us/visualstudio/msbuild/how-to-use-environment-variables-in-a-build)或[共享属性文件](https://learn.microsoft.com/en-us/visualstudio/msbuild/customize-by-directory?view=vs-2022#directorybuildprops-example)中，以避免重复。
:::

3. （可选）如果您希望在单独的 XML 命名空间中使用图表，可以使用 `https://avaloniaui.net/controls/charts`。这不是必需的——默认的 `https://github.com/avaloniaui` 命名空间也包含 `Avalonia.Controls.Charts`。

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:charts="https://avaloniaui.net/controls/charts">
    <StackPanel Spacing="16">
        <charts:CartesianChart Title="收入" Height="300" />
        <charts:SankeyChart Title="用户流程" Height="300" />
    </StackPanel>
</UserControl>
```

有关安装 Avalonia Pro 控件的更多信息，请参阅[安装 Avalonia Pro](/tools/installing-avalonia-pro)。

## 示例用例

图表适用于：

- **业务仪表板：** KPI 卡片、趋势线、柱状比较和漏斗视图，用于运营数据。
- **金融应用程序：** 蜡烛图、OHLC、Heikin-Ashi 和其他价格行为图表，用于交易和市场分析。
- **分析和报告：** 热力图、散点图、直方图和表格图表，用于探索和展示数据集。
- **科学和统计视图：** 箱线图、小提琴图、误差线和马赛克图，用于分布分析。
- **流程和层次可视化：** 桑基图、组织结构图、树图和网络图，用于关系数据。
- **地理数据：** 等值线地图、气泡地图和热力图叠加，用于区域比较。
- **进度和状态指示器：** 圆形仪表、线性仪表和液体填充仪表，用于实时监控。

## 数据源

笛卡尔系列可以通过 `ItemsSource`、`CategoryPath` 和 `ValuePath` 绑定到对象集合。对于简单数值系列，`ItemsSource` 也可以是直接的 `IList<double>`、`IList<int>`、`IList<float>`、`IList<decimal>` 或 `IList<Point>`。直接数值列表使用项目索引作为类别值。

系列在计算边界和渲染连续数据时跳过非有限数值，如 `NaN` 和无穷大。

笛卡尔系列使用 `EmptyPointMode` 来控制如何处理渲染系列中的 null 或非有限点。支持的值有 `Zero`、`Gap`、`Average` 和 `Interpolate`。默认值为 `Zero`。

## 通用图表属性

大多数图表控件通过 `ChartBase` 共享这些属性。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 图表上方显示的文本。 | `null` |
| `Palette` | 用于生成系列或项目颜色的可选图表调色板。 | `null` |
| `LabelForeground` | 当未设置更具体的标签画笔时，用于图表级标签的画笔。 | `null` |
| `PlotAreaContent` | 在有效绘图区域上排列的可选控件。 | `null` |

某些图表类型在视觉表面存在时，还会暴露额外的图表级样式表面。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `AxisBrush` | 用于轴线、刻度线、径向轴或等效刻度指南的画笔。 | `null` |
| `GridLineBrush` | 用于图表级网格线的画笔。 | `null` |
| `PlotAreaBackground` | 仅用于数据绘图矩形的画笔，而非整个图表背景。 | `null` |

## 通用系列属性

大多数系列通过 `ChartSeries` 共享这些属性。各个图表页面列出了其特定系列类型的其他属性。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `Title` | 在图例和生成的工具提示内容中显示的系列名称。 | `null` |
| `ItemsSource` | 系列使用的数据集合。 | `null` |
| `ValuePath` | 用于主要数值的属性路径。 | `null` |
| `Fill` | 用于填充系列表面或项目内部的画笔。 | `null` |
| `Stroke` | 用于系列轮廓、线条或项目边框的画笔。 | `null` |
| `StrokeThickness` | 基础描边粗细。某些默认主题会设置特定类型的值。 | `1.0` |
| `PointBrushPath` | 可选的属性路径，为每个数据项解析 `IBrush`。支持的系列将其用于各个点、标记、段或切片。 | `null` |

## 动画

图表和系列共享一个动画管道。

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `IsAnimationEnabled` | 启用图表入场动画。 | `true` |
| `AnimationDuration` | 图表入场动画的持续时间。 | `00:00:01` |
| `Easing` | 应用于图表动画进度的缓动函数。 | `CubicEaseOut` |
| `AnimationDelay` | 系列动画开始前的系列级延迟。 | `00:00:00` |
| `AnimationProgress` | 系列级动画进度，从 `0.0` 到 `1.0`。 | `1.0` |

## 扩展性

自定义笛卡尔系列派生自 `CartesianSeries` 并实现 `RenderSeries(in SeriesRenderContext context)`。渲染上下文提供系列绘图逻辑所需的图表实例、绘图区域、类别映射、视口边界和已解析的画笔。系列可以重写 `CreateLegendItem` 来自定义其图例标记，并可以通过 `GetDataPoints()` 或相关的范围和极坐标数据点结构暴露数据，用于边界、工具提示和交互。升级现有的自定义笛卡尔系列时，将任何较旧的 `TryRenderSelf(...)` 渲染逻辑移到 `RenderSeries(in SeriesRenderContext context)` 中。

金融覆盖层可以通过实现 `IFinancialChartOverlaySeries` 在 `FinancialChart` 中渲染。有关覆盖层特定行为，请参见[金融图表](/controls/data-display/charts/financial/financial-chart)。

## 图表分类

### 分析和 KPI 图表

用于总结指标、识别模式和展示发现的图表。

| 图表 | 描述 |
| --- | --- |
| [KPI 卡片](/controls/data-display/charts/analytics/kpi-card) | 显示关键指标以及趋势指示器和迷你图。 |
| [热力图图表](/controls/data-display/charts/analytics/heatmap-chart) | 将值矩阵显示为颜色编码的网格。 |
| [漏斗图](/controls/data-display/charts/analytics/funnel-chart) | 可视化数量在每个阶段递减的顺序过程。 |
| [华夫饼图](/controls/data-display/charts/analytics/waffle-chart) | 将百分比表示为网格中的填充单元格。 |
| [词云图](/controls/data-display/charts/analytics/word-cloud-chart) | 按频率调整单词大小以显示数据集中的重要性。 |
| [子弹图](/controls/data-display/charts/analytics/bullet-chart) | 在紧凑布局中比较主要值与目标和定性范围。 |
| [日历热力图](/controls/data-display/charts/analytics/calendar-heatmap-chart) | 在逐周的日历网格中显示每日活动强度。 |
| [矩阵图](/controls/data-display/charts/analytics/matrix-chart) | 在网格中显示两组类别之间的关系。 |
| [表格图](/controls/data-display/charts/analytics/table-chart) | 以行和列呈现数据，可选择性设置条件格式。 |
| [碰撞图](/controls/data-display/charts/analytics/bump-chart) | 使用交叉线跟踪排名随时间的变化。 |
| [斜率图](/controls/data-display/charts/analytics/slope-chart) | 使用斜线比较两个时间点的值。 |
| [金字塔图](/controls/data-display/charts/analytics/pyramid-chart) | 垂直堆叠段以显示层次或顺序比例。 |
| [主题河流图](/controls/data-display/charts/analytics/theme-river-chart) | 构建主题河流样式的布局，带有堆叠面积系列和居中的间隔偏移。 |
| [象形柱状图](/controls/data-display/charts/analytics/pictorial-bar-chart) | 用按值大小排列的图标或形状替换标准柱状条。 |

### 气泡和打包图

通过标记大小编码数值，通常省略传统轴或网格的图表。

| 图表 | 描述 |
| --- | --- |
| [气泡图](/controls/data-display/charts/bubble/bubble-chart) | 绘制 X 和 Y 值，并使用气泡大小作为第三个度量。 |
| [气泡云图](/controls/data-display/charts/bubble/bubble-cloud-chart) | 在无轴的有机聚类布局中排列大小不一的气泡。 |
| [打包气泡图](/controls/data-display/charts/bubble/packed-bubble-chart) | 将类别气泡紧密打包到紧凑空间中，用于部分与整体比较。 |

### 笛卡尔图表

在水平和垂直轴上绘制数据的图表。用于趋势、比较、分布和相关性。

| 图表 | 描述 |
| --- | --- |
| [柱状图](/controls/data-display/charts/cartesian/bar-chart) | 使用矩形柱比较跨类别的离散数量。 |
| [折线图](/controls/data-display/charts/cartesian/line-chart) | 用直线段连接数据点以显示随时间变化的趋势。 |
| [面积图](/controls/data-display/charts/cartesian/area-chart) | 填充线下区域以强调累积总量或体积。 |
| [组合图](/controls/data-display/charts/cartesian/combo-chart) | 在单个图表上组合多种笛卡尔系列类型，支持可选的次 Y 轴。 |
| [散点图](/controls/data-display/charts/cartesian/scatter-chart) | 绘制单个数据点以揭示两个变量之间的相关性。 |
| [样条图](/controls/data-display/charts/cartesian/spline-chart) | 用曲线连接数据点以显示时间相关数据的渐进变化。 |
| [阶梯图](/controls/data-display/charts/cartesian/step-line-chart) | 用水平和垂直阶梯连接点，用于离散状态变化。 |
| [堆叠柱状图](/controls/data-display/charts/cartesian/stacked-bar-chart) | 使用堆叠柱显示跨类别的部分与整体关系。 |
| [堆叠面积图](/controls/data-display/charts/cartesian/stacked-area-chart) | 使用堆叠填充区域显示随时间累积的总量。 |
| [范围面积图](/controls/data-display/charts/cartesian/range-area-chart) | 将高低范围显示为两个值之间的填充带。 |
| [瀑布图](/controls/data-display/charts/cartesian/waterfall-chart) | 显示初始值如何通过一系列正负贡献而变化。 |
| [直方图](/controls/data-display/charts/cartesian/histogram-chart) | 将连续值分组到数据桶中以显示频率分布。 |
| [帕累托图](/controls/data-display/charts/cartesian/pareto-chart) | 结合柱状图和累积线来识别重要因素。 |

### 圆形图表

将数据表示为圆形的段。

| 图表 | 描述 |
| --- | --- |
| [饼图](/controls/data-display/charts/circular/pie-chart) | 将圆划分为比例切片以显示部分与整体关系。 |
| [环形图](/controls/data-display/charts/circular/donut-chart) | 具有空心中心的饼图，通常用于在中间显示总计。 |
| [半环形图](/controls/data-display/charts/circular/semi-donut-chart) | 半圆形环形图，用于紧凑的部分与整体视图。 |

### 比较图表

用于前后分析、背靠背比较和比例分配的图表。

| 图表 | 描述 |
| --- | --- |
| [发散柱状图](/controls/data-display/charts/comparison/diverging-bar-chart) | 从中基线左右延伸柱状条。 |
| [哑铃图](/controls/data-display/charts/comparison/dumbbell-chart) | 用线和标记连接每个类别的两个值。 |
| [梅科图](/controls/data-display/charts/comparison/mekko-chart) | 结合可变宽度列和堆叠段以显示大小和构成。 |
| [镜像柱状图](/controls/data-display/charts/comparison/mirror-bar-chart) | 将两个柱状系列围绕中心线背靠背放置。 |
| [议会图](/controls/data-display/charts/comparison/parliament-chart) | 在半圆形布局中显示席位分布。 |
| [人口金字塔图](/controls/data-display/charts/comparison/population-pyramid-chart) | 按有序条带显示两个对立的人口分布。 |
| [龙卷风图](/controls/data-display/charts/comparison/tornado-chart) | 绘制双向水平柱状条用于敏感性或排名比较。 |
| [维恩图](/controls/data-display/charts/comparison/venn-diagram-chart) | 可视化集合重叠和交集值。 |

### 工程和科学图表

用于技术曲面、多变量科学视图和专业坐标系的图表。

| 图表 | 描述 |
| --- | --- |
| [地毯图](/controls/data-display/charts/engineering/carpet-plot-chart) | 将两个自变量和一个因变量映射到倾斜网格上。 |
| [六边形图](/controls/data-display/charts/engineering/hexbin-chart) | 将密集的 2D 点云聚合为六边形密度桶。 |
| [史密斯图](/controls/data-display/charts/engineering/smith-chart) | 在归一化射频网格上可视化复杂阻抗或导纳数据。 |
| [三元图](/controls/data-display/charts/engineering/ternary-chart) | 绘制总和为常数的三部分组合。 |
| [风玫瑰图](/controls/data-display/charts/engineering/wind-rose-chart) | 将方向频率分布显示为堆叠的极坐标扇区。 |

### 金融图表

用于价格和市场数据分析的专业图表。

| 图表 | 描述 |
| --- | --- |
| [金融图表](/controls/data-display/charts/financial/financial-chart) | 在共享价格轴上承载金融系列，如蜡烛图和 OHLC。 |
| [蜡烛图](/controls/data-display/charts/financial/candlestick-chart) | 将每个周期的开盘价、最高价、最低价和收盘价显示为蜡烛形状。 |
| [OHLC 图](/controls/data-display/charts/financial/ohlc-chart) | 将 OHLC 价格数据显示为带刻度线的垂直柱。 |
| [Heikin-Ashi 图](/controls/data-display/charts/financial/heikin-ashi-chart) | 平滑的蜡烛图变体，可过滤短期噪音。 |
| [Hilo 图](/controls/data-display/charts/financial/hilo-chart) | 仅绘制每个周期的高低值作为垂直线。 |
| [Kagi 图](/controls/data-display/charts/financial/kagi-chart) | 过滤小幅价格变动以显示重要的方向变化。 |
| [Renko 图](/controls/data-display/charts/financial/renko-chart) | 将价格变动绘制为固定大小的砖块，忽略时间。 |
| [点数图](/controls/data-display/charts/financial/point-and-figure-chart) | 使用 X 和 O 符号列来跟踪供需关系。 |

### 仪表

单个值相对于范围的视觉显示。常见于监控仪表板和实时状态面板。

| 图表 | 描述 |
| --- | --- |
| [圆形仪表](/controls/data-display/charts/gauges/circular-gauge-chart) | 带有指针或弧形指示器的表盘式仪表。 |
| [仪表图](/controls/data-display/charts/gauges/gauge-chart) | 在半圆形表盘上显示单个值，可选指针。 |
| [线性仪表](/controls/data-display/charts/gauges/linear-gauge-chart) | 带有指针或填充的水平或垂直轨道。 |
| [渐变环图](/controls/data-display/charts/gauges/gradient-ring-chart) | 绘制多个同心进度环，共享一个最大值。 |
| [液体填充仪表](/controls/data-display/charts/gauges/liquid-fill-gauge) | 将百分比表示为形状内上升的液位。 |
| [进度环形图](/controls/data-display/charts/gauges/progress-donut-chart) | 按比例填充以指示进度的圆形弧。 |

### 层次和流程图

用于可视化关系、流程和树状结构的图表。

| 图表 | 描述 |
| --- | --- |
| [流程图](/controls/data-display/charts/hierarchy/flow-chart) | 使用节点和有向边可视化工作流、决策树和系统映射。 |
| [桑基图](/controls/data-display/charts/hierarchy/sankey-chart) | 使用比例带显示节点之间的流动量。 |
| [冲积图](/controls/data-display/charts/hierarchy/alluvial-chart) | 跟踪项目如何在跨阶段类别之间过渡。 |
| [树图](/controls/data-display/charts/hierarchy/treemap-chart) | 将层次数据表示为按值大小排列的嵌套矩形。 |
| [旭日图](/controls/data-display/charts/hierarchy/sunburst-chart) | 将层次显示为从中心辐射的同心环。 |
| [圆形填充图](/controls/data-display/charts/hierarchy/circle-packing-chart) | 嵌套圆圈以表示层次比例。 |
| [火焰图](/controls/data-display/charts/hierarchy/flame-graph) | 从下到上显示开销或持续时间数据的层次堆栈。 |
| [冰柱图](/controls/data-display/charts/hierarchy/icicle-chart) | 将层次显示为从上到下的堆叠矩形带。 |
| [系统树图](/controls/data-display/charts/hierarchy/dendrogram-chart) | 用于聚类和分类上下文的树状图。 |
| [缩进树图](/controls/data-display/charts/hierarchy/indented-tree-chart) | 将层次显示为具有可展开节点的缩进列表。 |
| [径向树图](/controls/data-display/charts/hierarchy/radial-tree-chart) | 在圆形布局中排列树层次结构。 |
| [组织结构图](/controls/data-display/charts/hierarchy/organization-chart) | 可视化汇报结构和团队层次。 |
| [思维导图](/controls/data-display/charts/hierarchy/mindmap-chart) | 在 `FlowChart` 之上构建发散式布局，具有发散的节点和链接。 |
| [网络图](/controls/data-display/charts/hierarchy/network-chart) | 绘制节点和边以显示关系，无固定层次。 |
| [力导向图](/controls/data-display/charts/hierarchy/force-directed-graph) | 使用模拟物理力排列节点以揭示聚类。 |
| [弦图](/controls/data-display/charts/hierarchy/chord-diagram) | 将实体之间的成对关系显示为围绕圆的弧。 |
| [弧线图](/controls/data-display/charts/hierarchy/arc-diagram-chart) | 显示排列在直线轴上的节点之间的连接。 |
| [工艺流程图](/controls/data-display/charts/hierarchy/process-flow-chart) | 使用 `FlowChart` 表示顺序或分支工作流。 |

### 地图

将数据叠加到地理或自定义空间布局上的图表。

| 图表 | 描述 |
| --- | --- |
| [等值线地图](/controls/data-display/charts/maps/choropleth-map-chart) | 按数值对地图区域着色以显示地理分布。 |
| [气泡地图](/controls/data-display/charts/maps/bubble-map-chart) | 在地图上放置大小不等的圆圈以表示特定位置的值。 |
| [热力图](/controls/data-display/charts/maps/heatmap-map-chart) | 在地图上应用颜色渐变以显示强度或密度。 |
| [形状地图](/controls/data-display/charts/maps/shape-map-chart) | 将自定义区域渲染为数据驱动的地图。 |
| [座位图](/controls/data-display/charts/maps/seat-map-chart) | 显示场馆或平面图布局，具有交互式座位选择。 |

### 径向图表

使用圆形坐标系而非笛卡尔轴的图表。

| 图表 | 描述 |
| --- | --- |
| [极坐标图](/controls/data-display/charts/radial/polar-chart) | 在极坐标系中绘制任意角度和半径值。 |
| [雷达图](/controls/data-display/charts/radial/radar-chart) | 在圆形轴网格上将多变量数据绘制为多边形。 |
| [极坐标面积图](/controls/data-display/charts/radial/polar-area-chart) | 将圆划分为等角度段，按值大小调整。 |
| [南丁格尔玫瑰图](/controls/data-display/charts/radial/nightingale-rose-chart) | 一种极坐标面积图，其中半径而非面积编码值。 |
| [径向柱状图](/controls/data-display/charts/radial/radial-bar-chart) | 将类别显示为围绕中心点不同长度的弧。 |
| [径向折线图](/controls/data-display/charts/radial/radial-line-chart) | 投影到圆形轴上的折线图。 |

### 排程和时间线图

用于基于时间的规划、项目管理和序列数据的图表。

| 图表 | 描述 |
| --- | --- |
| [甘特图](/controls/data-display/charts/scheduling/gantt-chart) | 在水平时间轴上显示任务及其持续时间。 |
| [时间线图](/controls/data-display/charts/scheduling/timeline-chart) | 沿线性轴按时间顺序显示事件。 |
| [泳道图](/controls/data-display/charts/scheduling/swimlane-chart) | 将任务组织成平行行以显示所有权或阶段。 |
| [螺旋时间线图](/controls/data-display/charts/scheduling/spiral-timeline-chart) | 沿螺旋排列基于时间的数据以显示周期性模式。 |
| [迷你图](/controls/data-display/charts/scheduling/sparkline-chart) | 用于在小空间内显示趋势的内联微型图表。 |

### 统计图表

用于分布和比较统计分析的图表。

| 图表 | 描述 |
| --- | --- |
| [蜂群图](/controls/data-display/charts/statistical/beeswarm-plot-chart) | 在每个类别内将单个观测值显示为非重叠点。 |
| [箱线图](/controls/data-display/charts/statistical/boxplot-chart) | 使用中位数、四分位数和异常值总结分布。 |
| [等高线图](/controls/data-display/charts/statistical/contour-plot-chart) | 将 2D 标量场显示为等高线和可选的填充带。 |
| [密度图](/controls/data-display/charts/statistical/density-plot-chart) | 使用核密度估计渲染平滑的分布曲线。 |
| [误差线图](/controls/data-display/charts/statistical/error-bar-chart) | 为数据点添加误差或不确定性指示符。 |
| [马赛克图](/controls/data-display/charts/statistical/mosaic-chart) | 将两个分类变量的比例可视化为嵌套矩形。 |
| [平行坐标图](/controls/data-display/charts/statistical/parallel-coordinates-chart) | 将多变量记录比较为跨平行轴的线。 |
| [山脊线图](/controls/data-display/charts/statistical/ridgeline-chart) | 堆叠多个重叠的分布以进行形状比较。 |
| [条带图](/controls/data-display/charts/statistical/strip-plot-chart) | 显示每个类别的单个观测值，带有抖动和可选的平均线。 |
| [小提琴图](/controls/data-display/charts/statistical/violin-plot-chart) | 结合箱线图和核密度形状以显示分布。 |

## 共享元素

大多数图表类型共享以下可配置元素：

| 元素 | 描述 |
| --- | --- |
| [图例](/controls/data-display/charts/shared-elements/legend-chart) | 按名称和颜色标识每个数据系列。 |
| [工具提示](/controls/data-display/charts/shared-elements/tooltip-chart) | 在悬停或点击时显示数据值。 |
| [十字准线](/controls/data-display/charts/shared-elements/crosshairs-chart) | 绘制跟随指针跨图表的交叉线。 |
| [数据标签](/controls/data-display/charts/shared-elements/data-labels-chart) | 直接在图表上渲染每个数据点的值。 |
| [图表导出](/controls/data-display/charts/shared-elements/export-chart) | 将图表视图保存为 PNG 或 JPEG 文件、PNG 流或保存对话框。 |
| [标记](/controls/data-display/charts/shared-elements/markers-chart) | 在每个数据值处添加点符号。 |
| [趋势线](/controls/data-display/charts/shared-elements/trendline-chart) | 在系列上叠加回归线或移动平均线。 |
| [注释](/controls/data-display/charts/shared-elements/annotations-chart) | 在特定数据坐标处放置标签、线或形状。 |
| [轴自定义](/controls/data-display/charts/shared-elements/axis-customization-chart) | 控制刻度、标签、网格线和比例。 |
| [交互](/controls/data-display/charts/shared-elements/interactions-chart) | 配置缩放、平移、选择、悬停高亮和轨迹球行为。 |

## 另请参阅

- [Avalonia Pro 定价和访问](https://avaloniaui.net/pricing)
