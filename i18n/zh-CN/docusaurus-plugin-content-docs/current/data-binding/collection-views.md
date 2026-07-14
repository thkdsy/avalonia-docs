---
id: collection-views
title: 集合的排序、筛选与分组
description: 使用 DataGridCollectionView 和 DynamicData 对绑定集合进行排序、筛选和分组。
doc-type: how-to
---

Avalonia 并没有像 WPF 那样内置一个等价的 `ICollectionView`。因此，排序、筛选和分组通常会在绑定到控件之前，由视图模型先行处理。这种方式能让 UI 层保持简单，也更便于测试相关逻辑。

## 筛选集合

最常见的模式，是使用一个能够响应筛选条件变化的派生集合。你可以借助 LINQ，或者一个类似 `CollectionViewSource` 的包装器来实现：

### 使用 ObservableCollection 手动筛选

```csharp
public partial class MainViewModel : ObservableObject
{
    private readonly ObservableCollection<Person> _allPeople;

    [ObservableProperty]
    private string _searchText = "";

    public ObservableCollection<Person> FilteredPeople { get; } = new();

    public MainViewModel()
    {
        _allPeople = new ObservableCollection<Person>
        {
            new("Alice", 30),
            new("Bob", 25),
            new("Charlie", 35),
        };

        ApplyFilter();
    }

    partial void OnSearchTextChanged(string value)
    {
        ApplyFilter();
    }

    private void ApplyFilter()
    {
        FilteredPeople.Clear();
        var filtered = string.IsNullOrEmpty(SearchText)
            ? _allPeople
            : _allPeople.Where(p =>
                p.Name.Contains(SearchText, StringComparison.OrdinalIgnoreCase));

        foreach (var person in filtered)
            FilteredPeople.Add(person);
    }
}
```

```xml
<StackPanel Spacing="8">
    <TextBox Text="{Binding SearchText}" PlaceholderText="Search..." />
    <ListBox ItemsSource="{Binding FilteredPeople}">
        <ListBox.ItemTemplate>
            <DataTemplate>
                <TextBlock Text="{Binding Name}" />
            </DataTemplate>
        </ListBox.ItemTemplate>
    </ListBox>
</StackPanel>
```

### 使用 DynamicData（推荐用于复杂场景）

[DynamicData](https://github.com/reactivemarbles/DynamicData) 库提供了响应式集合转换能力，并且与 Avalonia 的响应式模型能够很好地集成：

```csharp
using DynamicData;
using DynamicData.Binding;

public class MainViewModel : ObservableObject
{
    private readonly SourceList<Person> _source = new();
    private readonly ReadOnlyObservableCollection<Person> _filtered;

    [ObservableProperty]
    private string _searchText = "";

    public ReadOnlyObservableCollection<Person> FilteredPeople => _filtered;

    public MainViewModel()
    {
        _source.AddRange(new[]
        {
            new Person("Alice", 30),
            new Person("Bob", 25),
            new Person("Charlie", 35),
        });

        var filterPredicate = this.WhenPropertyChanged(x => x.SearchText)
            .Select(x => CreateFilter(x.Value));

        _source.Connect()
            .Filter(filterPredicate)
            .Sort(SortExpressionComparer<Person>.Ascending(p => p.Name))
            .Bind(out _filtered)
            .Subscribe();
    }

    private static Func<Person, bool> CreateFilter(string? searchText)
    {
        if (string.IsNullOrEmpty(searchText))
            return _ => true;

        return person =>
            person.Name.Contains(searchText, StringComparison.OrdinalIgnoreCase);
    }
}
```

当项目被添加、移除，或者筛选文本发生变化时，DynamicData 会自动更新 `FilteredPeople`。

## 对集合排序

### 简单排序

在绑定之前，先对源集合进行排序：

```csharp
public ObservableCollection<Person> People { get; }

public MainViewModel()
{
    var sorted = _rawData.OrderBy(p => p.Name);
    People = new ObservableCollection<Person>(sorted);
}
```

### 动态排序

你可以用属性来控制排序方式：

```csharp
[ObservableProperty]
private string _sortProperty = "Name";

[ObservableProperty]
private bool _sortDescending = false;

partial void OnSortPropertyChanged(string value) => ApplySort();
partial void OnSortDescendingChanged(bool value) => ApplySort();

private void ApplySort()
{
    var sorted = SortProperty switch
    {
        "Name" => SortDescending
            ? _allPeople.OrderByDescending(p => p.Name)
            : _allPeople.OrderBy(p => p.Name),
        "Age" => SortDescending
            ? _allPeople.OrderByDescending(p => p.Age)
            : _allPeople.OrderBy(p => p.Age),
        _ => _allPeople.AsEnumerable()
    };

    People.Clear();
    foreach (var person in sorted)
        People.Add(person);
}
```

```xml
<StackPanel Spacing="8">
    <StackPanel Orientation="Horizontal" Spacing="8">
        <ComboBox SelectedItem="{Binding SortProperty}">
            <ComboBoxItem Content="Name" />
            <ComboBoxItem Content="Age" />
        </ComboBox>
        <ToggleButton Content="Descending" IsChecked="{Binding SortDescending}" />
    </StackPanel>
    <ListBox ItemsSource="{Binding People}" />
</StackPanel>
```

### 使用 DynamicData

```csharp
_source.Connect()
    .Sort(SortExpressionComparer<Person>.Ascending(p => p.Name))
    .Bind(out _sorted)
    .Subscribe();
```

## 分组

Avalonia 的 `ItemsControl` 并不像 WPF 的 `CollectionViewSource` 那样内置分组支持。要显示分组数据，通常做法是把各组拍平成一个单一集合，并在其中加入组头项。

:::tip
`DataGrid` 控件则支持通过 `DataGridCollectionView` 进行内置分组。详情请参阅 [DataGrid grouping how-to](/docs/how-to/datagrid-how-to#grouping)。
:::

### 使用带组头的扁平列表

创建一个同时表示组头和具体项的视图模型：

```csharp
public abstract class ListItem { }

public class GroupHeader : ListItem
{
    public string Title { get; }
    public GroupHeader(string title) => Title = title;
}

public class PersonItem : ListItem
{
    public Person Person { get; }
    public PersonItem(Person person) => Person = person;
}
```

Build the grouped list:

```csharp
public ObservableCollection<ListItem> GroupedPeople { get; } = new();

private void BuildGroups()
{
    GroupedPeople.Clear();
    var groups = _allPeople.GroupBy(p => p.Age / 10 * 10); // Group by decade

    foreach (var group in groups.OrderBy(g => g.Key))
    {
        GroupedPeople.Add(new GroupHeader($"{group.Key}s"));
        foreach (var person in group.OrderBy(p => p.Name))
            GroupedPeople.Add(new PersonItem(person));
    }
}
```

Use a `DataTemplateSelector` (via `DataTemplate` with `DataType`) to render headers and items differently:

```xml
<ListBox ItemsSource="{Binding GroupedPeople}">
    <ListBox.DataTemplates>
        <DataTemplate DataType="local:GroupHeader">
            <TextBlock Text="{Binding Title}"
                       FontWeight="Bold" FontSize="14"
                       Margin="0,8,0,4" />
        </DataTemplate>
        <DataTemplate DataType="local:PersonItem">
            <StackPanel Orientation="Horizontal" Spacing="8" Margin="12,0,0,0">
                <TextBlock Text="{Binding Person.Name}" />
                <TextBlock Text="{Binding Person.Age}" Foreground="Gray" />
            </StackPanel>
        </DataTemplate>
    </ListBox.DataTemplates>
</ListBox>
```

### With DynamicData GroupOn

```csharp
_source.Connect()
    .GroupOn(p => p.Department)
    .Transform(group => new DepartmentGroup(group.Key, group.List))
    .Bind(out _groups)
    .Subscribe();
```

## Best practices

- Keep filtering, sorting, and grouping logic in the view model, not in code-behind.
- For large collections, use DynamicData for efficient reactive updates instead of rebuilding the collection on every change.
- When sorting or filtering changes, avoid clearing and re-adding if possible. DynamicData handles incremental updates automatically.
- Use `ReadOnlyObservableCollection<T>` for the public property to prevent external modification.
- Consider debouncing filter input (e.g., with `Throttle`) for search boxes that filter on every keystroke.

## See also

- [How to Bind to a Collection](/docs/data-binding/how-to-bind-to-a-collection): Basic collection binding patterns.
- [Data Templates](/docs/data-templates/introduction-to-data-templates): Controlling how items are rendered.
- [INotifyPropertyChanged](/docs/data-binding/inotifypropertychanged): Change notification for view models.
