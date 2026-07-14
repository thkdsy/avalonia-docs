---
id: filtering
title: 筛选
description: 了解如何在 Avalonia TreeDataGrid 控件中使用谓词函数对行进行筛选，包括多条件、枚举、空值安全和分层筛选模式。
doc-type: reference
tags:
  - avalonia pro
  - avalonia enterprise
---

:::info
该控件可作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分使用。
:::

筛选可让你的 `TreeDataGrid` 只显示符合特定条件的行。`FlatTreeDataGridSource` 和 `HierarchicalTreeDataGridSource` 都支持通过谓词函数进行筛选。

:::note
筛选需要使用代码后置 `Source` 方式。目前还没有与之等价的 XAML 筛选写法。你必须在视图模型中创建 `FlatTreeDataGridSource` 或 `HierarchicalTreeDataGridSource`。
:::

## 启用筛选

你可以在 `FlatTreeDataGridSource` 或 `HierarchicalTreeDataGridSource` 上调用 `Filter` 方法并传入谓词函数来启用筛选。该谓词会接收每个模型项；若返回 `true`，则该项可见；若返回 `false`，则该项被隐藏。

### 基本字符串筛选

若要根据 `_filterString` 中的值，对具有字符串 `Name` 属性的模型进行筛选：

```csharp
Source.Filter(x => x.Name.Contains(_filterString, StringComparison.CurrentCultureIgnoreCase));
```

### 多条件筛选

你可以在筛选谓词中组合多个条件：

```csharp
// 按姓名和最小年龄同时筛选
Source.Filter(x =>
    x.Name.Contains(_filterString, StringComparison.CurrentCultureIgnoreCase) &&
    x.Age >= _minimumAge);

// 按名字、姓氏或邮箱筛选
Source.Filter(x =>
    x.FirstName.Contains(_searchText, StringComparison.CurrentCultureIgnoreCase) ||
    x.LastName.Contains(_searchText, StringComparison.CurrentCultureIgnoreCase) ||
    x.Email.Contains(_searchText, StringComparison.CurrentCultureIgnoreCase));
```

### 枚举和类别筛选

当模型包含枚举或类别属性时，你可以根据选中的值进行筛选：

```csharp
// 按选中的部门枚举进行筛选
Source.Filter(x => _selectedDepartment == null || x.Department == _selectedDepartment);
```

### 空值安全筛选

如果你要筛选的属性可能为 `null`，则应在谓词中防止出现 `NullReferenceException`：

```csharp
Source.Filter(x =>
    (x.Name?.Contains(_filterString, StringComparison.CurrentCultureIgnoreCase) ?? false) ||
    (x.Email?.Contains(_filterString, StringComparison.CurrentCultureIgnoreCase) ?? false));
```

### 复杂筛选

筛选谓词可以使用任意 C# 表达式，包括 LINQ 方法和辅助函数：

```csharp
// 使用 LINQ 方法筛选
Source.Filter(x => x.Tags.Any(tag => tag.Contains(_filterText)));

// 使用辅助方法筛选
Source.Filter(x => IsMatchingCriteria(x));

private bool IsMatchingCriteria(Person person)
{
    if (string.IsNullOrWhiteSpace(_filterText))
        return true;

    return person.Name.Contains(_filterText, StringComparison.CurrentCultureIgnoreCase) ||
           person.Department.Contains(_filterText, StringComparison.CurrentCultureIgnoreCase);
}
```

## 更新筛选器

当筛选谓词依赖外部变量时（例如上面示例中的 `_filterString`），每当这些变量发生变化，你都需要刷新筛选器。调用 `RefreshFilter` 可以让系统重新对每个项目求值：

```csharp
private string _filterString = string.Empty;

public string FilterString
{
    get => _filterString;
    set
    {
        _filterString = value;
        Source.RefreshFilter();
    }
}
```

:::info
当你调用 `Filter` 以替换谓词本身时，不需要再调用 `RefreshFilter`。只有当现有谓词依赖的外部变量发生变化时，才需要调用它。
:::

## 清除筛选

若要移除筛选并重新显示所有项目，请向 `Filter` 方法传入 `null`：

```csharp
Source.Filter(null);
```

当用户清空搜索框或重置筛选控件时，这种做法非常有用。

## 分层数据筛选

当你使用 `HierarchicalTreeDataGridSource` 对分层数据进行筛选时，谓词会在层级结构的每一层对每个项目独立求值。每个项目是否显示，仅取决于它自己是否匹配筛选条件。控件不会因为子项匹配就自动显示父项，反之亦然。

:::warning
对大型分层树进行筛选可能代价较高，因为必须访问每个节点。如果你担心性能问题，可以考虑将筛选状态构建到数据模型中，以便跳过整棵子树。
:::

### 保持父项可见

如果你希望只要任一子项匹配筛选条件，其父项也保持可见，那么需要你自行实现这部分逻辑。一种做法是预先计算一个匹配 ID 集合（包括祖先 ID），然后在谓词中检查成员关系：

```csharp
var matchingIds = new HashSet<int>();

void CollectMatches(IEnumerable<TreeNode> nodes)
{
    foreach (var node in nodes)
    {
        if (node.Name.Contains(_filterText, StringComparison.CurrentCultureIgnoreCase))
        {
            // 将该节点及其所有祖先加入集合
            var current = node;
            while (current != null)
            {
                matchingIds.Add(current.Id);
                current = current.Parent;
            }
        }

        CollectMatches(node.Children);
    }
}

CollectMatches(_rootNodes);
Source.Filter(x => matchingIds.Contains(x.Id));
```

## 性能注意事项

筛选操作在 UI 线程上运行，并会重新计算数据源中的每一项。对于大型数据集，请注意以下建议：

- **对用户输入进行节流。** 当筛选由 [`TextBox`](/api/avalonia/controls/textbox) 驱动时，可使用 `Observable.Throttle` 或延迟计时器，避免每次按键都运行谓词。
- **保持谓词足够轻量。** 避免在谓词内部执行分配、正则表达式或数据库调用。
- **预先计算高成本值。** 将可搜索文本保存到专用属性中，这样谓词只需进行简单的字符串比较。
- **对于超大集合优先上游筛选。** 如果数据集有成千上万行，可考虑先筛选底层集合，再将结果绑定到网格。

### 节流筛选示例

你可以使用 Reactive Extensions 对来自 `TextBox` 的筛选更新进行节流：

```csharp
this.WhenAnyValue(x => x.SearchText)
    .Throttle(TimeSpan.FromMilliseconds(300))
    .ObserveOn(RxApp.MainThreadScheduler)
    .Subscribe(_ => ApplyFilter());
```

## 完整示例

下面的示例将搜索 `TextBox` 连接到 `FlatTreeDataGridSource<Person>` 的筛选逻辑。

**视图模型：**

```csharp
public class PersonListViewModel : ViewModelBase
{
    private readonly ObservableCollection<Person> _allPeople;
    private string _searchText = string.Empty;

    public FlatTreeDataGridSource<Person> Source { get; }

    public string SearchText
    {
        get => _searchText;
        set
        {
            if (_searchText != value)
            {
                _searchText = value;
                OnPropertyChanged();
                ApplyFilter();
            }
        }
    }

    public PersonListViewModel()
    {
        _allPeople = new ObservableCollection<Person>
        {
            new Person { Name = "John Doe", Age = 30, Department = "IT" },
            new Person { Name = "Jane Smith", Age = 25, Department = "HR" },
            new Person { Name = "Bob Johnson", Age = 35, Department = "IT" },
        };

        Source = new FlatTreeDataGridSource<Person>(_allPeople)
            .WithTextColumn(x => x.Name)
            .WithTextColumn(x => x.Age)
            .WithTextColumn(x => x.Department);
    }

    private void ApplyFilter()
    {
        if (string.IsNullOrWhiteSpace(_searchText))
        {
            Source.Filter(null);
        }
        else
        {
            Source.Filter(x =>
                x.Name.Contains(_searchText, StringComparison.CurrentCultureIgnoreCase) ||
                x.Department.Contains(_searchText, StringComparison.CurrentCultureIgnoreCase));
        }
    }
}
```

**视图：**

```xml
<StackPanel>
    <TextBox Text="{Binding SearchText}"
             PlaceholderText="Search..."
             Margin="0,0,0,10" />
    <TreeDataGrid Source="{Binding Source}"
                  Height="400" />
</StackPanel>
```

## 另请参阅

- [TreeDataGrid 概览](/controls/data-display/structured-data/treedatagrid/)
- [列类型](/controls/data-display/structured-data/treedatagrid/column-types)
- [排序](/controls/data-display/structured-data/treedatagrid/sorting)
- [选择模式](/controls/data-display/structured-data/treedatagrid/selection-modes)
- [展开与折叠操作](/controls/data-display/structured-data/treedatagrid/expand-and-collapse)
