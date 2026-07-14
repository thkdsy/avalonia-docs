---
id: tabcontrol-how-to
title: "如何：使用 TabControl"
description: 学习如何使用 Avalonia TabControl 创建静态标签页、动态标签页、可关闭标签页以及自定义样式标签页。
doc-type: how-to
---

本指南将带你了解常见的 `TabControl` 使用场景，包括静态标签页、动态绑定标签页、可关闭标签页、带图标的标题、标签位置以及样式设置。

## 静态标签页

最简单的方式，是直接在 XAML 中使用 [`TabItem`](/api/avalonia/controls/tabitem) 元素定义标签页。每个 `TabItem` 都包含一个 `Header`（标签标题）和一段子内容（标签被选中时显示的内容）。

```xml
<TabControl>
    <TabItem Header="General">
        <StackPanel Margin="16" Spacing="8">
            <TextBlock Text="General settings content" />
        </StackPanel>
    </TabItem>
    <TabItem Header="Appearance">
        <StackPanel Margin="16" Spacing="8">
            <TextBlock Text="Appearance settings content" />
        </StackPanel>
    </TabItem>
    <TabItem Header="Advanced">
        <StackPanel Margin="16" Spacing="8">
            <TextBlock Text="Advanced settings content" />
        </StackPanel>
    </TabItem>
</TabControl>
```

这种方式非常适合在设计阶段就已经确定标签页数量的场景。如果你需要在运行时动态添加或移除标签页，请使用下面介绍的动态方式。

## 从集合生成动态标签页

如果你希望标签页由数据驱动，就可以把 `ItemsSource` 绑定到视图模型中的 `ObservableCollection`。这样不仅可以在运行时动态增删标签页，也能让 UI 和逻辑保持分离。

### 步骤 1：创建标签项视图模型

先定义一个表示单个标签页的简单类：

```csharp
public class TabItemViewModel
{
    public string Header { get; }
    public object Content { get; }

    public TabItemViewModel(string header, object content)
    {
        Header = header;
        Content = content;
    }
}
```

### 步骤 2：在主视图模型中公开集合

```csharp
public partial class MainViewModel : ObservableObject
{
    public ObservableCollection<TabItemViewModel> Tabs { get; } = new()
    {
        new TabItemViewModel("Home", new HomeViewModel()),
        new TabItemViewModel("Settings", new SettingsViewModel()),
    };

    [ObservableProperty]
    private TabItemViewModel? _selectedTab;
}
```

### 步骤 3：在 XAML 中进行绑定

使用 `ItemTemplate` 定义标签头的显示方式，再用 `ContentTemplate` 定义标签页主体内容：

```xml
<TabControl ItemsSource="{Binding Tabs}"
            SelectedItem="{Binding SelectedTab}">
    <TabControl.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Header}" />
        </DataTemplate>
    </TabControl.ItemTemplate>
    <TabControl.ContentTemplate>
        <DataTemplate>
            <ContentControl Content="{Binding Content}" />
        </DataTemplate>
    </TabControl.ContentTemplate>
</TabControl>
```

:::tip
如果每个标签页的 `Content` 都是不同类型的视图模型，你可以使用 `DataTemplateSelector`，或者在 `Application.DataTemplates` 中定义对应的 `DataTemplate`，这样每种视图模型都能自动解析为正确的视图。
:::

## 可关闭标签页

你可以在每个标签头中添加关闭按钮，让用户自行关闭标签页。下面这个关闭按钮的 `Command` 使用了祖先绑定，以便访问 `MainViewModel`。

```xml
<TabControl ItemsSource="{Binding Tabs}"
            SelectedItem="{Binding SelectedTab}">
    <TabControl.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal" Spacing="8">
                <TextBlock Text="{Binding Header}" VerticalAlignment="Center" />
                <Button Content="x" FontSize="10" Padding="4,2"
                        Background="Transparent" BorderThickness="0"
                        Command="{Binding $parent[TabControl].((vm:MainViewModel)DataContext).CloseTabCommand}"
                        CommandParameter="{Binding}" />
            </StackPanel>
        </DataTemplate>
    </TabControl.ItemTemplate>
    <TabControl.ContentTemplate>
        <DataTemplate>
            <ContentControl Content="{Binding Content}" />
        </DataTemplate>
    </TabControl.ContentTemplate>
</TabControl>
```

在视图模型中，你需要处理移除逻辑，并在必要时更新当前选中项，避免用户关闭标签后停留在空白页：

```csharp
[RelayCommand]
private void CloseTab(TabItemViewModel tab)
{
    Tabs.Remove(tab);
    if (SelectedTab == tab)
        SelectedTab = Tabs.FirstOrDefault();
}
```

:::note
祖先绑定中使用的 XAML 命名空间 `vm` 必须与你的视图模型命名空间一致。例如，可以在 XAML 根元素上添加 `xmlns:vm="using:MyApp.ViewModels"`。
:::

:::tip
如果你不希望最后一个标签页也被关闭，可以在移除前先检查 `Tabs.Count`：

```csharp
if (Tabs.Count > 1)
    Tabs.Remove(tab);
```
:::

## 带图标的标签页

你可以自定义标签头，在文本旁边加入图标。

### 带图标的静态标签页

可以把 `TabItem.Header` 设置为一个同时包含 `PathIcon` 和 `TextBlock` 的面板：

```xml
<TabItem>
    <TabItem.Header>
        <StackPanel Orientation="Horizontal" Spacing="6">
            <PathIcon Data="{StaticResource HomeIcon}" Width="14" Height="14" />
            <TextBlock Text="Home" />
        </StackPanel>
    </TabItem.Header>
    <views:HomeView />
</TabItem>
```

### 带图标的动态标签页

如果你的 `TabItemViewModel` 暴露了一个 `StreamGeometry` 类型的 `IconData` 属性，就可以在 `ItemTemplate` 中直接使用它：

```xml
<TabControl.ItemTemplate>
    <DataTemplate>
        <StackPanel Orientation="Horizontal" Spacing="6">
            <PathIcon Data="{Binding IconData}" Width="14" Height="14" />
            <TextBlock Text="{Binding Header}" />
        </StackPanel>
    </DataTemplate>
</TabControl.ItemTemplate>
```

## 标签位置

默认情况下，标签会显示在顶部。你可以使用 `TabStripPlacement` 属性调整它们的位置。可用值包括 `Top`、`Bottom`、`Left` 和 `Right`。

```xml
<!-- 左侧标签（垂直布局） -->
<TabControl TabStripPlacement="Left">
    <TabItem Header="Page 1"><TextBlock Text="Content 1" /></TabItem>
    <TabItem Header="Page 2"><TextBlock Text="Content 2" /></TabItem>
</TabControl>

<!-- 底部标签 -->
<TabControl TabStripPlacement="Bottom">
    <TabItem Header="Tab 1"><TextBlock Text="Content 1" /></TabItem>
</TabControl>
```

:::note
当你使用 `Left` 或 `Right` 布局时，标签头会垂直堆叠。如果标签标题较宽，建议给 `TabItem` 设置固定 `Width`，或者约束整个标签条宽度，以避免布局问题。
:::

## 绑定 `SelectedIndex`

如果你更希望通过数字索引而不是对象引用来跟踪当前选中的标签页，可以绑定 `SelectedIndex` 属性：

```xml
<TabControl SelectedIndex="{Binding ActiveTabIndex}">
```

```csharp
[ObservableProperty]
private int _activeTabIndex;
```

当你需要通过代码切换到某个特定标签页时，这种方式非常有用，例如在向导式界面中跳转到某一步。

## 延迟加载标签内容

默认情况下，`TabControl` 在首次加载时就会创建每个标签页的内容。如果你的标签页中包含代价较高的控件或大型数据集，可以延迟到标签真正被选中时再创建内容：

```csharp
public partial class LazyTabViewModel : ObservableObject
{
    private ObservableObject? _content;

    public string Header { get; }
    private readonly Func<ObservableObject> _contentFactory;

    public LazyTabViewModel(string header, Func<ObservableObject> contentFactory)
    {
        Header = header;
        _contentFactory = contentFactory;
    }

    public ObservableObject Content => _content ??= _contentFactory();
}
```

`Content` 属性使用了工厂委托和空合并赋值运算符（`??=`），因此内容视图模型只会在第一次访问时创建。当你把 `ContentTemplate` 绑定到 `{Binding Content}` 时，这个 getter 只有在选中对应选项卡时才会触发。

## 选项卡样式设置

### 自定义选项卡头部外观

你可以在 `TabControl.Styles` 中添加样式，以调整间距、字体大小和当前选中项卡的指示效果：

```xml
<TabControl.Styles>
    <!-- Tab strip background -->
    <Style Selector="TabControl /template/ ItemsPresenter#PART_ItemsPresenter">
        <Setter Property="Margin" Value="0" />
    </Style>

    <!-- Individual tab items -->
    <Style Selector="TabItem">
        <Setter Property="Padding" Value="16,8" />
        <Setter Property="FontSize" Value="13" />
    </Style>

    <!-- Selected tab indicator -->
    <Style Selector="TabItem:selected">
        <Setter Property="Foreground" Value="#6366F1" />
    </Style>
</TabControl.Styles>
```

### 移除选项卡边框

如果你想让选项卡呈现扁平、无边框的外观：

```xml
<TabControl.Styles>
    <Style Selector="TabItem">
        <Setter Property="Background" Value="Transparent" />
    </Style>
</TabControl.Styles>
```

:::tip
你也可以组合多个样式选择器，进一步针对悬停和按下状态设置样式。例如，`TabItem:pointerover` 会在指针悬停到选项卡上时生效。
:::

## 另请参阅

- [TabControl reference](/controls/navigation/tabcontrol): Full property tables and additional examples.
- [How to bind tabs](/docs/data-binding/how-to-bind-tabs): Detailed walkthrough for binding tab content to collections.
- [Navigation how-to](/docs/how-to/navigation-how-to): Other navigation patterns such as `Carousel` and page-based navigation.
- [Data binding introduction](/docs/data-binding/introduction-to-data-binding): Core concepts for binding your UI to view models.
