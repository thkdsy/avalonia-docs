---
id: treeview
title: TreeView
---

import TreeViewAnimalHierarchyScreenshot from '/img/controls/treeview/treeview-animal-hierarchy.gif';
import TreeViewEnhancedAnimalHierarchyScreenshot from '/img/controls/treeview/treeview-enhanced-animal-hierarchy.gif';

`TreeView` 控件可用于显示分层数据，并允许选择项目。项目使用模板呈现，因此你可以自定义它们的显示方式。

这里有两个数据源：一个是控件本身的主项目源，它提供分层数据的根节点；另一个是项目模板中的项目源，它允许控件列出分层数据的下一层级。

## 常用属性

你最常使用的通常是这些属性：

<table>
  <thead>
    <tr>
      <th width="316">属性</th>
      <th>说明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>ItemsSource</code></td>
      <td>作为控件数据源使用的绑定集合。</td>
    </tr>
    <tr>
      <td><code>ItemsControl.ItemTemplate</code></td>
      <td>项目模板，包含一个会应用到各个项目上的 DataTemplate，可用于更改项目外观。</td>
    </tr>
    <tr>
      <td><code>ItemsControl.ItemsPanel</code></td>
      <td>用于放置项目的容器面板。默认值为 StackPanel。有关自定义 ItemsPanel 的方法，请参阅[此页面](/docs/custom-controls/custom-itemspanel)。</td>
    </tr>
    <tr>
      <td><code>ItemsControl.Styles</code></td>
      <td>应用到 ItemControl 任意子元素上的样式。</td>
    </tr>
  </tbody>
</table>

## 示例

此示例使用 MVVM 模式的视图模型来保存基于 C# 节点类的分层数据。在这个例子中，视图模型的 `Nodes` 集合中只有一个根节点：

```xml
<TreeView ItemsSource="{Binding Nodes}">
  <TreeView.ItemTemplate>
    <TreeDataTemplate ItemsSource="{Binding SubNodes}">
      <TextBlock Text="{Binding Title}"/>
    </TreeDataTemplate>
  </TreeView.ItemTemplate>
</TreeView>
```

```csharp title='C# View Model'
using AvaloniaControls.Models;
using System.Collections.ObjectModel;

namespace AvaloniaControls.ViewModels
{
    public class MainWindowViewModel : ViewModelBase
    {
        public ObservableCollection<Node> Nodes{ get; }

        public MainWindowViewModel()
        {
            Nodes = new ObservableCollection<Node>
            {                
                new Node("Animals", new ObservableCollection<Node>
                {
                    new Node("Mammals", new ObservableCollection<Node>
                    {
                        new Node("Lion"), new Node("Cat"), new Node("Zebra")
                    })
                })
            };
        }
    }
}
```

```csharp title='C# Node Class'
using System.Collections.ObjectModel;

namespace AvaloniaControls.Models
{
    public class Node
    {
        public ObservableCollection<Node>? SubNodes { get; }
        public string Title { get; }
  
        public Node(string title)
        {
            Title = title;
        }

        public Node(string title, ObservableCollection<Node> subNodes)
        {
            Title = title;
            SubNodes = subNodes;
        }
    }
}
```

默认情况下会显示根节点（或多个根节点）。用户可以点击相邻的箭头来展开或收起各个节点。点击节点标题会选中该项目。在触摸和手写笔设备上，选中操作会在指针释放时发生，而不是按下时发生，这样用户就可以从节点上开始滚动手势而不会改变当前选中项。

<Image light={TreeViewAnimalHierarchyScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

这是前一个示例的扩展示例，包含多个根节点、修改后的项目模板，以及在视图模型代码中设置的初始选中项：

```xml
<TreeView Margin="10"
          ItemsSource="{Binding Nodes}" 
          SelectedItems="{Binding SelectedNodes}"
          SelectionMode="Multiple">
  <TreeView.ItemTemplate>
    <TreeDataTemplate ItemsSource="{Binding SubNodes}">
      <Border HorizontalAlignment="Left"
              BorderBrush="Gray" BorderThickness="1"
              CornerRadius="5" Padding="15 3">
        <TextBlock Text="{Binding Title}" />
      </Border>
    </TreeDataTemplate>
  </TreeView.ItemTemplate>
</TreeView>
```

```csharp title='C# View Model'
using AvaloniaControls.Models;
using System.Collections.ObjectModel;
using System.Linq;

namespace AvaloniaControls.ViewModels
{
    public class MainWindowViewModel : ViewModelBase
    {
        public ObservableCollection<Node> Nodes { get; }
        public ObservableCollection<Node> SelectedNodes { get; }

        public MainWindowViewModel()
        {
            SelectedNodes = new ObservableCollection<Node>();
            Nodes = new ObservableCollection<Node>
            {                
                new Node("Animals", new ObservableCollection<Node>
                {
                    new Node("Mammals", new ObservableCollection<Node>
                    {
                        new Node("Lion"), new Node("Cat"), new Node("Zebra")
                    })
                }),
                new Node("Birds", new ObservableCollection<Node>
                {
                    new Node("Robin"), new Node("Condor"), 
                    new Node("Parrot"), new Node("Eagle")
                }),
                new Node("Insects", new ObservableCollection<Node>
                {
                    new Node("Locust"), new Node("House Fly"), 
                    new Node("Butterfly"), new Node("Moth")
                }),
            };

            var moth = Nodes.Last().SubNodes?.Last();
            if (moth!=null) SelectedNodes.Add(moth);    
        }
    }
}
```

```csharp title='C# Node Class'
using System.Collections.ObjectModel;

namespace AvaloniaControls.Models
{
    public class Node
    {
        public ObservableCollection<Node>? SubNodes { get; }
        public string Title { get; }
  
        public Node(string title)
        {
            Title = title;
        }

        public Node(string title, ObservableCollection<Node> subNodes)
        {
            Title = title;
            SubNodes = subNodes;
        }
    }
}
```

树视图会在需要时自动添加滚动条。按住 Ctrl 键可以扩展选择。

<Image light={TreeViewEnhancedAnimalHierarchyScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 展开事件

当项目被展开或折叠时，`TreeViewItem` 会引发 `Expanded` 和 `Collapsed` 路由事件。这些事件会冒泡，因此你可以在 `TreeView` 级别统一处理它们：

```csharp
treeView.AddHandler(TreeViewItem.ExpandedEvent, (sender, args) =>
{
    var item = (TreeViewItem)args.Source!;
    // 响应展开，例如按需加载子数据
});

treeView.AddHandler(TreeViewItem.CollapsedEvent, (sender, args) =>
{
    var item = (TreeViewItem)args.Source!;
    // 响应折叠
});
```

## 另请参阅

- [TreeView API 参考](/api/avalonia/controls/treeview)
- [`TreeView.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/TreeView.cs)
