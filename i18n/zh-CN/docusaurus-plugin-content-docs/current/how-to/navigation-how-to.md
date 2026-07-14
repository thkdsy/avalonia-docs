---
id: navigation-how-to
title: "如何：在视图之间导航"
description: Avalonia 应用程序中在视图和页面之间切换的常见模式。
doc-type: how-to
---

本指南介绍 Avalonia 应用程序中在视图（页面）之间切换的常见模式。不同模式适用于不同场景，从简单的双页面应用，到带有历史导航能力的完整桌面壳层界面都包括在内。

## 选择导航模式

开始之前，先考虑哪一种模式最适合你的需求：

| 模式 | 最适合的场景 |
|---|---|
| 搭配数据模板的 [`ContentControl`](/api/avalonia/controls/contentcontrol) | 页面数量较少的小型应用 |
| [`TransitioningContentControl`](/api/avalonia/controls/transitioningcontentcontrol) | 与上面类似，但带动画切换效果 |
| `TabControl` | 设置界面、文档编辑器 |
| 侧边栏导航 | 带主菜单的桌面应用 |
| 回退栈导航 | 向导流程、浏览器式历史导航 |

## 使用 ContentControl 切换视图

最简单的导航模式，是使用 `ContentControl` 来显示不同的视图模型，并通过数据模板解析出对应视图。

```xml
<Window x:Class="MyApp.Views.MainWindow"
        xmlns:vm="using:MyApp.ViewModels"
        xmlns:views="using:MyApp.Views">
    <Window.DataTemplates>
        <DataTemplate DataType="vm:HomeViewModel">
            <views:HomeView />
        </DataTemplate>
        <DataTemplate DataType="vm:SettingsViewModel">
            <views:SettingsView />
        </DataTemplate>
    </Window.DataTemplates>

    <Grid RowDefinitions="Auto,*">
        <StackPanel Grid.Row="0" Orientation="Horizontal" Spacing="8" Margin="8">
            <Button Content="Home" Command="{Binding GoHomeCommand}" />
            <Button Content="Settings" Command="{Binding GoSettingsCommand}" />
        </StackPanel>

        <ContentControl Grid.Row="1" Content="{Binding CurrentPage}" />
    </Grid>
</Window>
```

对应的视图模型：

```csharp
public partial class MainViewModel : ObservableObject
{
    [ObservableProperty]
    private ObservableObject _currentPage;

    public MainViewModel()
    {
        _currentPage = new HomeViewModel();
    }

    [RelayCommand]
    private void GoHome() => CurrentPage = new HomeViewModel();

    [RelayCommand]
    private void GoSettings() => CurrentPage = new SettingsViewModel();
}
```

当 `CurrentPage` 变化时，`ContentControl` 会自动查找匹配的 [`DataTemplate`](/api/avalonia/markup/xaml/templates/datatemplate)，并显示对应的视图。之所以能工作，是因为 Avalonia 会沿视觉树向上查找一个 `DataType` 与 `Content` 所赋对象匹配的 `DataTemplate`。

:::tip
如果你的视图模型很多，那么手动逐个列出 `DataTemplate` 会比较繁琐。后文中的 [View locator pattern](#view-locator-pattern) 提供了一种自动化替代方案。
:::

## 使用过渡动画切换视图

如果你想让视图切换带动画，可以把 `ContentControl` 替换为 `TransitioningContentControl`：

```xml
<TransitioningContentControl Content="{Binding CurrentPage}">
    <TransitioningContentControl.PageTransition>
        <CrossFade Duration="0:0:0.25" />
    </TransitioningContentControl.PageTransition>
</TransitioningContentControl>
```

可用的内置过渡效果包括：

| 过渡效果 | 作用 |
|---|---|
| `CrossFade` | 在旧内容和新内容之间做淡入淡出 |
| `PageSlide` | 水平或垂直滑动内容 |
| `CompositePageTransition` | 把多个过渡效果组合在一起 |

```xml
<!-- 滑动过渡 -->
<TransitioningContentControl.PageTransition>
    <PageSlide Duration="0:0:0.3" Orientation="Horizontal" />
</TransitioningContentControl.PageTransition>

<!-- 组合效果：滑动 + 淡入淡出 -->
<TransitioningContentControl.PageTransition>
    <CompositePageTransition>
        <CrossFade Duration="0:0:0.2" />
        <PageSlide Duration="0:0:0.3" Orientation="Horizontal" />
    </CompositePageTransition>
</TransitioningContentControl.PageTransition>
```

## 基于标签页的导航

当你希望用户在一组固定面板之间切换时，例如设置分类或文档标签页，可以使用 `TabControl`。

```xml
<TabControl>
    <TabItem Header="General">
        <views:GeneralSettingsView />
    </TabItem>
    <TabItem Header="Appearance">
        <views:AppearanceSettingsView />
    </TabItem>
    <TabItem Header="Advanced">
        <views:AdvancedSettingsView />
    </TabItem>
</TabControl>
```

### 从集合生成动态标签页

如果你的标签页需要由数据驱动（例如已打开的文档），就把 `ItemsSource` 绑定到视图模型中的集合上。

```xml
<TabControl ItemsSource="{Binding OpenDocuments}"
            SelectedItem="{Binding ActiveDocument}">
    <TabControl.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal" Spacing="8">
                <TextBlock Text="{Binding Title}" />
                <Button Content="x" FontSize="10"
                        Command="{Binding $parent[TabControl].((vm:MainViewModel)DataContext).CloseDocumentCommand}"
                        CommandParameter="{Binding}" />
            </StackPanel>
        </DataTemplate>
    </TabControl.ItemTemplate>
    <TabControl.ContentTemplate>
        <DataTemplate>
            <views:DocumentView />
        </DataTemplate>
    </TabControl.ContentTemplate>
</TabControl>
```

## 侧边栏导航

一种常见的桌面端模式，是把持久显示的菜单放在侧边栏，而主内容区根据选择切换视图。下面的示例使用 `ListBox` 实现菜单，用 `TransitioningContentControl` 实现内容区切换。

```xml
<Grid ColumnDefinitions="220,*">
    <!-- 侧边栏 -->
    <Border Grid.Column="0" Background="#F3F4F6">
        <ListBox ItemsSource="{Binding MenuItems}"
                 SelectedItem="{Binding SelectedMenuItem}"
                 Background="Transparent">
            <ListBox.ItemTemplate>
                <DataTemplate>
                    <StackPanel Orientation="Horizontal" Spacing="8" Margin="8,4">
                        <PathIcon Data="{Binding Icon}" Width="16" Height="16" />
                        <TextBlock Text="{Binding Title}" />
                    </StackPanel>
                </DataTemplate>
            </ListBox.ItemTemplate>
        </ListBox>
    </Border>

    <!-- 内容区域 -->
    <TransitioningContentControl Grid.Column="1" Content="{Binding CurrentPage}" />
</Grid>
```

```csharp
public partial class MainViewModel : ObservableObject
{
    public ObservableCollection<MenuItem> MenuItems { get; } = new()
    {
        new MenuItem("Home", "HomeIcon", () => new HomeViewModel()),
        new MenuItem("Settings", "SettingsIcon", () => new SettingsViewModel()),
        new MenuItem("About", "InfoIcon", () => new AboutViewModel()),
    };

    [ObservableProperty]
    private MenuItem? _selectedMenuItem;

    [ObservableProperty]
    private ObservableObject? _currentPage;

    partial void OnSelectedMenuItemChanged(MenuItem? value)
    {
        CurrentPage = value?.CreatePage();
    }
}

public record MenuItem(string Title, string Icon, Func<ObservableObject> CreatePage);
```

## 带回退栈的导航

如果你的应用程序需要类似浏览器的前进/后退能力（例如向导流程或文件浏览器），可以通过两个栈来维护访问历史。

```csharp
public partial class NavigationViewModel : ObservableObject
{
    private readonly Stack<ObservableObject> _backStack = new();
    private readonly Stack<ObservableObject> _forwardStack = new();

    [ObservableProperty]
    private ObservableObject? _currentPage;

    public bool CanGoBack => _backStack.Count > 0;
    public bool CanGoForward => _forwardStack.Count > 0;

    public void NavigateTo(ObservableObject page)
    {
        if (CurrentPage is not null)
            _backStack.Push(CurrentPage);

        _forwardStack.Clear();
        CurrentPage = page;

        OnPropertyChanged(nameof(CanGoBack));
        OnPropertyChanged(nameof(CanGoForward));
    }

    [RelayCommand(CanExecute = nameof(CanGoBack))]
    private void GoBack()
    {
        if (CurrentPage is not null)
            _forwardStack.Push(CurrentPage);

        CurrentPage = _backStack.Pop();

        OnPropertyChanged(nameof(CanGoBack));
        OnPropertyChanged(nameof(CanGoForward));
        GoBackCommand.NotifyCanExecuteChanged();
        GoForwardCommand.NotifyCanExecuteChanged();
    }

    [RelayCommand(CanExecute = nameof(CanGoForward))]
    private void GoForward()
    {
        if (CurrentPage is not null)
            _backStack.Push(CurrentPage);

        CurrentPage = _forwardStack.Pop();

        OnPropertyChanged(nameof(CanGoBack));
        OnPropertyChanged(nameof(CanGoForward));
        GoBackCommand.NotifyCanExecuteChanged();
        GoForwardCommand.NotifyCanExecuteChanged();
    }
}
```

```xml
<Grid RowDefinitions="Auto,*">
    <StackPanel Grid.Row="0" Orientation="Horizontal" Spacing="4" Margin="8">
        <Button Content="Back" Command="{Binding GoBackCommand}" />
        <Button Content="Forward" Command="{Binding GoForwardCommand}" />
    </StackPanel>

    <TransitioningContentControl Grid.Row="1" Content="{Binding CurrentPage}" />
</Grid>
```

## 视图定位器模式

与其为每个视图模型都声明一个 `DataTemplate`，你也可以使用视图定位器，按照约定自动解析对应视图。该定位器会把完整类型名中的 `ViewModel` 替换为 `View`，然后实例化得到的类型。

```csharp
public class ViewLocator : IDataTemplate
{
    public Control Build(object? data)
    {
        if (data is null) return new TextBlock { Text = "No data" };

        var name = data.GetType().FullName!
            .Replace("ViewModel", "View", StringComparison.Ordinal);

        var type = Type.GetType(name);

        if (type is not null)
            return (Control)Activator.CreateInstance(type)!;

        return new TextBlock { Text = $"View not found: {name}" };
    }

    public bool Match(object? data)
    {
        return data is ObservableObject;
    }
}
```

在 `App.axaml` 中注册该定位器，使其全局生效：

```xml
<Application.DataTemplates>
    <local:ViewLocator />
</Application.DataTemplates>
```

这样一来，任何绑定到视图模型的 `ContentControl` 都会自动解析出对应视图。例如，`HomeViewModel` 会映射到 `HomeView`，`SettingsViewModel` 会映射到 `SettingsView`。

:::note
这种约定要求你的视图类与视图模型类位于平行命名空间中（例如 `MyApp.ViewModels.HomeViewModel` 和 `MyApp.Views.HomeView`）。如果你的项目使用了不同的目录或命名结构，请相应调整 `Build` 方法中的字符串替换逻辑。
:::

## 另请参阅

- [Page transitions](/docs/graphics-animation/page-transitions)：视图之间的过渡动画。
- [Data templates](/docs/data-templates/introduction-to-data-templates)：数据模板如何解析视图。
- [View locator](/docs/data-templates/view-locator)：视图模型到视图的自动映射。
- [The MVVM pattern](/docs/fundamentals/the-mvvm-pattern)：视图模型架构。
