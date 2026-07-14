---
id: how-to-bind-to-a-collection
title: 如何绑定到集合
description: 将 ObservableCollection 绑定到列表控件，以便在添加、删除或修改项时自动更新 UI。
doc-type: how-to
---

当你的应用程序需要显示动态项目列表时，通常会把某个集合属性绑定到列表控件上，例如 [`ListBox`](/api/avalonia/controls/listbox)、[`ItemsControl`](/api/avalonia/controls/itemscontrol) 或 `ComboBox`。借助 `ObservableCollection<T>`，只要你添加、删除或重新排序项目，UI 就会保持同步。本指南将带你了解 Avalonia 中集合绑定的常见场景。

## 为什么使用 `ObservableCollection<T>`

普通的 `List<T>` 在内容发生变化时不会通知 UI。如果你在运行时向 `List<T>` 中添加一项，控件并不会自动更新。`ObservableCollection<T>` 实现了 `INotifyCollectionChanged`，会触发一系列事件，而 Avalonia 会监听这些事件，从而让绑定控件自动刷新。

在以下情况下应优先使用 `ObservableCollection<T>`：

- 初次加载之后，集合中的项还会被添加或删除。
- 你希望 UI 自动反映变化，而无需手动重新绑定。

如果你的集合是静态的（只加载一次且之后不再修改），那么普通的 `List<T>` 或数组就足够了。

## 绑定到简单的 `ObservableCollection`

先来看一个把 `ObservableCollection<string>` 绑定到 `ListBox` 的例子。

在视图模型中定义这个集合：

```csharp
public class MainViewModel : ObservableObject
{
    private ObservableCollection<string> _items;

    public ObservableCollection<string> Items
    {
        get => _items;
        set => SetProperty(ref _items, value);
    }

    public MainViewModel()
    {
        Items = new ObservableCollection<string> { "Item 1", "Item 2", "Item 3" };
    }
}
```

然后在 AXAML 中把该集合绑定到 `ListBox`：

```xml
<ListBox ItemsSource="{Binding Items}" />
```

当你在视图模型中调用 `Items.Add("Item 4")` 时，`ListBox` 会立即显示这一新项。

## 绑定到复杂对象集合

当集合中存放的是带有多个属性的对象时，应使用 `DataTemplate` 来控制每一项的显示方式。若想让集合中单个对象的属性变化也能反映到 UI 上，那么这些项类型本身也必须实现变更通知。

定义一个继承自 `ObservableObject` 的 `Person` 类：

```csharp
public class Person : ObservableObject
{
    private string _name;
    private int _age;

    public string Name
    {
        get => _name;
        set => SetProperty(ref _name, value);
    }

    public int Age
    {
        get => _age;
        set => SetProperty(ref _age, value);
    }
}
```

在视图模型中公开一个 `ObservableCollection<Person>`：

```csharp
public class MainViewModel : ObservableObject
{
    private ObservableCollection<Person> _people;

    public ObservableCollection<Person> People
    {
        get => _people;
        set => SetProperty(ref _people, value);
    }

    public MainViewModel()
    {
        People = new ObservableCollection<Person>
        {
            new Person { Name = "John Doe", Age = 30 },
            new Person { Name = "Jane Doe", Age = 28 }
        };
    }
}
```

然后把该集合通过 `DataTemplate` 绑定到 `ListBox`：

```xml
<ListBox ItemsSource="{Binding People}">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal">
                <TextBlock Text="{Binding Name}" Margin="0,0,10,0" />
                <TextBlock Text="{Binding Age}" />
            </StackPanel>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

这样每个 `Person` 都会并排显示其 `Name` 和 `Age`。由于 `Person` 继承自 `ObservableObject`，因此当你在代码中修改某个人的 `Name` 或 `Age` 时，对应的 `ListBox` 行也会自动更新，无需额外处理。

## 在运行时添加和删除项

一种常见模式，是将集合绑定与命令结合起来，让用户可以主动添加或删除条目：

```csharp
public class MainViewModel : ObservableObject
{
    public ObservableCollection<string> Items { get; } = new();

    public ICommand AddItemCommand { get; }
    public ICommand RemoveItemCommand { get; }

    public MainViewModel()
    {
        Items.Add("First item");

        AddItemCommand = new RelayCommand(() =>
        {
            Items.Add($"Item {Items.Count + 1}");
        });

        RemoveItemCommand = new RelayCommand(() =>
        {
            if (Items.Count > 0)
                Items.RemoveAt(Items.Count - 1);
        },
        () => Items.Count > 0);
    }
}
```

```xml
<DockPanel>
    <StackPanel DockPanel.Dock="Top" Orientation="Horizontal" Spacing="8" Margin="0,0,0,8">
        <Button Content="Add" Command="{Binding AddItemCommand}" />
        <Button Content="Remove" Command="{Binding RemoveItemCommand}" />
    </StackPanel>
    <ListBox ItemsSource="{Binding Items}" />
</DockPanel>
```

## 对不可选列表使用 `ItemsControl`

如果你不需要选择行为，可以使用 `ItemsControl` 代替 `ListBox`。它会渲染每一项，但不会提供选中高亮或键盘导航：

```xml
<ItemsControl ItemsSource="{Binding People}">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <Border Padding="8" Margin="0,0,0,4" Background="#f0f0f0" CornerRadius="4">
                <TextBlock Text="{Binding Name}" />
            </Border>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
```

## 常见陷阱

| 问题 | 原因 | 解决方案 |
|---|---|---|
| 添加项后 UI 不更新 | 使用了 `List<T>` 而不是 `ObservableCollection<T>` | 改用 `ObservableCollection<T>` |
| 集合中某项属性变化后 UI 不更新 | 项类型没有实现 `INotifyPropertyChanged` | 让项类型继承 `ObservableObject` 或自行实现 `INotifyPropertyChanged` |
| 替换整个集合后 UI 不更新 | 持有集合的属性本身没有变更通知 | 对该属性使用 `SetProperty`（或 `[ObservableProperty]` 特性） |

## 另请参阅

- [Collection views](/docs/data-binding/collection-views): Sort, filter, and group bound collections.
- [Master-detail binding](/docs/data-binding/master-detail): Display details for the selected item in a list.
- [Data templates](/docs/data-templates/introduction-to-data-templates): Control how items are displayed.
- [INotifyPropertyChanged](/docs/data-binding/inotifypropertychanged): Change notification for view models.
- [Binding to commands](/docs/data-binding/binding-to-commands): Wire up buttons and other actions.








