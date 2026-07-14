---
id: treeview-how-to
title: "如何：使用 TreeView"
description: 了解 TreeView 中的层级数据绑定、延迟加载、选择处理与自定义方式。
doc-type: how-to
---

本指南介绍常见的 TreeView 使用场景：层级数据绑定、延迟加载、选择处理以及自定义节点显示。

## 基础层级绑定

使用 `HierarchicalDataTemplate`，可以把 [`TreeView`](/api/avalonia/controls/treeview) 绑定到一棵由视图模型对象组成的树结构上：

```csharp
public class FolderItem
{
    public string Name { get; set; } = "";
    public ObservableCollection<FolderItem> Children { get; } = new();
}
```

```xml
<TreeView ItemsSource="{Binding RootFolders}">
    <TreeView.ItemTemplate>
        <TreeDataTemplate ItemsSource="{Binding Children}">
            <TextBlock Text="{Binding Name}" />
        </TreeDataTemplate>
    </TreeView.ItemTemplate>
</TreeView>
```

[`TreeDataTemplate`](/api/avalonia/markup/xaml/templates/treedatatemplate) 是这里的关键：它的 `ItemsSource` 属性告诉 `TreeView` 应该去哪里查找每个节点的子项。同一个模板会在每一层递归使用。

### 多种节点类型

你可以结合 `DataType` 使用不同模板模式，以展示不同类型的节点：

```csharp
public class FolderNode
{
    public string Name { get; set; } = "";
    public ObservableCollection<object> Children { get; } = new();
}

public class FileNode
{
    public string Name { get; set; } = "";
    public long Size { get; set; }
}
```

```xml
<TreeView ItemsSource="{Binding RootItems}">
    <TreeView.DataTemplates>
        <TreeDataTemplate DataType="local:FolderNode" ItemsSource="{Binding Children}">
            <StackPanel Orientation="Horizontal" Spacing="4">
                <PathIcon Data="{StaticResource FolderIcon}" />
                <TextBlock Text="{Binding Name}" />
            </StackPanel>
        </TreeDataTemplate>
        <DataTemplate DataType="local:FileNode">
            <StackPanel Orientation="Horizontal" Spacing="4">
                <PathIcon Data="{StaticResource FileIcon}" />
                <TextBlock Text="{Binding Name}" />
                <TextBlock Text="{Binding Size, StringFormat='{}{0:N0} bytes'}"
                           Foreground="Gray" />
            </StackPanel>
        </DataTemplate>
    </TreeView.DataTemplates>
</TreeView>
```

由于 `FileNode` 没有子项，因此它使用普通的 `DataTemplate`（不需要 `ItemsSource`）；而 `FolderNode` 则使用 `TreeDataTemplate`，以支持继续展开。

## 选择

### 单选

通过绑定 `SelectedItem` 来跟踪当前选中的节点：

```xml
<TreeView ItemsSource="{Binding Items}"
          SelectedItem="{Binding SelectedNode}">
```

```csharp
[ObservableProperty]
private object? _selectedNode;

partial void OnSelectedNodeChanged(object? value)
{
    if (value is FolderNode folder)
        LoadFolderContents(folder);
}
```

### 多选

通过 `SelectionMode` 启用多选：

```xml
<TreeView ItemsSource="{Binding Items}"
          SelectionMode="Multiple">
```

你可以在 code-behind 中通过 `SelectedItems` 属性访问已选项，或者使用 `SelectionChanged` 事件：

```csharp
private void OnSelectionChanged(object? sender, SelectionChangedEventArgs e)
{
    var tree = (TreeView)sender!;
    var selectedItems = tree.SelectedItems;
    // 处理已选中的项目
}
```

## 延迟加载（展开时加载）

对于大型树结构，如果一开始就把全部子节点加载出来成本太高，那么更适合在用户展开节点时按需加载：

```csharp
public partial class LazyFolderNode : ObservableObject
{
    private bool _isLoaded;

    public string Name { get; }
    public string Path { get; }
    public ObservableCollection<LazyFolderNode> Children { get; } = new();

    // 先放一个占位子项，这样展开箭头才会显示出来
    public LazyFolderNode(string name, string path, bool hasChildren = true)
    {
        Name = name;
        Path = path;
        if (hasChildren)
            Children.Add(new LazyFolderNode("Loading...", "", false));
    }

    [ObservableProperty]
    private bool _isExpanded;

    partial void OnIsExpandedChanged(bool value)
    {
        if (value && !_isLoaded)
        {
            _isLoaded = true;
            LoadChildren();
        }
    }

    private void LoadChildren()
    {
        Children.Clear();
        foreach (var dir in Directory.GetDirectories(Path))
        {
            var name = System.IO.Path.GetFileName(dir);
            Children.Add(new LazyFolderNode(name, dir));
        }
    }
}
```

在 `TreeDataTemplate` 中绑定 `IsExpanded`：

```xml
<TreeView ItemsSource="{Binding RootFolders}">
    <TreeView.Styles>
        <Style Selector="TreeViewItem">
            <Setter Property="IsExpanded" Value="{Binding IsExpanded, Mode=TwoWay}" />
        </Style>
    </TreeView.Styles>
    <TreeView.ItemTemplate>
        <TreeDataTemplate ItemsSource="{Binding Children}">
            <TextBlock Text="{Binding Name}" />
        </TreeDataTemplate>
    </TreeView.ItemTemplate>
</TreeView>
```

这里通过 `TreeViewItem` 样式，把容器上的 `IsExpanded` 绑定到了视图模型属性上。当用户展开节点时，setter 会触发 `OnIsExpandedChanged`，从而加载子节点。

## 异步延迟加载

如果子节点需要从数据库或 API 中加载，可以这样做：

```csharp
partial void OnIsExpandedChanged(bool value)
{
    if (value && !_isLoaded)
    {
        _isLoaded = true;
        _ = LoadChildrenAsync();
    }
}

private async Task LoadChildrenAsync()
{
    var items = await _service.GetChildrenAsync(Id);

    Children.Clear();
    foreach (var item in items)
        Children.Add(new LazyFolderNode(item));
}
```

由于 `LoadChildrenAsync` 是 `async` 方法，因此在 `await` 之后会回到 UI 线程，所以更新 `Children` 时无需显式调用 dispatcher。

## 通过代码展开和折叠

如果你想一次性展开或折叠所有节点，可以递归遍历整棵树：

```csharp
private void ExpandAll(IEnumerable<LazyFolderNode> nodes)
{
    foreach (var node in nodes)
    {
        node.IsExpanded = true;
        ExpandAll(node.Children);
    }
}

private void CollapseAll(IEnumerable<LazyFolderNode> nodes)
{
    foreach (var node in nodes)
    {
        node.IsExpanded = false;
        CollapseAll(node.Children);
    }
}
```

## 搜索与筛选

你可以通过隐藏不匹配搜索条件的节点来筛选树内容。由于 `TreeView` 本身没有内置筛选机制，因此通常做法是从源数据重新构建一棵“可见树”：

```csharp
[ObservableProperty]
private string _searchText = "";

partial void OnSearchTextChanged(string value)
{
    FilteredItems.Clear();
    foreach (var root in _allItems)
    {
        var filtered = FilterNode(root, value);
        if (filtered is not null)
            FilteredItems.Add(filtered);
    }
}

private FolderNode? FilterNode(FolderNode node, string search)
{
    // 检查当前节点是否匹配
    var matches = node.Name.Contains(search, StringComparison.OrdinalIgnoreCase);

    // 递归筛选子节点
    var filteredChildren = node.Children
        .Select(c => FilterNode(c, search))
        .Where(c => c is not null)
        .ToList();

    // Include this node if it matches or has matching children
    if (matches || filteredChildren.Count > 0)
    {
        var result = new FolderNode { Name = node.Name };
        foreach (var child in filteredChildren)
            result.Children.Add(child!);
        return result;
    }

    return null;
}
```

## 在 TreeView 中使用拖放

启用拖放，以便重新排列树节点：

```xml
<TreeView ItemsSource="{Binding Items}"
          DragDrop.AllowDrop="True">
    <TreeView.Styles>
        <Style Selector="TreeViewItem">
            <Setter Property="DragDrop.AllowDrop" Value="True" />
        </Style>
    </TreeView.Styles>
</TreeView>
```

你可以在 code-behind 中处理拖拽事件，或者使用行为封装。完整 API 请参阅 [Drag and Drop](/docs/input-interaction/drag-and-drop)。

## 设置 TreeViewItem 样式

自定义树节点的显示外观：

```xml
<TreeView.Styles>
    <!-- 修改展开/折叠图标 -->
    <Style Selector="TreeViewItem:empty /template/ ToggleButton#PART_ExpandCollapseChevron">
        <Setter Property="IsVisible" Value="False" />
    </Style>

    <!-- 高亮选中项 -->
    <Style Selector="TreeViewItem:selected /template/ ContentPresenter#PART_HeaderPresenter">
        <Setter Property="Background" Value="{DynamicResource SystemAccentColor}" />
        <Setter Property="Foreground" Value="White" />
    </Style>

    <!-- 添加缩进 -->
    <Style Selector="TreeViewItem">
        <Setter Property="Padding" Value="4" />
    </Style>
</TreeView.Styles>
```

## 另请参阅

- [TreeView Control Reference](/controls/data-display/structured-data/treeview)：属性表与基础示例。
- [Data Templates](/docs/data-templates/introduction-to-data-templates)：数据模板的工作方式。
- [Drag and Drop](/docs/input-interaction/drag-and-drop)：拖放支持说明。
- [Collection Views](/docs/data-binding/collection-views)：集合筛选与排序。
