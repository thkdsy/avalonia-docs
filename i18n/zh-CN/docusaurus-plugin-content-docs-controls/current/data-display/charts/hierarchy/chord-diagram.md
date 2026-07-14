---
id: chord-diagram
title: 弦图
description: 在圆形布局中可视化实体之间的相互关系，非常适合显示复杂的定向流动，如贸易或 migration 模式。
doc-type: reference
tags:
  - avalonia pro
---

import chartsFlowChord from '/img/controls/charts/charts-flow-chord.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

弦图在圆形布局中可视化实体之间的相互关系。它们对于显示一组项目之间的复杂定向流动非常有用。

<Image light={chartsFlowChord} maxWidth={400} position="center" cornerRadius="true" alt="弦图，显示以圆形排列的实体之间的定向流动，连接弦的宽度不同。" />

## 何时使用
- **贸易关系**：可视化国家之间的进出口关系。
- **迁移模式**：显示人口在不同地理区域之间的流动。
- **系统交互**：可视化软件系统中模块之间的调用依赖关系。

## 代码示例

### XAML
```xml
<ChordDiagramChart xmlns="https://github.com/avaloniaui" Name="ChordDiagramSample" Title="贸易关系" Height="350"
                            ItemsSource="{Binding ChordData}"
                            SourcePath="Source"
                            TargetPath="Target"
                            ValuePath="Value" />
```

### 数据模型 (C#)
```csharp
public record TradeLink(string Source, string Target, double Value);

public ObservableCollection<TradeLink> ChordData { get; } = new()
{
    new("美国", "中国", 50),
    new("美国", "欧洲", 40),
    new("欧洲", "中国", 30),
    new("中国", "美国", 45),
    new("欧洲", "美国", 35)
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 关系集合。 | `null` |
| `SourcePath` | 源实体路径。 | `null` |
| `TargetPath` | 目标实体路径。 | `null` |
| `ValuePath` | 关系强度/权重路径。 | `null` |
| `ArcPadding` | 外环上段之间的角度间距。 | `0.02` |
| `ArcThickness` | 外部弧线的粗细。 | `20.0` |
| `ChordOpacity` | 应用于连接弦的不透明度。 | `0.6` |
| `ShowLabels` | 是否显示外部弧线的标签。 | `true` |
