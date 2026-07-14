---
id: master-detail
title: 主从明细绑定
description: 实现主从明细模式，使选中某项时在绑定视图中显示其详细信息。
doc-type: how-to
---

主从明细模式会同时显示一个项目列表（“主列表”）以及当前选中项的详细信息。你会在邮件客户端、设置界面、文件管理器以及许多其他应用中看到这种模式。借助数据绑定和 `DataContext` 继承，Avalonia 可以比较直接地实现它。

## 基础主从明细

把 [`ListBox`](/api/avalonia/controls/listbox) 绑定到一个集合上，然后在旁边的面板中显示当前选中项的属性。详细信息面板把自己的 `DataContext` 设置为 `SelectedPerson`，因此其中的每个绑定都会以当前选中对象为解析目标：

```xml
<Grid ColumnDefinitions="250,*">
    <!-- 主列表：项目集合 -->
    <ListBox Grid.Column="0"
             ItemsSource="{Binding People}"
             SelectedItem="{Binding SelectedPerson}">
        <ListBox.ItemTemplate>
            <DataTemplate>
                <TextBlock Text="{Binding Name}" />
            </DataTemplate>
        </ListBox.ItemTemplate>
    </ListBox>

    <!-- 明细：选中项属性 -->
    <StackPanel Grid.Column="1" Margin="16"
                DataContext="{Binding SelectedPerson}"
                IsVisible="{Binding $parent[Grid].((vm:MainViewModel)DataContext).SelectedPerson,
                            Converter={x:Static ObjectConverters.IsNotNull}}">
        <TextBlock Text="{Binding Name}" FontSize="20" FontWeight="Bold" />
        <TextBlock Text="{Binding Email}" Margin="0,4,0,0" />
        <TextBlock Text="{Binding Department}" Margin="0,4,0,0" />
    </StackPanel>
</Grid>
```

对应的视图模型：

```csharp
public partial class MainViewModel : ObservableObject
{
    public ObservableCollection<Person> People { get; } = new()
    {
        new Person("Alice", "alice@example.com", "Engineering"),
        new Person("Bob", "bob@example.com", "Design"),
        new Person("Charlie", "charlie@example.com", "Marketing"),
    };

    [ObservableProperty]
    private Person? _selectedPerson;
}

public record Person(string Name, string Email, string Department);
```

:::tip
当你在明细面板上设置 `DataContext` 后，里面的每个绑定都会以当前选中项为相对目标进行解析。这样 XAML 会更简洁，因为你不需要在每个属性路径前都重复写 `SelectedPerson.`。
:::

:::note
明细面板上的 `IsVisible` 绑定会沿视觉树向上找到父级 `Grid`，并将其 `DataContext` 转换为你的视图模型类型。这样做是必要的，因为当没有选中任何项时，明细面板自己的 `DataContext` 会是 `null`，此时本地的 `IsVisible` 绑定将无法正确求值。
:::

## 可编辑的明细视图

如果你想在明细面板中使用双向绑定，请启用 `TwoWay` 模式，并确保你的模型实现了 `INotifyPropertyChanged`。如果你使用的是 MVVM Toolkit，那么 `[ObservableProperty]` 特性会自动生成所需的通知逻辑：

```csharp
public partial class Person : ObservableObject
{
    [ObservableProperty]
    private string _name;

    [ObservableProperty]
    private string _email;

    [ObservableProperty]
    private string _department;

    public Person(string name, string email, string department)
    {
        _name = name;
        _email = email;
        _department = department;
    }
}
```

```xml
<StackPanel Grid.Column="1" Margin="16"
            DataContext="{Binding SelectedPerson}">
    <TextBox Text="{Binding Name}" PlaceholderText="Name" />
    <TextBox Text="{Binding Email}" PlaceholderText="Email" Margin="0,8,0,0" />
    <TextBox Text="{Binding Department}" PlaceholderText="Department" Margin="0,8,0,0" />
</StackPanel>
```

文本框中的变化会自动更新主列表中的对应项，因为两个面板共享的是同一个对象引用。如果你的模型使用的是 `record` 类型（如前面的基础示例），那么在可编辑场景下，你就需要改用能够触发 `PropertyChanged` 通知的类。

:::warning
如果你的 `ListBox.ItemTemplate` 显示的正是你正在编辑的那个属性（例如 `Name`），那么只有当模型触发了 `PropertyChanged` 时，列表项才会实时更新。普通的 POCO 或 C# record 并不会让主列表自动刷新。
:::

## 使用独立明细视图模型的主从模式

对于复杂的明细视图，建议创建一个专门的明细视图模型，并在选择项变化时同步更新它。当你需要加载额外数据、执行验证或管理明细专属命令时，这种方式尤其合适：

```csharp
public partial class MainViewModel : ObservableObject
{
    public ObservableCollection<Person> People { get; } = new();

    [ObservableProperty]
    private Person? _selectedPerson;

    [ObservableProperty]
    private PersonDetailViewModel? _detail;

    partial void OnSelectedPersonChanged(Person? value)
    {
        Detail = value is not null ? new PersonDetailViewModel(value) : null;
    }
}

public partial class PersonDetailViewModel : ObservableObject
{
    private readonly Person _person;

    public PersonDetailViewModel(Person person)
    {
        _person = person;
        LoadDetails();
    }

    [ObservableProperty]
    private string _biography = "";

    [ObservableProperty]
    private ObservableCollection<string> _recentActivity = new();

    private void LoadDetails()
    {
        // 为当前选中的人员加载额外数据。
    }
}
```

```xml
<ContentControl Grid.Column="1" Content="{Binding Detail}">
    <ContentControl.DataTemplates>
        <DataTemplate DataType="vm:PersonDetailViewModel">
            <StackPanel Margin="16" Spacing="8">
                <TextBlock Text="{Binding Biography}" TextWrapping="Wrap" />
                <ItemsControl ItemsSource="{Binding RecentActivity}" />
            </StackPanel>
        </DataTemplate>
    </ContentControl.DataTemplates>
</ContentControl>
```

:::tip
如果你的明细数据是异步加载的，可以考虑先在构造函数中设置占位值（或者显示加载指示器），等异步操作完成后再更新属性。这样可以避免数据加载期间界面短暂闪白。
:::

## 带导航的主从明细

在移动端或紧凑布局中，明细视图通常会替代主列表显示，而不是并排出现。你可以借助可见性切换以及 `TransitioningContentControl` 来实现这种带动画的过渡：

```csharp
public partial class MainViewModel : ObservableObject
{
    public ObservableCollection<Person> People { get; } = new();

    [ObservableProperty]
    private Person? _selectedPerson;

    [ObservableProperty]
    private bool _showDetail;

    partial void OnSelectedPersonChanged(Person? value)
    {
        ShowDetail = value is not null;
    }

    [RelayCommand]
    private void GoBack()
    {
        SelectedPerson = null;
        ShowDetail = false;
    }
}
```

```xml
<Panel>
    <!-- 主列表 -->
    <ListBox ItemsSource="{Binding People}"
             SelectedItem="{Binding SelectedPerson}"
             IsVisible="{Binding !ShowDetail}" />

    <!-- 明细视图 -->
    <StackPanel IsVisible="{Binding ShowDetail}" Margin="16">
        <Button Content="Back" Command="{Binding GoBackCommand}" />
        <TextBlock Text="{Binding SelectedPerson.Name}" FontSize="20" />
    </StackPanel>
</Panel>
```

:::note
如果你希望用户返回主列表时还能保留滚动位置，就应当让 `ListBox` 保持在视觉树中（例如通过 `IsVisible` 隐藏），而不是用 `ContentControl` 直接替换掉它。被隐藏的控件会保留自身状态。
:::

## 嵌套主从明细

对于像“分类包含项目”这样的层级数据，你可以串联多个主从明细层级。每一列的 `ItemsSource` 都绑定到前一层当前选中的项：

```xml
<Grid ColumnDefinitions="200,200,*">
    <!-- Level 1: Categories -->
    <ListBox Grid.Column="0"
             ItemsSource="{Binding Categories}"
             SelectedItem="{Binding SelectedCategory}">
        <ListBox.ItemTemplate>
            <DataTemplate>
                <TextBlock Text="{Binding Name}" />
            </DataTemplate>
        </ListBox.ItemTemplate>
    </ListBox>

    <!-- Level 2: Items in category -->
    <ListBox Grid.Column="1"
             ItemsSource="{Binding SelectedCategory.Items}"
             SelectedItem="{Binding SelectedItem}">
        <ListBox.ItemTemplate>
            <DataTemplate>
                <TextBlock Text="{Binding Title}" />
            </DataTemplate>
        </ListBox.ItemTemplate>
    </ListBox>

    <!-- Level 3: Item details -->
    <StackPanel Grid.Column="2" Margin="16"
                DataContext="{Binding SelectedItem}">
        <TextBlock Text="{Binding Title}" FontSize="20" FontWeight="Bold" />
        <TextBlock Text="{Binding Description}" TextWrapping="Wrap" />
    </StackPanel>
</Grid>
```

:::warning
当用户选择新的分类时，第二层 `ListBox` 会收到新的 `ItemsSource`，并因此丢失当前选中项。如果你没有在 `SelectedCategory` 变化时于视图模型中显式清除 `SelectedItem`，就可能继续显示上一个分类遗留下来的详情内容。可以在 `OnSelectedCategoryChanged` 中将 `SelectedItem` 重置为 `null` 来处理这个问题。
:::

## 空选择占位内容

当没有选中任何项目时，可以显示提示信息或图形，以避免详情区域看起来一片空白：

```xml
<Panel Grid.Column="1">
    <!-- Shown when nothing is selected -->
    <TextBlock Text="Select an item to view details"
               HorizontalAlignment="Center"
               VerticalAlignment="Center"
               Foreground="Gray"
               IsVisible="{Binding SelectedPerson,
                           Converter={x:Static ObjectConverters.IsNull}}" />

    <!-- Detail panel -->
    <StackPanel DataContext="{Binding SelectedPerson}"
                IsVisible="{Binding $parent[Panel].((vm:MainViewModel)DataContext).SelectedPerson,
                            Converter={x:Static ObjectConverters.IsNotNull}}">
        <TextBlock Text="{Binding Name}" FontSize="20" />
    </StackPanel>
</Panel>
```

你可以将这里的占位 `TextBlock` 替换成图片、图标，或任何符合你应用设计风格的自定义布局。

## 另请参阅

- [Bind to a collection](/docs/data-binding/how-to-bind-to-a-collection): Binding `ItemsSource` and `DataTemplate` usage.
- [Data templates](/docs/data-templates/introduction-to-data-templates): Controlling how items are displayed.
- [Data context](/docs/data-binding/data-context): How `DataContext` flows through the control tree.
- [Collection views](/docs/data-binding/collection-views): Sorting, filtering, and grouping bound collections.
- [Compiled bindings](/docs/data-binding/compiled-bindings): Improve binding performance and catch errors at compile time.
