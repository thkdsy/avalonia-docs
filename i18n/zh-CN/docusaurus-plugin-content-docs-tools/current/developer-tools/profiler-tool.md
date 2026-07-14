---
id: profiler-tool
title: 应用分析器工具
sidebar_label: 分析器工具
doc-type: reference
---

与主要包含单一维度信息、适合以图表形式展示的 [metrics](/tools/developer-tools/profiler-tool) 不同，profiler 会捕获更丰富的数据。录制完成后，`Developer Tools` 会聚合结果，并以表格形式显示。

## 录制分析数据

1. 点击 **Record** 按钮以开始一次分析会话。
2. 在后台采集数据的同时，应用程序会继续正常运行。
3. 再次点击 **Record** 以停止录制。
4. 结果会被聚合，并按不同分析器类型显示在独立的选项卡中。

为了获得最佳结果，请尽量隔离你想测量的那段交互。例如，在录制期间只打开某个特定视图，或只触发一次特定 UI 操作，这样数据就不会被无关活动稀释。

## 样式匹配

当控件被创建或添加到可视树中时，Avalonia 会评估所有处于激活状态的样式选择器，以确定哪些选择器适用于该控件。Style Matching 分析器会聚合这些匹配尝试。

列说明：
- **Selector**：当前被评估的样式选择器（例如 `TextBlock.h1`、`Button:pointerover > ContentPresenter`）
- **Elapsed**：在所有匹配尝试中，评估该选择器所花费的总时间
- **Fast Reject Count**：该选择器在未进行完整评估前被快速排除的次数。快速排除发生在控件能够通过简单的静态检查被排除时，例如类型或控件名称不匹配。需要注意的是，被快速排除的控件在激活器变化后也不会再次被评估。例如，`TextBox` 不会针对 `Button:pointerover` 再次评估，因为它已经在类型检查时被排除了。
- **Match Attempts**：该选择器针对控件进行测试的总次数
- **Matches**：成功匹配的次数

如果匹配尝试次数很多，但成功匹配次数很少，这通常说明选择器范围过宽。你可以考虑将目标收窄到具体控件类型，而不是基类，以减少控件创建期间不必要的匹配。

## 样式激活器

与用于衡量初始选择器解析过程的 Style Matching 不同，这个分析器测量的是选择器在运行时被重新评估的频率。当用户与应用交互时，条件选择器（即带有 `:pointerover`、`:focus`、`:pressed` 这类激活器的选择器）会不断启用和停用，从而触发重新评估。

列说明：
- **Selector**：被重新评估的样式选择器
- **Elapsed**：重新评估所花费的总时间
- **Evaluations**：该选择器激活器被重新评估的总次数
- **Active Evaluations**：最终导致样式进入激活状态的评估次数
- **Activator**：负责触发的激活器类型（例如伪类、属性匹配）

如果某个选择器的评估次数异常高，可能意味着它的开关状态变化比预期更频繁。适当缩小这类选择器的作用范围会有所帮助。

## 资源查找

每当控件、样式或绑定通过键解析某个资源（例如画刷、Thickness 或模板）时，Avalonia 都会沿着资源层级逐级查找，直到找到匹配项。这个分析器会按键聚合这些查找记录。

列说明：
- **Key**：正在查找的资源键
- **Elapsed**：在所有查找中解析该键所花费的总时间
- **Total Lookups**：该键被请求的总次数
- **Successful**：成功找到匹配资源的次数
- **Theme Variant**：查找时处于活动状态的主题变体（Light/Dark）

如果某个键的总查找次数很多，但成功次数很少，通常说明资源定义缺失，或资源键拼写有误。你可以使用 [Resources Tool](/tools/developer-tools/resources-tool) 检查各个作用域下可用的资源。

## 另请参阅

- [指标工具](/tools/developer-tools/metrics-tool)
- [资源工具](/tools/developer-tools/resources-tool)
