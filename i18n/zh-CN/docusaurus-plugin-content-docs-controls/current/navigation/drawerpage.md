---
title: DrawerPage
description: '`DrawerPage` 将滑出式抽屉面板与主内容区域结合在一起。抽屉可以作为浮出覆盖层打开，也可以作为分栏侧边栏常驻显示，或以紧凑导航栏形式呈现。'
doc-type: reference
---

import DrawerPageClosedScreenshot from '/img/controls/drawerpage/drawerpage-closed.png';
import DrawerPageOpenScreenshot from '/img/controls/drawerpage/drawerpage-open.png';
import DrawerPageHeaderFooterScreenshot from '/img/controls/drawerpage/drawerpage-header-footer.png';
import DrawerPageCompactCollapsedScreenshot from '/img/controls/drawerpage/drawerpage-compact-collapsed.png';
import DrawerPageCompactExpandedScreenshot from '/img/controls/drawerpage/drawerpage-compact-expanded.png';
import DrawerPageSplitScreenshot from '/img/controls/drawerpage/drawerpage-split.png';
import DrawerPageRightScreenshot from '/img/controls/drawerpage/drawerpage-right.png';
import DrawerPageRtlScreenshot from '/img/controls/drawerpage/drawerpage-rtl.png';

# DrawerPage

`DrawerPage` 是一种将滑出式抽屉面板与主内容区域结合在一起的页面。抽屉既可以作为浮出覆盖层打开，也可以作为常驻的分栏侧边栏显示，或以紧凑导航栏的形式呈现。抽屉面板可以从屏幕任意边缘出现，并支持滑动手势、键盘快捷键以及背景遮罩层。

`DrawerPage` 在内部是基于 `SplitView` 构建的。`DrawerBehavior` 和 `DrawerLayoutBehavior` 属性用于控制最终采用哪种 `SplitViewDisplayMode`。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 类型 | 默认值 | 说明 |
| -------- | ---- | ------- | ----------- |
| `Content` | `object?` | `null` | 主内容区域。这是 XAML 的内容属性，可接受 `Page` 或任意视图。 |
| `ContentTemplate` | `IDataTemplate?` | built-in | 当 `Content` 是数据对象时，用于显示它的数据模板。 |
| `Drawer` | `object?` | `null` | 抽屉面板内容。可接受 `Page` 或任意视图。 |
| `DrawerTemplate` | `IDataTemplate?` | `null` | 当 `Drawer` 是数据对象时，用于显示它的数据模板。 |
| `IsOpen` | `bool` | `false` | 抽屉面板当前是否打开。支持绑定。当 `DrawerBehavior` 为 `Disabled` 时，将其设为 `true` 不会产生效果；当 `DrawerBehavior` 为 `Locked` 时，将其设为 `false` 也不会生效。 |
| `DrawerLength` | `double` | `320` | 抽屉完全展开时的宽度；对于顶部和底部布局，则表示高度。 |
| `CompactDrawerLength` | `double` | `48` | 在 `CompactOverlay` 和 `CompactInline` 布局模式下，可见紧凑导航栏的宽度；对于 `Top` 和 `Bottom` 布局，则表示高度。 |
| `DrawerBreakpointLength` | `double` | `0` | 响应式断点。当大于 `0` 且页面宽度低于该值时，无论 `DrawerLayoutBehavior` 如何设置，布局都会自动切换为 `Overlay`。仅适用于 `Left` 和 `Right` 布局。设为 `0` 可禁用响应式切换。 |
| `IsGestureEnabled` | `bool` | `true` | 启用滑动手势来打开和关闭抽屉。 |
| `DrawerBehavior` | `DrawerBehavior` | `Auto` | 控制打开和关闭行为。见下方 `DrawerBehavior` 取值表。 |
| `DrawerLayoutBehavior` | `DrawerLayoutBehavior` | `Overlay` | 当 `DrawerBehavior` 为 `Auto` 时，控制布局模式。见下方 `DrawerLayoutBehavior` 取值表。 |
| `DrawerPlacement` | `DrawerPlacement` | `Left` | 控制抽屉从哪个边缘滑出。见下方 `DrawerPlacement` 取值表。 |
| `DrawerHeader` | `object?` | `null` | 渲染在抽屉面板顶部、主抽屉内容上方的内容。 |
| `DrawerFooter` | `object?` | `null` | 渲染在抽屉面板底部、主抽屉内容下方的内容。 |
| `DrawerIcon` | `object?` | `null` | 显示在抽屉切换按钮上的图标数据。可结合 `DrawerIconTemplate` 控制渲染方式。若未提供模板，默认模板可处理 `Geometry`、`PathIcon`、`IImage` 和 SVG 路径字符串。 |
| `DrawerIconTemplate` | `IDataTemplate?` | `null` | 用于在每个图标呈现槽位中渲染 `DrawerIcon` 的数据模板。每个呈现器都会独立根据该模板生成自己的可视对象。 |
| `DrawerBackground` | `IBrush?` | `null` | 抽屉面板的背景画刷。 |
| `DrawerHeaderBackground` | `IBrush?` | `null` | 抽屉头部区域的背景画刷。 |
| `DrawerHeaderForeground` | `IBrush?` | `null` | 抽屉头部区域的前景画刷。 |
| `DrawerFooterBackground` | `IBrush?` | `null` | 抽屉底部区域的背景画刷。 |
| `DrawerFooterForeground` | `IBrush?` | `null` | 抽屉底部区域的前景画刷。 |
| `BackdropBrush` | `IBrush?` | `null` | 在覆盖模式下，当抽屉打开时渲染在内容区域上的画刷。半透明画刷可实现遮罩层效果。为 `null` 时不显示。 |
| `DisplayMode` | `SplitViewDisplayMode` | computed | 只读。当前实际应用到内部 `SplitView` 的显示模式。它会由 `DrawerBehavior`、`DrawerLayoutBehavior` 和 `DrawerBreakpointLength` 自动管理。 |
| `HorizontalContentAlignment` | `HorizontalAlignment` | `Stretch` | 主内容的水平对齐方式。 |
| `VerticalContentAlignment` | `VerticalAlignment` | `Stretch` | 主内容的垂直对齐方式。 |

## DrawerBehavior 取值

| 值 | 说明 |
| ----- | ----------- |
| `Auto` | 默认值。使用 `DrawerLayoutBehavior` 决定显示模式，并遵守 `DrawerBreakpointLength` 的响应式切换规则。 |
| `Flyout` | 始终以覆盖层方式打开，忽略 `DrawerLayoutBehavior`。 |
| `Locked` | 始终保持打开状态。用户无法关闭，也不能通过设置 `IsOpen = false` 来关闭。 |
| `Disabled` | 始终关闭。抽屉切换按钮会被隐藏，手势也会被阻止。设置 `IsOpen = true` 不会生效。 |

## DrawerLayoutBehavior 取值

在 `DrawerBehavior` 为 `Auto` 时生效。

| 值 | 说明 |
| ----- | ----------- |
| `Overlay` | 默认值。抽屉作为浮出层滑入内容区域之上。 |
| `Split` | 抽屉和内容始终并排显示。 |
| `CompactOverlay` | 始终显示一个 `CompactDrawerLength` 宽度的窄导航栏。打开时，抽屉会以覆盖层方式展开。 |
| `CompactInline` | 始终显示一个窄导航栏。打开时，抽屉会展开并将内容区域推开。 |

## DrawerPlacement 取值

| 值 | 说明 |
| ----- | ----------- |
| `Left` | 默认值。抽屉从左边缘滑入。 |
| `Right` | 抽屉从右边缘滑入。 |
| `Top` | 抽屉从顶部向下滑入。 |
| `Bottom` | 抽屉从底部向上滑入。 |

## 事件

| 事件 | 参数类型 | 说明 |
| ----- | --------- | ----------- |
| `Opened` | `RoutedEventArgs` | 当抽屉切换到打开状态时引发的冒泡路由事件。 |
| `Closing` | `DrawerClosingEventArgs` | 在抽屉即将关闭前引发的冒泡路由事件。将 `args.Cancel = true` 可阻止关闭。 |
| `Closed` | `RoutedEventArgs` | 当抽屉关闭完成后引发的冒泡路由事件，前提是关闭操作未被取消。 |

## 手势与键盘

- 向内滑动以打开：从边缘区域（默认 20 px）向内滑动可打开抽屉。在紧凑模式下，滑动区域就是紧凑导航栏本身。
- 滑动关闭：当抽屉打开时，向远离抽屉的方向滑动即可关闭。
- Escape 键：关闭覆盖式或紧凑覆盖式抽屉。当 `DrawerBehavior` 为 `Locked` 时无效。
- 系统返回键：当抽屉处于打开状态，且不在 `Locked` 或 `Disabled` 模式时，会关闭抽屉。
- 点击遮罩层：当设置了 `BackdropBrush` 时，点击背景遮罩会关闭抽屉。

## NavigationPage 集成

当 `Content` 是 `NavigationPage` 时，`DrawerPage` 会自动：

- 隐藏自身顶部栏，并将标题渲染交给 `NavigationPage` 的导航栏。
- 在 `NavigationPage` 堆栈根页面处，在导航栏中显示汉堡菜单切换按钮。
- 当用户在堆栈中继续深入导航时，隐藏汉堡按钮，并由返回按钮取代其位置。

## 示例

### XAML 中的基础 DrawerPage

```xml
<DrawerPage xmlns="https://github.com/avaloniaui"
            xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
            x:Class="MyApp.Shell">

    <DrawerPage.Drawer>
        <ContentPage Header="Menu">
            <StackPanel Spacing="8" Margin="12">
                <Button Content="Home"     Click="OnHomeClick" />
                <Button Content="Profile"  Click="OnProfileClick" />
                <Button Content="Settings" Click="OnSettingsClick" />
            </StackPanel>
        </ContentPage>
    </DrawerPage.Drawer>

    <NavigationPage>
        <local:HomePage />
    </NavigationPage>

</DrawerPage>
```

### 代码方式创建基础 DrawerPage

```csharp
var drawerPage = new DrawerPage
{
    Drawer = new ContentPage
    {
        Header = "Menu",
        Content = new StackPanel
        {
            Children =
            {
                new Button { Content = "Home" },
                new Button { Content = "Settings" }
            }
        }
    },
    Content = new NavigationPage { Content = new HomePage() }
};

window.Page = drawerPage;
```

<Image light={DrawerPageClosedScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

<Image light={DrawerPageOpenScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 切换抽屉开关

```csharp
// 打开
drawerPage.IsOpen = true;

// 关闭
drawerPage.IsOpen = false;

// 切换
drawerPage.IsOpen = !drawerPage.IsOpen;
```

在 XAML 中可通过按钮绑定来触发：

```xml
<Button Content="Toggle Menu"
        Command="{Binding ToggleDrawerCommand}" />
```

```csharp
// 在视图模型中
[RelayCommand]
private void ToggleDrawer() => IsDrawerOpen = !IsDrawerOpen;
```

### 从抽屉菜单中导航

当用户点击菜单项后关闭抽屉：

```csharp
private void OnHomeClick(object? sender, RoutedEventArgs e)
{
    if (Content is NavigationPage nav)
        _ = nav.PopToRootAsync();

    IsOpen = false;
}

private void OnProfileClick(object? sender, RoutedEventArgs e)
{
    if (Content is NavigationPage nav)
        _ = nav.PushAsync(new ProfilePage());

    IsOpen = false;
}
```

### 带头部和底部的 Drawer

```xml
<DrawerPage xmlns="https://github.com/avaloniaui"
            xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
            x:Class="MyApp.Shell"
            DrawerLength="280">

    <DrawerPage.Drawer>
        <ContentPage Header="Navigation">
        </ContentPage>
    </DrawerPage.Drawer>

    <DrawerPage.DrawerHeader>
        <Border Background="#1E3A5F" Padding="16">
            <StackPanel>
                <TextBlock Text="My App"
                           Foreground="White"
                           FontSize="20"
                           FontWeight="SemiBold" />
                <TextBlock Text="user@example.com"
                           Foreground="#AAD4F5"
                           FontSize="13" />
            </StackPanel>
        </Border>
    </DrawerPage.DrawerHeader>

    <DrawerPage.DrawerFooter>
        <Border Padding="12">
            <Button Content="Sign Out"
                    Click="OnSignOutClick"
                    HorizontalAlignment="Stretch" />
        </Border>
    </DrawerPage.DrawerFooter>

    <NavigationPage>
        <local:HomePage />
    </NavigationPage>

</DrawerPage>
```

<Image light={DrawerPageHeaderFooterScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 常驻侧边栏（Split 布局）

抽屉会始终保持打开，并显示在内容旁边。不需要切换按钮或手势。

```csharp
var drawerPage = new DrawerPage
{
    DrawerLayoutBehavior = DrawerLayoutBehavior.Split,
    DrawerLength = 240,
    Drawer = sidebarPage,
    Content = new NavigationPage { Content = new HomePage() }
};
```

<Image light={DrawerPageSplitScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 紧凑导航栏

一个窄图标栏会始终可见。点按或滑动它可打开完整抽屉。在 `CompactInline` 模式下，展开时会将内容区域推开；在 `CompactOverlay` 模式下，展开的抽屉会覆盖在内容之上。

```xml
<DrawerPage DrawerLayoutBehavior="CompactInline"
            CompactDrawerLength="56"
            DrawerLength="240">

    <DrawerPage.Drawer>
        <StackPanel Spacing="2" Margin="0,8">
            <Button HorizontalAlignment="Stretch" Background="Transparent"
                    ToolTip.Tip="Home">
                <StackPanel HorizontalAlignment="Center" Spacing="3">
                    <PathIcon Width="20" Height="20"
                              Data="M10,20V14H14V20H19V12H22L12,3L2,12H5V20H10Z" />
                    <TextBlock Text="Home" FontSize="9" HorizontalAlignment="Center" />
                </StackPanel>
            </Button>
            <Button HorizontalAlignment="Stretch" Background="Transparent"
                    ToolTip.Tip="Profile">
                <StackPanel HorizontalAlignment="Center" Spacing="3">
                    <PathIcon Width="20" Height="20"
                              Data="M12,4A4,4 0 0,1 16,8A4,4 0 0,1 12,12A4,4 0 0,1 8,8A4,4 0 0,1 12,4M12,14C16.42,14 20,15.79 20,18V20H4V18C4,15.79 7.58,14 12,14Z" />
                    <TextBlock Text="Profile" FontSize="9" HorizontalAlignment="Center" />
                </StackPanel>
            </Button>
        </StackPanel>
    </DrawerPage.Drawer>

    <NavigationPage>
        <local:HomePage />
    </NavigationPage>

</DrawerPage>
```

```csharp
var drawerPage = new DrawerPage
{
    DrawerLayoutBehavior = DrawerLayoutBehavior.CompactInline,
    CompactDrawerLength = 56,
    DrawerLength = 240,
    Drawer = navRailPage,
    Content = new NavigationPage { Content = new HomePage() }
};
```

<Image light={DrawerPageCompactCollapsedScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

<Image light={DrawerPageCompactExpandedScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 使用 DrawerBreakpointLength 的响应式布局

在宽窗口下自动使用分栏侧边栏，在窄窗口下自动切换为覆盖式抽屉。当页面宽度低于断点时，抽屉会关闭并切换为 `Overlay`；当宽度重新高于该值时，已配置的布局会恢复，抽屉也会自动重新打开。

这仅适用于 `Left` 和 `Right` 布局。

```xml
<DrawerPage DrawerLayoutBehavior="Split"
            DrawerBreakpointLength="720"
            DrawerLength="240">
    <!-- drawer and content -->
</DrawerPage>
```

```csharp
var drawerPage = new DrawerPage
{
    DrawerLayoutBehavior = DrawerLayoutBehavior.Split,
    DrawerBreakpointLength = 720, // overlay when width < 720 px, split when >= 720 px
    Drawer = menuPage,
    Content = mainPage
};
```

### RTL 支持

`DrawerPlacement` 表示的是逻辑方向，而不是物理方向。当 `FlowDirection` 为 `RightToLeft` 时，`Left` 和 `Right` 的布局会镜像：`Left` 会显示在右边缘，`Right` 会显示在左边缘。滑动手势也会自动镜像，因此可以在无需额外代码的情况下支持阿拉伯语、希伯来语等 RTL 语言环境。

```xml
<DrawerPage FlowDirection="RightToLeft"
            DrawerPlacement="Left">
    <!-- 在 RTL 模式下，抽屉会从右边缘打开 -->
</DrawerPage>
```

<Image light={DrawerPageRtlScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 右侧抽屉

适用于筛选面板、详情检查器或上下文侧边栏等场景：

```xml
<DrawerPage DrawerPlacement="Right"
            DrawerLength="300">

    <DrawerPage.Drawer>
        <ContentPage Header="Filters">
            <local:FilterPanel />
        </ContentPage>
    </DrawerPage.Drawer>

    <local:ResultsPage />

</DrawerPage>
```

<Image light={DrawerPageRightScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 背景遮罩层

在覆盖模式下，可在抽屉后方添加半透明遮罩层：

```csharp
drawerPage.BackdropBrush = new SolidColorBrush(Colors.Black, opacity: 0.4);
```

点击遮罩层会自动关闭抽屉。

### 取消关闭

你可以使用 `Closing` 事件来阻止抽屉关闭，例如当抽屉面板中存在未保存的工作时：

```csharp
drawerPage.Closing += (sender, e) =>
{
    if (HasUnsavedChanges)
        e.Cancel = true;
};
```

### 响应打开与关闭

```csharp
drawerPage.Opened += (sender, e) =>
{
    Console.WriteLine("抽屉已打开");
};

drawerPage.Closed += (sender, e) =>
{
    Console.WriteLine("抽屉已关闭");
};
```

你也可以通过属性变更处理程序来监视 `IsOpen`：

```csharp
drawerPage.PropertyChanged += (sender, e) =>
{
    if (e.Property == DrawerPage.IsOpenProperty)
        Console.WriteLine($"IsOpen: {drawerPage.IsOpen}");
};
```

### 针对 IsOpen 的 MVVM 绑定

```xml
<DrawerPage IsOpen="{Binding IsMenuOpen}"
            DrawerLayoutBehavior="Overlay">
    <!-- ... -->
</DrawerPage>
```

```csharp
// 在视图模型中：
[ObservableProperty]
private bool _isMenuOpen;
```

### 自定义抽屉图标

可将 `DrawerIcon` 设置为 `Geometry` 值，并提供 `DrawerIconTemplate`，这样每个图标呈现器都会独立生成自己的可视对象：

```xml
<DrawerPage>
    <DrawerPage.DrawerIcon>
        <StreamGeometry>M3,6H21V8H3V6M3,11H21V13H3V11M3,16H21V18H3V16Z</StreamGeometry>
    </DrawerPage.DrawerIcon>
    <DrawerPage.DrawerIconTemplate>
        <DataTemplate DataType="Geometry">
            <PathIcon Data="{Binding}" />
        </DataTemplate>
    </DrawerPage.DrawerIconTemplate>
    <!-- ... -->
</DrawerPage>
```

你也可以在运行时以编程方式更改图标：

```csharp
drawerPage.DrawerIcon = Geometry.Parse("M4,8H8V4H4V8M10,20H14V16H10V20M4,20H8V16H4V20M4,14H8V10H4V14M10,14H14V10H10V14M16,4V8H20V4H16M10,8H14V4H10V8M16,14H20V10H16V14M16,20H20V16H16V20Z");
```

### 锁定抽屉（始终打开）

```csharp
drawerPage.DrawerBehavior = DrawerBehavior.Locked;
// IsOpen 会被强制设为 true。用户无法关闭它。
```

### 禁用抽屉

```csharp
drawerPage.DrawerBehavior = DrawerBehavior.Disabled;
// 切换按钮会被隐藏。IsOpen 保持为 false。手势被禁用。
```

### 在 NavigationPage 上使用 CrossFade 过渡的 DrawerPage

```csharp
var navPage = new NavigationPage
{
    Content = new HomePage(),
    PageTransition = new CrossFade(TimeSpan.FromMilliseconds(250))
};

var drawerPage = new DrawerPage
{
    DrawerLayoutBehavior = DrawerLayoutBehavior.Overlay,
    DrawerLength = 280,
    Drawer = menuPage,
    Content = navPage
};
```

## 另请参阅

- [API 参考](/api/avalonia/controls/drawerpage)
- [源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Page/DrawerPage.cs)
