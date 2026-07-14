---
title: ContentPage
description: '`ContentPage` 是 Avalonia 应用中最基础的页面构建块。它承载单个根视图，并可与其他页面容器协同工作。'
doc-type: reference
---

import ContentPageInNavigationScreenshot from '/img/controls/contentpage/contentpage-in-navigationpage.png';
import ContentPageStandaloneScreenshot from '/img/controls/contentpage/contentpage-standalone.png';
import ContentPageTopCommandBarScreenshot from '/img/controls/contentpage/contentpage-top-commandbar.png';
import ContentPageBottomCommandBarScreenshot from '/img/controls/contentpage/contentpage-bottom-commandbar.png';
import ContentPageSafeAreaDisabledScreenshot from '/img/controls/contentpage/contentpage-safe-area-disabled.png';
import ContentPageAsTabScreenshot from '/img/controls/contentpage/contentpage-as-tab.png';

# ContentPage

`ContentPage` 是 Avalonia 应用中最基础的页面构建块。它承载一个单独的根视图，通常是某个布局面板，并可与其他页面容器协同工作，例如 `NavigationPage`、`TabbedPage`、`DrawerPage` 和 `CarouselPage`。在使用页面导航系统的应用中，每一个界面通常都是一个 `ContentPage` 或其子类。

它的类层级为 `TemplatedControl -> Page -> ContentPage`，这意味着它能够完整受益于 Avalonia 的样式系统、主题系统以及数据绑定能力。

## Header 如何显示

来自 `Page` 基类的 `Header` 属性会根据所处上下文控制标题文本的显示方式：

| 宿主 | `Header` 的使用方式 |
| ---- | -------------------- |
| `NavigationPage` | 显示为导航栏中的页面标题。 |
| `TabbedPage` | 显示为标签栏中的标签文本。 |
| `DrawerPage` | 容器本身不会直接显示它。 |
| 独立 `Window` | 不会显示。此时应改为设置 `Window.Title`。 |

## 常用属性

你最常使用的通常是以下属性：

`ContentPage` 继承自 `Page`。下面这些属性由 `ContentPage` 自身定义。

| 属性 | 类型 | 默认值 | 说明 |
| -------- | ---- | ------- | ----------- |
| `Content` | `object?` | `null` | 页面上显示的单个根视图。这是 XAML 的内容属性，通常会是 `Grid` 或 `StackPanel` 之类的布局面板。 |
| `ContentTemplate` | `IDataTemplate?` | `null` | 当 `Content` 是数据对象而不是控件时，用于显示它的数据模板。 |
| `AutomaticallyApplySafeAreaPadding` | `bool` | `true` | 为 `true` 时，会自动将平台安全区域内边距（刘海、状态栏、Home 指示器）作为 padding 应用到内容呈现器上。对于地图或启动页这类全出血设计，可将其设为 `false`。 |
| `TopCommandBar` | `object?` | `null` | 渲染在页面内容上方命令栏插槽中的内容，通常为 `CommandBar`。当值为 `null` 时会自动隐藏。 |
| `BottomCommandBar` | `object?` | `null` | 渲染在页面内容下方命令栏插槽中的内容。适合移动端底部工具栏。值为 `null` 时会自动隐藏。 |
| `HorizontalContentAlignment` | `HorizontalAlignment` | `Stretch` | `Content` 在其呈现器中的水平对齐方式。 |
| `VerticalContentAlignment` | `VerticalAlignment` | `Stretch` | `Content` 在其呈现器中的垂直对齐方式。 |

## `Page` 基类属性

以下属性继承自 `Page`，可用于所有页面类型。

| 属性 | 类型 | 默认值 | 说明 |
| -------- | ---- | ------- | ----------- |
| `Header` | `object?` | `null` | 显示在导航栏或标签条中的文本或自定义视图。可接受字符串或任意 Avalonia 控件。 |
| `Icon` | `object?` | `null` | 当页面托管在 `TabbedPage` 中时，显示在标签条中的图标。可接受 `Geometry`、`PathIcon`、`IImage` 或 SVG 路径字符串。 |
| `IconTemplate` | `IDataTemplate?` | `null` | 用于显示 `Icon` 属性的数据模板。设置后，图标会通过使用该模板的 `ContentPresenter` 进行呈现，而不是依赖内置图标类型处理逻辑。 |
| `SafeAreaPadding` | `Thickness` | `default` | 当前作用于此页面的安全区域内边距，由父页面或平台传递而来。 |
| `Navigation` | `INavigation?` | `null` | 由父级 `NavigationPage` 注入的导航服务。当页面不在 `NavigationPage` 中时，其值为 `null`。 |
| `IsInNavigationPage` | `bool` | `false` | 当此页面托管在 `NavigationPage` 中时为 `true`。适合用来按条件显示或隐藏页面内的返回导航控件。 |

## 导航事件

`Page` 定义了三个导航生命周期事件。每当用户导航到该页面或从该页面离开时，它们会按如下顺序触发。

| 顺序 | 事件 | 签名 | 说明 |
| ----- | ----- | --------- | ----------- |
| 1 | `Navigating` | `Func<NavigatingFromEventArgs, Task>` | 在离开当前页面之前异步触发。每个已注册处理程序都会按顺序等待完成。将 `args.Cancel = true` 可中止导航。 |
| 2 | `NavigatedFrom` | `EventHandler<NavigatedFromEventArgs>` | 当页面不再是活动页面后触发。可用于释放资源或停止计时器。 |
| 3 | `NavigatedTo` | `EventHandler<NavigatedToEventArgs>` | 当页面成为活动页面时触发。可用于加载或刷新数据。 |

此外，还有一个用于处理平台返回按钮的路由事件：

| 事件 | 说明 |
| ----- | ----------- |
| `PageNavigationSystemBackButtonPressed` | 当此页面处于活动状态时，按下平台返回按钮会引发该冒泡路由事件。可通过处理它来拦截或自定义返回导航。 |

### 重写生命周期方法

与其直接订阅这些事件，不如在子类中重写对应的虚方法：

```csharp
public partial class FeedPage : ContentPage
{
    private CancellationTokenSource? _cts;

    protected override void OnNavigatedTo(NavigatedToEventArgs args)
    {
        base.OnNavigatedTo(args);
        _cts = new CancellationTokenSource();
        _ = LoadDataAsync(_cts.Token);
    }

    protected override void OnNavigatingFrom(NavigatingFromEventArgs args)
    {
        // 同步的导航前钩子。
        // 如需执行异步逻辑，请改为订阅 Navigating 事件。
    }

    protected override void OnNavigatedFrom(NavigatedFromEventArgs args)
    {
        base.OnNavigatedFrom(args);
        _cts?.Cancel();
        _cts = null;
    }

    private async Task LoadDataAsync(CancellationToken ct) { /* ... */ }
}
```

`NavigatedTo` 和 `NavigatedFrom` 与 Avalonia 的 `Loaded` 和 `Unloaded` 事件不同。`Loaded` 会在控件加入可视树时触发一次；而 `NavigatedTo` 会在页面每次成为顶层页面时触发，包括用户访问子页面后再返回此页面的情况。对于每次访问都必须刷新的数据，应使用 `NavigatedTo`。

## 示例

### 最小化 XAML `ContentPage` 示例

```xml
<ContentPage xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MyApp.HomePage"
             Header="Home">

    <StackPanel Margin="20" Spacing="16">
        <TextBlock Text="Hello, Avalonia!" FontSize="24" />
        <Button Content="Go to Details" Click="OnGoToDetailsClick" />
    </StackPanel>

</ContentPage>
```

### 代码方式创建 `ContentPage`

```csharp
var page = new ContentPage
{
    Header = "Home",
    Content = new StackPanel
    {
        Spacing = 16,
        Margin = new Thickness(20),
        Children =
        {
            new TextBlock { Text = "Hello, Avalonia!", FontSize = 24 },
            new Button { Content = "Go to Details" }
        }
    }
};
```

当页面托管在 `NavigationPage` 中时，外观如下：

<Image light={ContentPageInNavigationScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

如果没有导航容器，则只会显示页面内容：

<Image light={ContentPageStandaloneScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 将 `ContentPage` 作为窗口根页面

你可以直接将页面设置到窗口上，也可以根据需要再包裹一个 `NavigationPage` 以启用堆栈导航：

```csharp
// 不使用导航
window.Page = new MainPage();

// 启用导航支持
window.Page = new NavigationPage { Content = new MainPage() };
```

### 带可滚动布局的 `ContentPage`

当内容高度可能超过屏幕时，可将布局包裹在 `ScrollViewer` 中：

```xml
<ContentPage xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MyApp.ProfilePage"
             Header="Profile">

    <ScrollViewer>
        <StackPanel Margin="20" Spacing="12">
            <Image Source="avatar.png"
                   Width="80" Height="80"
                   HorizontalAlignment="Center" />
            <TextBlock Text="{Binding FullName}"
                       FontSize="20"
                       HorizontalAlignment="Center" />
            <TextBlock Text="{Binding Bio}"
                       TextWrapping="Wrap" />
        </StackPanel>
    </ScrollViewer>

</ContentPage>
```

### 使用 MVVM 绑定的 `ContentPage`

```xml
<ContentPage xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:vm="clr-namespace:MyApp.ViewModels"
             x:Class="MyApp.LoginPage"
             Header="Sign In"
             x:DataType="vm:LoginViewModel">

    <StackPanel Margin="32" Spacing="16" VerticalAlignment="Center">
        <TextBox Watermark="Email"
                 Text="{Binding Email}" />
        <TextBox Watermark="Password"
                 PasswordChar="*"
                 Text="{Binding Password}" />
        <Button Content="Sign In"
                Command="{Binding SignInCommand}"
                HorizontalAlignment="Stretch" />
        <TextBlock Text="{Binding ErrorMessage}"
                   Foreground="Red"
                   IsVisible="{Binding HasError}" />
    </StackPanel>

</ContentPage>
```

### TopCommandBar

你可以在 `TopCommandBar` 插槽中放置一个 `CommandBar` 或任意其他控件。当该值为 `null` 时，此插槽会自动隐藏。

```xml
<ContentPage xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MyApp.DocumentPage"
             Header="Document">

    <ContentPage.TopCommandBar>
        <CommandBar>
            <CommandBar.PrimaryCommands>
                <AppBarButton Label="Save"  Click="OnSaveClick">
                    <AppBarButton.Icon>
                        <PathIcon Data="M15,9H5V5H15M12,19A3,3 0 0,1..." />
                    </AppBarButton.Icon>
                </AppBarButton>
                <AppBarButton Label="Share" Click="OnShareClick">
                    <AppBarButton.Icon>
                        <PathIcon Data="M18,16.08C17.24,16.08..." />
                    </AppBarButton.Icon>
                </AppBarButton>
            </CommandBar.PrimaryCommands>
            <CommandBar.SecondaryCommands>
                <AppBarButton Label="Export as PDF" Click="OnExportClick" />
                <AppBarButton Label="Print"         Click="OnPrintClick" />
            </CommandBar.SecondaryCommands>
        </CommandBar>
    </ContentPage.TopCommandBar>

    <ScrollViewer>
        <TextBox AcceptsReturn="True" Text="{Binding DocumentText}" />
    </ScrollViewer>

</ContentPage>
```

<Image light={ContentPageTopCommandBarScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### BottomCommandBar

`BottomCommandBar` 的工作方式与之相同，只是它位于页面内容下方。它非常适合作为移动端的操作栏：

```xml
<ContentPage xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MyApp.PhotoPage"
             Header="Photo">

    <ContentPage.BottomCommandBar>
        <CommandBar>
            <CommandBar.PrimaryCommands>
                <AppBarButton Label="Like"    Click="OnLikeClick" />
                <AppBarButton Label="Comment" Click="OnCommentClick" />
                <AppBarButton Label="Share"   Click="OnShareClick" />
            </CommandBar.PrimaryCommands>
        </CommandBar>
    </ContentPage.BottomCommandBar>

    <Image Source="{Binding PhotoSource}" Stretch="UniformToFill" />

</ContentPage>
```

<Image light={ContentPageBottomCommandBarScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 在存在未保存更改时取消导航

`Navigating` 事件处理器是异步的。在决定是否取消导航之前，你可以等待对话框结果或执行异步校验：

```csharp
public partial class EditPage : ContentPage
{
    public bool HasUnsavedChanges { get; set; }

    public EditPage()
    {
        InitializeComponent();
        Navigating += OnNavigatingAsync;
    }

    private async Task OnNavigatingAsync(NavigatingFromEventArgs args)
    {
        if (!HasUnsavedChanges) return;

        var dialog = new ConfirmDialog("你有未保存的更改。要放弃这些更改吗？");
        bool confirmed = await dialog.ShowAsync();
        if (!confirmed)
            args.Cancel = true;
    }
}
```

### 拦截系统返回按钮

你可以重写 `OnSystemBackButtonPressed`，以处理来自硬件或操作系统级别的返回导航：

```csharp
public partial class WizardPage : ContentPage
{
    private int _step;

    protected override bool OnSystemBackButtonPressed()
    {
        if (_step > 0)
        {
            _step--;
            UpdateStep();
            return true; // 已处理，不再继续返回导航
        }
        return false; // 交由 NavigationPage 处理
    }
}
```

### 每次进入页面时刷新数据

```csharp
public partial class CartPage : ContentPage
{
    private readonly CartViewModel _vm;

    public CartPage(CartViewModel vm)
    {
        _vm = vm;
        DataContext = vm;
        InitializeComponent();
    }

    protected override void OnNavigatedTo(NavigatedToEventArgs args)
    {
        base.OnNavigatedTo(args);
        // 每次页面重新回到前台时都会执行，
        // 包括用户从子页面返回时。
        _ = _vm.RefreshCartAsync();
    }
}
```

### 不使用安全区域内边距的全出血布局

适用于包含头图、地图或其他需要延伸到状态栏下方的页面内容：

```xml
<ContentPage xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MyApp.SplashPage"
             AutomaticallyApplySafeAreaPadding="False">

    <Grid>
        <Image Source="hero.jpg" Stretch="UniformToFill" />
        <TextBlock Text="欢迎"
                   VerticalAlignment="Bottom"
                   Margin="20,0,20,40"
                   FontSize="32"
                   Foreground="White" />
    </Grid>

</ContentPage>
```

<Image light={ContentPageSafeAreaDisabledScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

### 带图标的 ContentPage 选项卡

```csharp
var homePage = new ContentPage
{
    Header = "首页",
    Icon   = Geometry.Parse("M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"),
    Content = homeView
};
```

<Image light={ContentPageAsTabScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 另请参阅

- [API 参考](/api/avalonia/controls/contentpage)
- [`ContentPage.cs` 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Page/ContentPage.cs)
- [`Page.cs` 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Page/Page.cs)
