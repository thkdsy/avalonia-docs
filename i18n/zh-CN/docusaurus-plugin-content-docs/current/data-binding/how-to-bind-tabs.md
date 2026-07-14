---
id: how-to-bind-tabs
title: 如何绑定标签页
description: 将 TabControl 绑定到视图模型集合，以创建动态标签页界面。
doc-type: how-to
---

当你的应用程序需要显示数量可变的标签页时，可以将 [`TabControl`](/api/avalonia/controls/tabcontrol) 绑定到一个视图模型集合，而不是在 XAML 中静态声明每一个标签页。这种方式特别适合标签数量由运行时决定的场景，例如由用户操作、加载数据或插件系统动态生成时。

通用模式如下：

1. 定义一个表示单个标签页的视图模型类（包含标题文本、内容以及其他状态）。
2. 在主视图模型中公开一个保存这些视图模型的 `ObservableCollection`。
3. 将 `TabControl.ItemsSource` 绑定到该集合，并通过 `ItemTemplate` 与 `ContentTemplate` 控制每个标签页的显示方式。

## 绑定支持示例

你可以通过 **数据绑定** 动态创建标签页项。具体做法是把 `TabControl` 的 `ItemsSource` 属性绑定到一个表示标签头和内容的对象集合上。

然后再使用 **数据模板** 来显示这些对象。

这个示例使用了由下列 `ItemViewModel` 类创建出来的对象集合：

```csharp
namespace MyApp.ViewModel;

public class ItemViewModel
{
    public string Header { get; }
    public string Content { get; }
    public ItemViewModel(string header, string content)
    {
        Header = header;
        Content = content;
    }
}
```

接着创建一个属性，用来访问 `ItemViewModel` 实例集合。

```csharp
public ObservableCollection<ItemViewModel> Items { get; set; } = new() {
    new ItemViewModel("One", "Some content on first tab"),
    new ItemViewModel("Two", "Some content on second tab"),
};
```

`TabStrip` 的标题内容由 `ItemTemplate` 属性定义，而 `TabItem` 的内容则由 `ContentTemplate` 属性定义。

最后，创建一个 `TabControl`，并把它的 `ItemsSource` 属性绑定到 `Items`。

```xml
<TabControl ItemsSource="{Binding Items}">
    <TabControl.ItemTemplate>
      <DataTemplate>
        <TextBlock Text="{Binding Header}" />
      </DataTemplate>
    </TabControl.ItemTemplate>
    <TabControl.ContentTemplate>
        <!-- ContentTemplate 中的 DataTemplate 必须在 DataType 中指定视图模型。
        别名 'vm' 对应的是 XAML 根元素属性中声明的视图模型命名空间，例如：
            xmlns:vm="using:MyApp.ViewModel"
        或者
            xmlns:vm="clr-namespace:MyApp.ViewModel;assembly=MyApp.ViewModel" -->
      <DataTemplate DataType="vm:ItemViewModel">
        <DockPanel LastChildFill="True">
          <TextBlock Text="This is content of selected tab" DockPanel.Dock="Top" FontWeight="Bold" />
          <TextBlock Text="{Binding Content}" />
        </DockPanel>
      </DataTemplate>
    </TabControl.ContentTemplate>
  </TabControl>
```

## 标签页生命周期说明

在使用数据绑定标签页时，请留意以下几点：

- **添加与移除标签页。** 因为 `ItemsSource` 绑定到了 `ObservableCollection`，所以只要往集合中添加或删除项，运行时的标签页也会自动同步增减。
- **内容重建。** 当用户切换标签页时，`TabControl` 会重新创建内容视觉树。如果你的标签内容构建成本较高，可以考虑缓存生成的视图，或者使用带独立视图模型的 `UserControl` 来保留状态。
- **当前选中标签。** 你可以把 `SelectedItem` 或 `SelectedIndex` 绑定到 `TabControl` 上，用于跟踪或控制当前活动标签。当你从集合中移除当前选中项时，选择状态会自动重置。
- **ContentTemplate 上的 DataType。** 请始终为 `ContentTemplate` 中使用的 `DataTemplate` 设置 `DataType`。否则绑定上下文可能无法正确解析。

## 另请参阅

- [TabControl](/controls/navigation/tabcontrol): Full reference for the `TabControl` control.
- [How to: Work with TabControl](/docs/how-to/tabcontrol-how-to): Static tabs, closeable tabs, and tab styling.
- [Data templates](/docs/data-templates/introduction-to-data-templates): Controlling how items are displayed.
- [Data binding syntax](/docs/data-binding/data-binding-syntax): Binding paths and modes.
