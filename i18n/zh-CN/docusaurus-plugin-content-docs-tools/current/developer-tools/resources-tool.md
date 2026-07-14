---
id: resources-tool
title: 资源工具
doc-type: reference
description: 使用 DevTools 中的 Resources 工具，在运行时检查、浏览并编辑 Avalonia 应用程序的资源层级结构。
---

Resources 工具提供了应用程序资源层级的可视化视图，使你能够在运行时检查、浏览并修改资源。该工具有助于你理解资源在 Avalonia 应用中是如何组织与解析的。

在执行资源查找时，Avalonia 会沿着这套层级结构进行搜索，直到找到匹配的键。Resources 工具会将应用级别的层级可视化，因此你可以看到哪些资源在全局范围内可用。

有关 Avalonia 资源的更多信息，请参阅 [How to use resources](https://docs.avaloniaui.net/docs/guides/styles-and-resources/resources)。

:::note

控件特定资源（即定义在单个控件上或控件模板中的资源）目前不会显示在该工具中。

:::

## 浏览资源提供程序树

左侧面板会显示资源提供程序树，其中包括：

1. Application（根级别）
2. 资源字典
3. 主题字典
4. 样式
5. 其他非字典型资源提供程序

树中的每个节点都代表一个资源作用域。选中某个节点后，其资源会显示在右侧面板中。

![Resources Tree](/img/tools/dev-tools/resources-providers-list.png)

## 检查和编辑资源

右侧面板会显示当前所选提供程序中的可用资源。

你可以直接在这个视图中编辑资源，从而尝试不同的值，并立即在应用程序中看到变化效果。

![Provider Tree](/img/tools/dev-tools/resources-provider-values.png)

:::note

目前暂不支持向某个提供程序中新增资源。

:::

### 资源编辑如何映射到 XAML

当你在 Resources 工具中编辑资源值时，实际上修改的是与你在 XAML 资源定义中创建的同一个运行时对象。例如，考虑 `App.axaml` 中如下的 XAML 资源定义：

```xml
<Application.Resources>
    <SolidColorBrush x:Key="PrimaryBrush" Color="#0078D4" />
    <x:Double x:Key="DefaultFontSize">14</x:Double>
    <Thickness x:Key="StandardPadding">8,12,8,12</Thickness>
</Application.Resources>
```

这些资源会显示在 Resources 工具中的 **Application** 节点下。你可以选中 `PrimaryBrush`，并在运行时修改它的 `Color` 属性，以便在不重新构建应用程序的情况下预览不同的主题颜色。

如果你的控件通过 `DynamicResource` 引用了这些资源，它们会自动更新：

```xml
<Button Background="{DynamicResource PrimaryBrush}"
        FontSize="{DynamicResource DefaultFontSize}"
        Padding="{DynamicResource StandardPadding}"
        Content="Save" />
```

当你找到满意的值后，请将它们复制回 XAML 源文件中，以永久保存这些更改。

## 过滤与排序

Resources 工具提供了多种选项，帮助你查找特定资源：

- **Include Nested**：启用后，会显示所选节点及其子节点中全部可用资源，从而模拟运行时资源查找的方式。这有助于你识别在层级中特定位置可以访问到哪些资源。
- **Sort by**：按键名的字母顺序排列资源，或按类型分组。
- **Order**：按升序或降序排序。
- **Search filter**：按键名或类型搜索资源。

![Filter view](/img/tools/dev-tools/resources-filter.png)

## 另请参阅

- [元素工具](/tools/developer-tools/elements-tool)
- [资源文件工具](/tools/developer-tools/assets-tool)
- [How to use resources](https://docs.avaloniaui.net/docs/guides/styles-and-resources/resources)
