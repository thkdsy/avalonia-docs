---
title: TabbedPage
description: '`TabbedPage` 通过标签栏显示一组页面。每个子 `Page` 都会成为一个标签页。'
doc-type: reference
---

import TabbedPageBottomScreenshot from '/img/controls/tabbedpage/tabbedpage-bottom.png';
import TabbedPageIconsScreenshot from '/img/controls/tabbedpage/tabbedpage-icons.png';
import TabbedPageTopScreenshot from '/img/controls/tabbedpage/tabbedpage-top.png';
import TabbedPageLeftScreenshot from '/img/controls/tabbedpage/tabbedpage-left.png';
import TabbedPageRightScreenshot from '/img/controls/tabbedpage/tabbedpage-right.png';
import TabbedPageInDrawerPageScreenshot from '/img/controls/tabbedpage/tabbedpage-in-drawerpage.png';

# TabbedPage

`TabbedPage` 通过标签栏显示一组页面。每个子 `Page` 都会成为一个标签页。标签页标题会使用页面的 `Header` 属性作为文本，使用其 `Icon` 属性作为图标。标签栏的位置默认会根据目标平台自动适配。

`TabbedPage` 继承自 `SelectingMultiPage`，而 `SelectingMultiPage` 又继承自 `MultiPage`。这条继承链提供了以下功能：

- `Pages` 集合
- `ItemsSource`
- `PageTemplate`
- `SelectedIndex`
- `SelectedPage`
- `CurrentPage`
- `SelectionChanged` 事件
- `PagesChanged` 事件
- `CurrentPageChanged` 事件

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 默认值 | 说明 |
| -------- | ---- | ------- | ----------- |
| `Pages` | `IEnumerable<Page>?` | `null` | 子页面集合。这是 XAML 的内容属性。支持任意 `IEnumerable<Page>`，包括可观察集合。 |
| `ItemsSource` | `IEnumerable?` | `null` | 视图模型集合。设置后，会优先于 `Pages` 作为项目源。可结合 `PageTemplate` 将每个数据项转换为 `Page`。 |
| `PageTemplate` | `IDataTemplate?` | `null` | 当数据源包含的是数据对象而不是页面本身时，用于生成 `Page` 实例的数据模板。 |
| `TabPlacement` | `TabPlacement` | `Auto` | 标签栏位置。见下方 `TabPlacement` 取值表。 |
| `IsKeyboardNavigationEnabled` | `bool` | `true` | 启用方向键以及 Ctrl+Tab、Ctrl+Shift+Tab 来切换标签页。 |
| `IsGestureEnabled` | `bool` | `false` | 启用滑动手势切换标签页。默认关闭。 |
| `PageTransition` | `IPageTransition?` | `null` | 切换标签页时播放的过渡动画。 |
| `IndicatorTemplate` | `IDataTemplate?` | `null` | 用于渲染每个标签项选择指示器的数据模板。 |
| `SelectedIndex` | `int` | `-1` | 当前选中标签页的从零开始索引。 |
| `SelectedPage` | `Page?` | `null` | 只读。当前选中的页面。 |

## TabPlacement 取值

| 值 | 说明 |
| ----- | ----------- |
| `Auto` | 平台自适应。在 Android 和 iOS 上解析为 `Bottom`，在其他平台上解析为 `Top`。 |
| `Top` | 标签栏位于内容区域顶部。 |
| `Bottom` | 标签栏位于内容区域底部。 |
| `Left` | 标签栏位于内容区域左侧。 |
| `Right` | 标签栏位于内容区域右侧。 |

## 附加属性

该属性设置在各个子 `Page` 上，用于控制标签页是否可用。

| 属性 | 类型 | 默认值 | 说明 |
| -------- | ---- | ------- | ----------- |
| `TabbedPage.IsTabEnabled` | `bool` | `true` | 当为 `false` 时，该标签页在视觉上会被禁用，并且在键盘与滑动导航中会被跳过。如果当前选中的标签页被禁用，选择会自动移动到最近的可用标签页。 |

## 标签图标

每个子 `Page` 上的 `Icon` 属性用于控制标签页标题中的图标。`TabbedPage` 会将页面的 `Icon` 和 `IconTemplate` 传递到底层 `TabItem`，因此这里沿用标准的 `Content` / `ContentTemplate` 渲染方式。

当未设置 `IconTemplate` 时，默认处理方式支持以下值：

- `Geometry`：渲染为路径图形。
- `PathIcon`：直接作为一对一控件映射使用。
- 带有 `GeometryDrawing` 的 `DrawingImage`：会提取其中的几何数据。
- `string`：解析为 SVG 路径几何字符串。
- `IImage`：渲染为位图图像。

你也可以在页面上设置 `IconTemplate`，以控制任意图标数据的渲染方式：

```xml
<ContentPage Header="Home">
    <ContentPage.Icon>
        <StreamGeometry>M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z</StreamGeometry>
    </ContentPage.Icon>
    <ContentPage.IconTemplate>
        <DataTemplate DataType="Geometry">
            <PathIcon Data="{Binding}" Width="20" Height="20" />
        </DataTemplate>
    </ContentPage.IconTemplate>
    <!-- content -->
</ContentPage>
```

## 事件

| 事件 | 说明 |
| ----- | ----------- |
| `SelectionChanged` | 当选中的标签页发生变化时引发。提供 `PreviousPage` 和 `CurrentPage`。 |
| `CurrentPageChanged` | 当 `CurrentPage` 发生变化时引发。 |
| `PagesChanged` | 当 `Pages` 集合发生变化时引发。 |

当活动标签页变化时，每个子 `Page` 上的导航生命周期事件（`NavigatedTo`、`Navigating`、`NavigatedFrom`）也会相应触发。

## 键盘导航

当 `IsKeyboardNavigationEnabled` 为 `true` 时：

- 水平标签页（Top、Bottom）：左右方向键可切换标签页。RTL 布局下方向会反转。
- 垂直标签页（Left、Right）：上下方向键可切换标签页。
- `Ctrl+Tab` 切换到下一个可用标签页，`Ctrl+Shift+Tab` 切换到上一个可用标签页。

被禁用的标签页（通过 `IsTabEnabled = false` 设置）始终会被跳过。

## 示例

### XAML 中的基础 TabbedPage

```xml
<TabbedPage xmlns="https://github.com/avaloniaui"
            xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
            x:Class="MyApp.MainTabs">

    <ContentPage Header="Home">
        <TextBlock Text="Home content"
                   VerticalAlignment="Center"
                   HorizontalAlignment="Center" />
    </ContentPage>

    <ContentPage Header="Profile">
        <TextBlock Text="Profile content"
                   VerticalAlignment="Center"
                   HorizontalAlignment="Center" />
    </ContentPage>

    <ContentPage Header="Settings">
        <TextBlock Text="Settings content"
                   VerticalAlignment="Center"
                   HorizontalAlignment="Center" />
    </ContentPage>

</TabbedPage>
```

### 代码方式创建 TabbedPage

```csharp
var tabbedPage = new TabbedPage
{
    TabPlacement = TabPlacement.Bottom,
    Pages = new AvaloniaList<Page>
    {
        new ContentPage { Header = "Home",     Content = homeView },
        new ContentPage { Header = "Profile",  Content = profileView },
        new ContentPage { Header = "Settings", Content = settingsView }
    }
};

window.Page = tabbedPage;
```

<Image light={TabbedPageBottomScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 标签图标

为每个子页面设置 `Icon` 属性：

```xml
<ContentPage Header="Home">
    <ContentPage.Icon>
        <StreamGeometry>M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z</StreamGeometry>
    </ContentPage.Icon>
    <!-- content -->
</ContentPage>

<ContentPage Header="Profile">
    <ContentPage.Icon>
        <StreamGeometry>M12,4A4,4 0 0,1 16,8A4,4 0 0,1 12,12A4,4 0 0,1 8,8A4,4 0 0,1 12,4M12,14C16.42,14 20,15.79 20,18V20H4V18C4,15.79 7.58,14 12,14Z</StreamGeometry>
    </ContentPage.Icon>
    <!-- content -->
</ContentPage>
```

```csharp
var homePage = new ContentPage
{
    Header = "Home",
    Icon   = Geometry.Parse("M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"),
    Content = homeView
};
```

<Image light={TabbedPageIconsScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 控制 TabPlacement

```csharp
// 显式设置为底部标签页（移动端默认）
tabbedPage.TabPlacement = TabPlacement.Bottom;

// 桌面端侧边栏标签页
tabbedPage.TabPlacement = TabPlacement.Left;
```

在 XAML 中：

```xml
<TabbedPage TabPlacement="Left">
    <!-- pages -->
</TabbedPage>
```

<Image light={TabbedPageTopScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

<Image light={TabbedPageLeftScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

<Image light={TabbedPageRightScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 动态管理标签页

可以在运行时添加或移除页面，标签栏会自动更新：

```csharp
var pages = new AvaloniaList<Page>();
tabbedPage.Pages = pages;

// 添加标签页
pages.Add(new ContentPage
{
    Header  = "Reports",
    Content = new ReportsView()
});

// 移除标签页
if (pages.Count > 1)
    pages.RemoveAt(pages.Count - 1);
```

### 启用滑动手势

```csharp
tabbedPage.IsGestureEnabled = true;
```

在水平标签栏上左右滑动，在垂直标签栏上上下滑动。

### 禁用标签页

```csharp
TabbedPage.SetIsTabEnabled(settingsPage, false);
```

In XAML:

```xml
<ContentPage TabbedPage.IsTabEnabled="False" Header="Settings">
    <!-- ... -->
</ContentPage>
```

被禁用的标签页会在键盘和滑动导航中被跳过。如果它在被禁用时正处于选中状态，选择会自动移动到最近的可用标签页。

### 响应选择变化

```csharp
tabbedPage.SelectionChanged += (sender, e) =>
{
    Console.WriteLine($"已从 {e.PreviousPage?.Header} 切换到 {e.CurrentPage?.Header}");
};
```

你也可以在子页面上使用导航生命周期事件，以便在某个标签页变为活动页时做出响应：

```csharp
public partial class ProfilePage : ContentPage
{
    protected override void OnNavigatedTo(NavigatedToEventArgs args)
    {
        base.OnNavigatedTo(args);
        // 该标签页现在已激活，刷新数据。
        _ = LoadProfileAsync();
    }
}
```

### 以编程方式选择标签页

```csharp
// 按索引选择
tabbedPage.SelectedIndex = 2;
```

### 每个标签页独立的导航堆栈

每个标签页都可以承载自己的 `NavigationPage`，从而维护独立的导航堆栈。某个标签页内部的导航不会影响其他标签页。`TabbedPage` 与 `NavigationPage` 之间没有特殊耦合：`TabbedPage` 会像处理其他 `Page` 一样处理子 `NavigationPage`，并从中读取 `Header` 和 `Icon` 以渲染标签栏。

```xml
<TabbedPage xmlns="https://github.com/avaloniaui"
            xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
            x:Class="MyApp.MainShell">

    <!-- Home 标签页：完整导航堆栈 -->
    <NavigationPage Header="Home">
        <local:HomeRootPage />
    </NavigationPage>

    <!-- Explore 标签页：独立堆栈 -->
    <NavigationPage Header="Explore">
        <local:ExploreRootPage />
    </NavigationPage>

    <!-- Profile 标签页：单页面，不需要导航堆栈 -->
    <ContentPage Header="Profile">
        <local:ProfileView />
    </ContentPage>

</TabbedPage>
```

```csharp
// 从该标签页中的 ContentPage 内部，在对应堆栈中导航：
private async void OnItemClick(object? sender, RoutedEventArgs e)
{
    await Navigation.PushAsync(new ItemDetailPage());
}
```

每个 `NavigationPage` 都会独立维护自己的返回堆栈。切换标签页不会重置该堆栈。

### 使用 ItemsSource 驱动数据型标签页

将视图模型集合绑定到 `ItemsSource`，并使用 `PageTemplate` 生成页面：

```xml
<TabbedPage xmlns="https://github.com/avaloniaui"
            xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
            xmlns:vm="clr-namespace:MyApp.ViewModels"
            x:Class="MyApp.MainTabs"
            ItemsSource="{Binding Sections}">

    <TabbedPage.PageTemplate>
        <DataTemplate x:DataType="vm:SectionViewModel">
            <ContentPage Header="{Binding Title}">
                <TextBlock Text="{Binding Description}"
                           VerticalAlignment="Center"
                           HorizontalAlignment="Center" />
            </ContentPage>
        </DataTemplate>
    </TabbedPage.PageTemplate>

</TabbedPage>
```

### DrawerPage 中的 TabbedPage

可将抽屉式导航菜单与内容区域中的标签页内容结合使用：

```xml
<DrawerPage xmlns="https://github.com/avaloniaui"
            xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
            x:Class="MyApp.Shell">

    <DrawerPage.Drawer>
        <ContentPage Header="Menu">
            <StackPanel Margin="12" Spacing="8">
                <Button Content="Dashboard" Click="OnDashboardClick" />
                <Button Content="Reports"   Click="OnReportsClick" />
            </StackPanel>
        </ContentPage>
    </DrawerPage.Drawer>

    <TabbedPage>
        <ContentPage Header="Overview">
            <local:OverviewView />
        </ContentPage>
        <ContentPage Header="Analytics">
            <local:AnalyticsView />
        </ContentPage>
    </TabbedPage>

</DrawerPage>
```

<Image light={TabbedPageInDrawerPageScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 另请参阅

- [API 参考](/api/avalonia/controls/tabbedpage)
- [源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Page/TabbedPage.cs)
