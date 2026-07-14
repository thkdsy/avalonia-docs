---
title: NavigationPage
description: '`NavigationPage` 提供基于栈的页面导航。它包含导航栏、返回按钮以及可选的页面级命令栏。'
doc-type: reference
---

import NavigationBasicExample from '/img/controls/navigationpage/navigation-basic-example.gif';
import NavigationLoginModal from '/img/controls/navigationpage/navigation-login-modal.gif';
import NavigationPagePushedScreenshot from '/img/controls/navigationpage/navigationpage-pushed.png';
import NavigationPageCustomBackScreenshot from '/img/controls/navigationpage/navigationpage-custom-back-button.png';
import NavigationPageNoNavbarScreenshot from '/img/controls/navigationpage/navigationpage-no-navbar.png';
import NavigationPageNoBackButtonScreenshot from '/img/controls/navigationpage/navigationpage-no-back-button.png';
import NavigationPageOverlayBarScreenshot from '/img/controls/navigationpage/navigationpage-overlay-bar.png';
import NavigationPageTopCommandBarScreenshot from '/img/controls/navigationpage/navigationpage-top-commandbar.png';
import NavigationPageModalScreenshot from '/img/controls/navigationpage/navigationpage-modal.png';
import NavigationPageDrawerIntegrationScreenshot from '/img/controls/navigationpage/navigationpage-drawer-integration.png';

`NavigationPage` 提供基于栈的导航能力。页面以独立的 [`ContentPage`](/controls/navigation/contentpage) 实例构建，并通过 `INavigation` 被压入或弹出栈。默认情况下，页面顶部会渲染一个导航栏，显示当前页面标题和返回按钮。你也可以在子页面中渲染可选的命令栏。

`NavigationPage` 直接实现了 `INavigation`。导航 API 既可以通过子页面上的 `Page.Navigation` 访问，也可以通过直接引用 `NavigationPage` 实例来访问。

## 导航栏布局

导航栏包含以下区域：

| 区域 | 内容 |
| ---- | -------- |
| 左侧 | 当栈深度大于 1 时显示返回按钮；当位于 `DrawerPage` 内的根页时显示汉堡切换按钮。 |
| 中间 | 当前活动页面 `Header` 属性中的页面标题。 |
| 右侧 | 通过 `NavigationPage.TopCommandBar` 附加属性设置的 `TopCommandBar` 控件。 |

## 常用属性

你最常使用的通常是以下属性：

| 属性 | 类型 | 默认值 | 说明 |
| -------- | ---- | ------- | ----------- |
| `Content` | `object?` | `null` | 根页面。设置该属性会自动将页面压入栈中。这也是 XAML 的内容属性。 |
| `PageTransition` | `IPageTransition?` | `null` | 在页面压栈或弹栈时播放的过渡动画。 |
| `ModalTransition` | `IPageTransition?` | `null` | 在显示或关闭模态页时播放的过渡动画。 |
| `HasShadow` | `bool` | `false` | 导航栏是否向下方页面内容投射阴影。 |
| `BarHeight` | `double` | `48` | 导航栏的默认高度，单位为设备无关像素。 |
| `EffectiveBarHeight` | `double` | computed | 只读。实际使用的导航栏高度，会考虑页面级覆盖值。 |
| `IsBackButtonVisible` | `bool` | `true` | 控制导航栏中是否允许显示返回按钮的全局开关。 |
| `IsGestureEnabled` | `bool` | `true` | 启用边缘滑动返回手势。 |
| `CanGoBack` | `bool` | computed | 只读。当导航栈中有多于一个条目时为 `true`。 |
| `IsBackButtonEffectivelyVisible` | `bool` | computed | 只读。解析后的返回按钮可见性，会综合 `IsBackButtonVisible`、页面级 `HasBackButton` 附加属性和栈深度。 |
| `ModalStack` | `IReadOnlyList<Page>` | computed | 只读。当前展示的模态页列表，索引 0 为最早显示的页面，最后一个为最上层页面。 |
| `NavigationStack` | `IReadOnlyList<Page>` | computed | 只读。页面栈的有序列表，根页面在前，顶部页面在后。 |
| `IsNavigating` | `bool` | computed | 只读。当导航操作（压栈、弹栈、替换或模态导航）进行中时为 `true`。 |
| `StackDepth` | `int` | computed | 只读。当前导航栈中的页面数量。 |

## 附加属性

你可以将这些属性设置在单独的子 `Page` 实例上，以控制每个页面各自的导航栏行为。

| 属性 | 类型 | 默认值 | 说明 |
| -------- | ---- | ------- | ----------- |
| `NavigationPage.HasNavigationBar` | `bool` | `true` | 控制某个特定页面是否显示导航栏。 |
| `NavigationPage.HasBackButton` | `bool` | `true` | 控制某个特定页面是否显示返回按钮。 |
| `NavigationPage.IsBackButtonEnabled` | `bool` | `true` | 控制某个特定页面上的返回按钮是否可用。 |
| `NavigationPage.BackButtonContent` | `object?` | `null` | 为某个特定页面自定义显示在返回按钮内部的内容。 |
| `NavigationPage.TopCommandBar` | `Control?` | `null` | 当该页面处于活动状态时，渲染在导航栏右侧区域中的控件。 |
| `NavigationPage.BottomCommandBar` | `Control?` | `null` | 当该页面处于活动状态时，渲染在页面内容下方命令栏区域中的控件。 |
| `NavigationPage.BarLayoutBehavior` | `BarLayoutBehavior?` | `null` | 覆盖某个特定页面的导航栏布局方式。 |
| `NavigationPage.BarHeightOverride` | `double?` | `null` | 覆盖某个特定页面的 `BarHeight`。 |

## BarLayoutBehavior 取值

| 值 | 说明 |
| ----- | ----------- |
| `Inset` | 默认值。导航栏会占用布局空间，页面内容会排列在其下方。 |
| `Overlay` | 导航栏会浮动在页面内容上方，内容会延伸到导航栏后面。可通过 [`SafeAreaPadding`](/docs/services/insets-manager) 处理内边距。 |

## 导航方法

导航通过 `INavigation` 接口完成，你可以在任意子页面上通过 `Page.Navigation` 访问它，也可以直接在 `NavigationPage` 实例上访问。

| 方法 | 说明 |
| ------ | ----------- |
| `PushAsync(page)` | 使用已配置的 `PageTransition` 压入一个页面。 |
| `PushAsync(page, transition)` | 使用指定过渡动画压入页面。传入 `null` 表示无动画。 |
| `PopAsync()` | 使用已配置的 `PageTransition` 弹出顶部页面。 |
| `PopAsync(transition)` | 使用指定过渡动画弹出页面。传入 `null` 表示无动画。 |
| `PopToRootAsync()` | 将所有页面弹出，直到返回根页面。 |
| `PopToRootAsync(transition)` | 使用指定过渡动画将所有页面弹出到根页面。 |
| `PopToPageAsync(page)` | 弹出指定页面之上的所有页面。 |
| `PopToPageAsync(page, transition)` | 使用指定过渡动画弹出指定页面之上的所有页面。 |
| `ReplaceAsync(page)` | 用一个新页面替换当前顶部页面。 |
| `ReplaceAsync(page, transition)` | 使用指定过渡动画，用一个新页面替换当前顶部页面。 |
| `InsertPage(page, before)` | 在栈中另一个页面之前立即插入页面，不带动画。 |
| `RemovePage(page)` | 从栈中移除某个特定页面，不带动画。 |
| `PushModalAsync(page)` | 以模态方式显示一个页面，并覆盖整个 `NavigationPage`。 |
| `PushModalAsync(page, transition)` | 使用指定过渡动画以模态方式显示页面。 |
| `PopModalAsync()` | 关闭最上层模态页。 |
| `PopModalAsync(transition)` | 使用指定过渡动画关闭最上层模态页。 |
| `PopAllModalsAsync()` | 关闭所有模态页。 |
| `PopAllModalsAsync(transition)` | 使用指定过渡动画关闭所有模态页。 |

当 `StackDepth > 1` 时，系统返回按钮会自动调用 `PopAsync()`。

## 事件

| 事件 | 参数类型 | 说明 |
| ----- | --------- | ----------- |
| `Pushed` | `NavigationEventArgs` | 在页面被压入栈后触发。`args.Page` 是被压入的页面。 |
| `Popped` | `NavigationEventArgs` | 在页面从栈中弹出后触发。`args.Page` 是被移除的页面。 |
| `PoppedToRoot` | `NavigationEventArgs` | 在 `PopToRootAsync` 完成后触发。`args.Page` 是新的顶部页面（根页面）。 |
| `PageInserted` | `PageInsertedEventArgs` | 在 `InsertPage` 完成后触发。`args.Page` 是被插入的页面；`args.Before` 是它插入到其前面的页面。 |
| `PageRemoved` | `PageRemovedEventArgs` | 在 `RemovePage` 完成后触发。`args.Page` 是被移除的页面。 |
| `ModalPushed` | `ModalPushedEventArgs` | 在模态页显示后触发。`args.Modal` 是该模态页面。 |
| `ModalPopped` | `ModalPoppedEventArgs` | 在模态页关闭后触发。`args.Modal` 是被关闭的页面。 |

## 示例

### 基础 NavigationPage

<Image light={NavigationBasicExample} alt="Animation showing a home page with a navigation bar; clicking 'Go to Details' slides to a detail page with Refresh and Share command buttons in the navigation bar" position="center" maxWidth={400} cornerRadius="true" />
<br />

这是一个基础版 `NavigationPage` 实现：在首页和详情页之间进行双页导航，并在详情页中额外显示顶部命令栏。

<Tabs groupId="basicExample">

  <TabItem label="MainWindow.axaml" value="mainWindowXaml">
    ```xml
    <NavigationPage>
        <views:HomePage/>
    </NavigationPage>
    ```
  </TabItem>

  <TabItem label="HomePage.axaml" value="homePageXaml">
    ```xml
    <ContentPage xmlns="https://github.com/avaloniaui"
               xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
               x:Class="TestApp1.Views.HomePage"
               Header="Home">
      <StackPanel HorizontalAlignment="Center"
                  VerticalAlignment="Center"
                  Spacing="12">
          <TextBlock Text="Welcome to the Home Page"
                     FontSize="22"
                     FontWeight="SemiBold"
                     HorizontalAlignment="Center"/>
          <TextBlock Text="Click the button to open the detail page."
                     HorizontalAlignment="Center"/>
          <Button Content="Go to Details"
                  Click="OnGoToDetailsClick"
                  HorizontalAlignment="Center"/>
      </StackPanel>
    </ContentPage>
    ```
  </TabItem>

  <TabItem label="HomePage.axaml.cs" value="homePageCs">
    ```csharp
    public partial class HomePage : ContentPage
    {
        public HomePage()
        {
            InitializeComponent();
        }

        private async void OnGoToDetailsClick(object? sender, RoutedEventArgs e)
        {
            if (Navigation is not null)
                await Navigation.PushAsync(new DetailPage());
        }
    }
    ```
  </TabItem>

  <TabItem label="DetailPage.axaml" value="detailPageXaml">
    ```xml
    <ContentPage xmlns="https://github.com/avaloniaui"
                 xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
                 x:Class="TestApp1.Views.DetailPage"
                 Header="Details">

      <ContentPage.TopCommandBar>
          <StackPanel Orientation="Horizontal" Spacing="8">
              <Button Content="Refresh" Click="OnRefreshClick"/>
              <Button Content="Share" Click="OnShareClick"/>
          </StackPanel>
      </ContentPage.TopCommandBar>

      <StackPanel HorizontalAlignment="Center"
                  VerticalAlignment="Center"
                  Spacing="12">
          <TextBlock Text="Detail Page"
                     FontSize="22"
                     FontWeight="SemiBold"
                     HorizontalAlignment="Center"/>
          <TextBlock Text="Use the back button to return to Home."
                     HorizontalAlignment="Center"/>
      </StackPanel>
    </ContentPage>
    ```
  </TabItem>

</Tabs>

### 压入和弹出页面

你可以在任意子 `ContentPage` 中访问 `Navigation`：

```csharp
// 压入一个新页面
private async void OnGoToDetailsClick(object? sender, RoutedEventArgs e)
{
    await Navigation.PushAsync(new DetailPage());
}

// 返回上一页
private async void OnBackClick(object? sender, RoutedEventArgs e)
{
    await Navigation.PopAsync();
}

// 从栈中的任意位置返回根页面
private async void OnGoHomeClick(object? sender, RoutedEventArgs e)
{
    await Navigation.PopToRootAsync();
}
```

### 跟踪栈深度和当前页面

```csharp
private void UpdateStatus()
{
    StatusText.Text = $"Depth: {Navigation.StackDepth}";
    HeaderText.Text = $"Current: {Navigation.NavigationStack[^1].Header}";
    CanGoBackText.Text = Navigation.CanGoBack ? "Can go back" : "At root";
}
```

### 隐藏导航栏

<Image light={NavigationPageNoNavbarScreenshot} alt="A detail page with the navigation bar hidden, showing command buttons at the top without a title or back button" position="center" maxWidth={400} cornerRadius="true"/>
<br />

<Tabs>

  <TabItem label="XAML" value="xaml">
  ```xml
  <ContentPage NavigationPage.HasNavigationBar="False">
      <!-- content -->
  </ContentPage>
  ```
  </TabItem>

  <TabItem label="C#" value="cs">
  ```csharp
  NavigationPage.SetHasNavigationBar(page, false);
  ```
  </TabItem>

</Tabs>

### 隐藏返回按钮

<Image light={NavigationPageNoBackButtonScreenshot} alt="A navigation bar showing the 'Details' page title without a back button" position="center" maxWidth={400} cornerRadius="true"/>
<br />

<Tabs>

  <TabItem label="XAML" value="xaml">
  ```xml
  <ContentPage NavigationPage.HasBackButton="False">
      <!-- 用户无法从这里返回 -->
  </ContentPage>
  ```
  </TabItem>

  <TabItem label="C#" value="cs">
  ```csharp
  NavigationPage.SetHasBackButton(page, false);
  ```
  </TabItem>

</Tabs>

### 自定义返回按钮

你可以用自定义文本或控件替换默认的返回箭头。

在这个示例中，通过在 `DetailPage.axaml.cs` 的代码后置中添加下面这行代码，把详情页中的返回箭头替换为文本 “Go Home”。

<Image light={NavigationPageCustomBackScreenshot} alt="A navigation bar showing 'Go Home' as a custom back button label next to the Details page title" position="center" maxWidth={400} cornerRadius="true"/>
<br />

```csharp
NavigationPage.SetBackButtonContent(this, new TextBlock { Text = "Go Home" });
```

### 按页面自定义 TopCommandBar

创建一个自定义命令栏，并将其分配给目标页面。当该目标页面位于栈顶时，它会渲染在导航栏区域内。

此示例展示了一组在查看详情页时显示的自定义按钮。它们被放在一个名为 `TopBar` 的自定义 [`UserControl`](/controls/primitives/usercontrol) 中，并定义在独立的 .axaml 和 .axaml.cs 文件里。随后便可以在 `DetailPage.axaml.cs` 的代码后置文件中引用 `TopBar`。

<Image light={NavigationPageTopCommandBarScreenshot} alt="A navigation bar with a back button, 'Details' title, and a top command bar containing Refresh, Share, Filter, and Add New buttons on the right" position="center" maxWidth={400} cornerRadius="true"/>
<br />

<Tabs>

  <TabItem label="TopBar.axaml" value="topBarXaml">
    ```xml
    <UserControl xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        x:Class="TestApp1.Views.TopBar">
    
      <CommandBar>
          <CommandBar.PrimaryCommands>
              <CommandBarButton Label="Refresh"/>
              <CommandBarButton Label="Share"/>
              <CommandBarButton Label="Filter"/>
              <CommandBarButton Label="Add New"/>
          </CommandBar.PrimaryCommands>
      </CommandBar>
    
    </UserControl>
    ```
  </TabItem>

  <TabItem label="TopBar.axaml.cs" value="topBarCs">
    ```csharp
    using Avalonia.Controls;

    namespace TestApp1.Views;

    public partial class TopBar : UserControl
    {
        public TopBar()
        {
            InitializeComponent();
        }
    }
    ```
  </TabItem>

  <TabItem label="DetailPage.axaml.cs" value="detailPageCs">
    ```csharp
    using Avalonia.Controls;

    namespace TestApp1.Views;

    public partial class DetailPage : ContentPage
    {
        public DetailPage()
        {
            InitializeComponent();
            NavigationPage.SetTopCommandBar(this, new TopBar());
        }
    }
    ```
  </TabItem>

</Tabs>

### 页面过渡动画

如果你希望为页面压栈和弹栈添加动画，可以在 `NavigationPage` 所在位置的代码后置文件中（本例中为 `MainWindow.axaml.cs`）设置 `NavigationPage` 的 `PageTransition` 属性。

```csharp
using Avalonia.Animation; // 添加此 using 语句以使用 Avalonia 内置动画

public partial class MainWindow : Window
{
    public MainWindow()
    {
        InitializeComponent();

        // 选中名为 "Nav" 的 NavigationPage 控件
        var nav = this.FindControl<NavigationPage>("Nav");                                         

        // 水平滑动（默认）
        nav.PageTransition = new PageSlide(TimeSpan.FromMilliseconds(300))

        // 交叉淡入淡出                          
        nav.PageTransition = new CrossFade(TimeSpan.FromMilliseconds(300));

        // 不使用动画
        nav.PageTransition = null;
    }
}
```

### 模态页面

以模态方式显示一个覆盖整个 `NavigationPage` 区域的页面。

这个示例展示了一个模拟登录页。它由首页通过 `PushModalAsync` 调起。点击 Login 会通过 `PopModalAsync` 关闭当前模态页，而点击 Cancel 则会通过 `PopAllModalsAsync` 关闭所有已打开的模态页。

<Image light={NavigationLoginModal} alt="Animation showing a modal login page with email and password fields being presented over the home page and dismissed" position="center" maxWidth={400} cornerRadius="true" />
<br />

<Tabs>

  <TabItem label="HomePage.axaml" value="homePageXaml">
  ```xml
  <ContentPage xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="TestApp1.Views.HomePage"
             Header="Home">
    <StackPanel HorizontalAlignment="Center"
                VerticalAlignment="Center"
                Spacing="12">
        <TextBlock Text="Welcome to the Home Page"
                   FontSize="22"
                   FontWeight="SemiBold"
                   HorizontalAlignment="Center"/>
        <Button Content="Login"
                Click="OnLoginClick"
                HorizontalAlignment="Center"/>
    </StackPanel>
  </ContentPage>
  ```
  </TabItem>

  <TabItem label="HomePage.axaml.cs" value="homePageCs">
  ```csharp
  using Avalonia.Controls;
  using Avalonia.Interactivity;

  namespace TestApp1.Views;

  public partial class HomePage : ContentPage
  {
      public HomePage()
      {
          InitializeComponent();
      }

      private async void OnLoginClick(object? sender, RoutedEventArgs e)
      {
          if (Navigation is not null)
              await Navigation.PushModalAsync(new LoginPage());
      }
  }
  ```
  </TabItem>

  <TabItem label="LoginPage.axaml" value="loginPageXaml">
  ```xml
  <ContentPage xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="TestApp1.Views.LoginPage"
             Header="Login">

    <StackPanel HorizontalAlignment="Center"
                VerticalAlignment="Center"
                Spacing="20">

        <TextBlock Text="Sign In"
                   FontSize="24"
                   HorizontalAlignment="Center"/>

        <StackPanel Spacing="4">
            <TextBlock Text="Email"/>
            <TextBox Name="EmailBox"
                     Watermark="Enter your email"/>
        </StackPanel>

        <StackPanel Spacing="4">
            <TextBlock Text="Password"/>
            <TextBox Name="PasswordBox"
                     Watermark="Enter your password"
                     PasswordChar="•"/>
        </StackPanel>

        <Button Content="Login"
                Click="OnLoginClick"/>

        <Button Content="Cancel"
                Click="OnCancelClick"/>

    </StackPanel>
  </ContentPage>
  ```
  </TabItem>

  <TabItem label="LoginPage.axaml.cs" value="loginPageCs">
  ```csharp
  using Avalonia.Controls;
  using Avalonia.Interactivity;

  namespace TestApp1.Views;

  public partial class LoginPage : ContentPage
  {
      public LoginPage()
      {
          InitializeComponent();
      }

      private async void OnLoginClick(object? sender, RoutedEventArgs e)
      {
          // This is where you specify your actual login auth logic
          if (Navigation is not null)
              await Navigation.PopModalAsync();
      }

      private async void OnCancelClick(object? sender, RoutedEventArgs e)
      {
          // Cancel button dismisses all open modals
          if (Navigation is not null)
              await Navigation.PopAllModalsAsync();
      }
  }
  ```
  </TabItem>

</Tabs>

### 统计当前打开的模态页数量

```csharp
int modalCount = Navigation.ModalStack.Count;
```

### 模态过渡动画

如果你希望为模态页的显示和关闭添加动画，可以在 `NavigationPage` 控件上设置 `ModalTransition` 属性。它的使用方式与 [`PageTransition`](#页面过渡动画) 类似。

<Tabs>

  <TabItem label="XAML" value="xaml">
  ```xml
  <NavigationPage>
    <NavigationPage.ModalTransition>
        <CrossFade Duration="0:0:0.30"/>
    </NavigationPage.ModalTransition>
  </NavigationPage>
  ```
  </TabItem>

  <TabItem label="C#" value="cs">
  ```csharp
  using Avalonia.Animation; // 添加此 using 语句以使用 Avalonia 内置动画

  public partial class MainWindow : Window
  {
      public MainWindow()
      {
          InitializeComponent();

          // 选中名为 "Nav" 的 NavigationPage 控件
          var nav = this.FindControl<NavigationPage>("Nav");

          // 设置交叉淡入淡出的模态过渡动画
          nav.ModalTransition = new CrossFade(TimeSpan.FromMilliseconds(300));
      }
  }
  ```
  </TabItem>

</Tabs>

### 覆盖式导航栏

使用 `BarLayoutBehavior.Overlay` 可以让导航栏浮动在大图头图或地图之上。

```xml
<ContentPage NavigationPage.BarLayoutBehavior="Overlay"
             Header="Map">
    <!-- 内容会延伸到导航栏后方 -->
    <MapView />
</ContentPage>
```

<Image light={NavigationPageOverlayBarScreenshot} alt="A page with the navigation bar overlaid on top of content, with a back button and share icon floating above a game app header image" position="center" maxWidth={250} cornerRadius="true"/>

### 登录成功后替换登录页

登录成功后，可以替换根页面，这样用户就无法再返回到登录页。

```csharp
private async void OnLoginSuccess()
{
    // 移除登录页并压入主页面
    Navigation.InsertPage(new MainPage(), before: Navigation.NavigationStack[0]);
    await Navigation.PopToRootAsync(transition: null);
}
```

或者使用 `ReplaceAsync` 直接替换顶部页面：

```csharp
await Navigation.ReplaceAsync(new HomePage());
```

### DrawerPage 集成

当 `NavigationPage` 作为 [`DrawerPage`](/controls/navigation/drawerpage) 的 `Content` 使用时，在栈的根页面上，导航栏左侧会出现汉堡菜单切换按钮。当用户继续向更深层页面导航时，它会消失。

```csharp
var shell = new DrawerPage
{
    Drawer  = new MenuPage(),
    Content = new NavigationPage { Content = new HomePage() }
};
window.Page = shell;
```

<Image light={NavigationPageDrawerIntegrationScreenshot} alt="A navigation bar at the root page showing a hamburger toggle button on the left to open the drawer" position="center" maxWidth={250} cornerRadius="true"/>

### 禁用返回滑动手势

```csharp
navPage.IsGestureEnabled = false;
```

## 另请参阅

- [API 参考](/api/avalonia/controls/navigationpage)
- [源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Page/NavigationPage.cs)
