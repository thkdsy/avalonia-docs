---
id: organization-chart
title: 组织结构图
description: 在层次布局中表示组织结构和汇报关系，显示职位、角色和职级。
doc-type: reference
tags:
  - avalonia pro
---

import chartsHierarchicalOrganization from '/img/controls/charts/charts-hierarchical-organization.png';

:::info
[图表](/controls/data-display/charts) 随 [Avalonia Pro](https://avaloniaui.net/pricing) 提供。
:::

组织结构图表示一个组织的结构，阐明汇报关系、相对职级和职位/角色。

<Image light={chartsHierarchicalOrganization} maxWidth={400} position="center" cornerRadius="true" alt="组织结构图，顶层 CEO 节点分支到各部门负责人，显示汇报关系。" />

## 何时使用
- **公司目录**：可视化企业的汇报结构。
- **项目层次**：绘制产品负责人、开发人员和利益相关者。
- **家谱**：显示谱系关系和血统。

## 代码示例

### XAML
```xml
<OrganizationChart xmlns="https://github.com/avaloniaui" Name="OrganizationChartSample" Title="公司结构" Height="350"
                                               ItemsSource="{Binding OrgChartData}" LabelPath="Name" ChildrenPath="Reports" />
```

### 数据模型 (C#)
```csharp
public class OrgNode
{
    public string Name { get; set; } = string.Empty;
    public string Position { get; set; } = string.Empty;
    public ObservableCollection<OrgNode> Reports { get; set; } = new();
}

public ObservableCollection<OrgNode> OrgChartData { get; } = new()
{
    new OrgNode { Name = "CEO", Reports = {
        new OrgNode { Name = "CTO", Reports = {
            new OrgNode { Name = "开发主管" }
        }},
        new OrgNode { Name = "CFO", Reports = {
            new OrgNode { Name = "会计" }
        }}
    }}
};
```

## 通用属性

| 属性 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `ItemsSource` | 层次数据源（根节点）。 | `null` |
| `LabelPath` | 节点标签的属性名称。 | `null` |
| `ChildrenPath` | 子节点集合的路径。 | `null` |
| `Orientation` | `Horizontal` 或 `Vertical` 布局。 | `Vertical` |
| `NodeWidth` | 每个节点框的宽度。 | `120.0` |
| `NodeHeight` | 每个节点框的高度。 | `50.0` |
| `NodeGap` | 层级和同级节点之间的间距。 | `30.0` |
