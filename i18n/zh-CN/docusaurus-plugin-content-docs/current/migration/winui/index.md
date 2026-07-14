---
title: WinUI / UWP
description: 将 WinUI 和 UWP 应用迁移到 Avalonia，并保留相近的 XAML、控件与 MVVM 模式。
doc-type: migration
---

WinUI 3 和 UWP 是微软现代化的 XAML 框架，但它们都只面向 Windows。如果你的应用需要覆盖 macOS、Linux、移动端或 Web，这就是一个明确的天花板。Avalonia 使用相近的 XAML 模型，支持相同的 MVVM 模式，并且可以运行在 .NET 能运行的所有地方。

如果你已经熟悉 WinUI 或 UWP，那么你离 Avalonia 其实比想象中更近。XAML 方言、数据绑定系统以及控件模型都很熟悉，主要差异集中在样式系统、命名方式以及少数工作方式不同的控件上。

:::tip[迁移需要帮助？]
Avalonia 团队拥有将 WinUI 和 UWP 应用迁移到 Avalonia 的实战经验。如果你希望获得专家指导，我们可以提供这项服务。更多信息请参阅 [Avalonia Services](https://avaloniaui.net/services)。
:::

## 关键差异

### 样式系统

WinUI 使用 `VisualStateManager`、视觉状态和 Storyboard 来处理控件外观变化。而 Avalonia 则完全用类 CSS 的选择器和伪类来替代这套机制。

**WinUI（VisualStateManager）：**

```xml
<VisualStateManager.VisualStateGroups>
    <VisualStateGroup x:Name="CommonStates">
        <VisualState x:Name="PointerOver">
            <VisualState.Setters>
                <Setter Target="RootBorder.Background" Value="{ThemeResource ButtonBackgroundPointerOver}" />
            </VisualState.Setters>
        </VisualState>
    </VisualStateGroup>
</VisualStateManager.VisualStateGroups>
```

**Avalonia（伪类）：**

```xml
<Style Selector="Button:pointerover /template/ Border#RootBorder">
    <Setter Property="Background" Value="{DynamicResource ButtonBackgroundPointerOver}" />
</Style>
```

Avalonia 的方式更简洁，也更容易组合复用。有关样式系统的完整说明，请参阅 [Styles](/docs/styling/styles)。

#### 从 AdaptiveTrigger 到容器查询

WinUI 在 `VisualStateManager` 中使用 `AdaptiveTrigger`，根据窗口尺寸来适配布局。Avalonia 则使用容器查询来替代，它可以响应任意祖先控件的尺寸变化，而不仅仅是窗口本身。

**WinUI（AdaptiveTrigger）：**

```xml
<VisualStateManager.VisualStateGroups>
    <VisualStateGroup>
        <VisualState x:Name="Narrow">
            <VisualState.StateTriggers>
                <AdaptiveTrigger MinWindowWidth="0" />
            </VisualState.StateTriggers>
            <VisualState.Setters>
                <Setter Target="ContentGrid.Columns" Value="1" />
            </VisualState.Setters>
        </VisualState>
        <VisualState x:Name="Wide">
            <VisualState.StateTriggers>
                <AdaptiveTrigger MinWindowWidth="800" />
            </VisualState.StateTriggers>
            <VisualState.Setters>
                <Setter Target="ContentGrid.Columns" Value="3" />
            </VisualState.Setters>
        </VisualState>
    </VisualStateGroup>
</VisualStateManager.VisualStateGroups>
```

**Avalonia（容器查询）：**

```xml
<Panel Container.Name="root" Container.Sizing="Width">
    <Panel.Styles>
        <ContainerQuery Name="root" Query="max-width:800">
            <Style Selector="UniformGrid#ContentGrid">
                <Setter Property="Columns" Value="1" />
            </Style>
        </ContainerQuery>
        <ContainerQuery Name="root" Query="min-width:800">
            <Style Selector="UniformGrid#ContentGrid">
                <Setter Property="Columns" Value="3" />
            </Style>
        </ContainerQuery>
    </Panel.Styles>

    <UniformGrid x:Name="ContentGrid">
        <!-- content -->
    </UniformGrid>
</Panel>
```

与 WinUI 的 `AdaptiveTrigger` 相比，容器查询有两个明显优势：

- **组件级响应式能力。** `AdaptiveTrigger` 总是基于窗口尺寸判断；而容器查询可以基于任意祖先容器，因此同一个组件无论出现在全宽区域、侧边栏还是对话框中，都能正确响应。
- **不需要视觉状态样板代码。** 你可以在同一个地方同时定义查询条件和样式，而不需要额外声明状态组、状态名或触发器对象。

你还可以在同一个查询中使用 `and` 或 `,` 组合宽度和高度条件，并同时针对任意可样式化属性进行设置，例如字号、间距、可见性、颜色以及布局属性。

有关完整的查询语法，请参阅 [容器查询](/docs/styling/container-queries)。若想了解如何在容器查询、`OnFormFactor`、可回流面板和代码驱动断点之间进行选择，请参阅 [响应式布局](/docs/layout/responsive-layouts)。

### XAML 命名空间

| WinUI / UWP | Avalonia |
|---|---|
| `xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"` | `xmlns="https://github.com/avaloniaui"` |
| `xmlns:muxc="using:Microsoft.UI.Xaml.Controls"` | `xmlns:controls="using:Avalonia.Controls"`（通常并不需要，默认命名空间已覆盖大多数控件） |
| `clr-namespace:` 或 `using:`（用于自定义类型） | `using:`（推荐）或 `clr-namespace:` |

### 数据绑定

核心绑定语法基本一致。WinUI 的 `x:Bind`（编译绑定）在 Avalonia 中也有对应方案：

| WinUI / UWP | Avalonia | 说明 |
|---|---|---|
| `{Binding Path}` | `{Binding Path}` | 相同 |
| `{x:Bind ViewModel.Name}` | 配合 `x:DataType` 使用 `{Binding Name}` | Avalonia 使用 `x:CompileBindings` 和 `x:DataType` 实现编译绑定 |
| `{Binding ElementName=myControl, Path=Text}` | `{Binding #myControl.Text}` | `#name` 简写 |
| `{Binding RelativeSource={RelativeSource Self}}` | `{Binding $self.Property}` | |
| `x:DefaultBindMode` | `x:CompileBindings="True"` | |

### 控件

大多数 WinUI 控件在 Avalonia 中都有直接对应项。主要差异在于命名，以及少数需要额外包支持的控件。

| WinUI / UWP | Avalonia | 说明 |
|---|---|---|
| `NavigationView` | 无直接等价项 | 可使用 [`SplitView`](/api/avalonia/controls/splitview) + [`ListBox`](/api/avalonia/controls/listbox)，或第三方控件 |
| `InfoBar` | 无直接等价项 | 可使用带样式的 `Border` 与自定义内容组合实现 |
| `TeachingTip` | 无直接等价项 | 可使用 `Popup` 或 `Flyout` |
| `PersonPicture` | 无直接等价项 | 可由 `Ellipse` 和 `Image` 组合实现 |
| `RatingControl` | 无直接等价项 | 可使用第三方控件 |
| `NumberBox` | `NumericUpDown` | 名称不同 |
| `Pivot` | `TabControl` | |
| `PivotItem` | `TabItem` | |
| `CalendarView` | `Calendar` | |
| `CalendarDatePicker` | `CalendarDatePicker` | 相同 |
| `CommandBar` | `Menu` 或 `ToolBar` | |
| `ContentDialog` | 用作对话框的 `Window` | Avalonia 使用窗口来实现对话框 |
| `MenuBar` | `Menu` | |
| `MenuFlyout` | `ContextMenu` | |
| `FlipView` | `Carousel` | |
| `ProgressRing` | 无直接内置等价项 | 可使用第三方控件或自定义动画 |
| `SplitView` | `SplitView` | 相同 |
| `TreeView` | `TreeView` | 相同 |
| `ListView` / `GridView` | `ListBox` | 可结合 `ItemTemplate` 和 `WrapPanel` 实现网格布局 |
| `Page` / `Frame` | [`NavigationPage`](/api/avalonia/controls/navigationpage) / `ContentPage` | 参阅 [NavigationPage](/controls/navigation/navigationpage) |

### 导航

Avalonia 提供了 `NavigationPage`，这是一种基于栈的导航系统，与 WinUI 中的 `Frame` + `Page` 模型很接近。它支持带动画的页面入栈和出栈、内置带返回按钮的导航栏，以及模态展示。

```xml
<NavigationPage xmlns="https://github.com/avaloniaui">
    <ContentPage Header="Home">
        <StackPanel Margin="16" Spacing="8">
            <TextBlock Text="主页" FontSize="24" />
            <Button Content="前往详情页" Click="OnGoToDetails" />
        </StackPanel>
    </ContentPage>
</NavigationPage>
```

```csharp
// 将新页面压入导航栈
await Navigation.PushAsync(new DetailsPage());

// 返回上一页
await Navigation.PopAsync();
```

如果应用更偏向轻量方案，也可以通过视图模型组合的方式处理导航，即根据应用状态切换 `ContentControl` 的内容：

```xml
<ContentControl Content="{Binding CurrentPage}" />
```

如果你为每个视图模型类型都注册了数据模板，那么 Avalonia 会自动解析正确的视图。这种方式非常适合那些不需要导航栏或动画页面切换的应用。

有关 `NavigationPage` 的完整说明，请参阅 [NavigationPage](/controls/navigation/navigationpage)。

### 资源与主题

| WinUI / UWP | Avalonia | 说明 |
|---|---|---|
| `ThemeResource` | `DynamicResource` | Avalonia 使用 `DynamicResource` 处理主题感知值 |
| `StaticResource` | `StaticResource` | 相同 |
| `ResourceDictionary.ThemeDictionaries` | `ResourceDictionary.ThemeDictionaries` | 概念相同 |
| `ElementTheme.Light` / `.Dark` | `RequestedThemeVariant` | |
| `AcrylicBrush` | `ExperimentalAcrylicBorder` | API 不同 |

### 文件结构

| WinUI / UWP | Avalonia |
|---|---|
| `.xaml` 扩展名 | `.axaml` 扩展名 |
| `App.xaml` | `App.axaml` |
| `MainWindow.xaml` | `MainWindow.axaml` |
| `.xaml.cs` 代码后置 | `.axaml.cs` 代码后置 |
| `Package.appxmanifest` | 无直接对应项（使用标准 .NET 项目） |

### 线程模型

| WinUI / UWP | Avalonia |
|---|---|
| `DispatcherQueue.TryEnqueue()` | `Dispatcher.UIThread.Post()` |
| `DispatcherQueue.GetForCurrentThread()` | `Dispatcher.UIThread` |
| `CoreDispatcher.RunAsync()` | `Dispatcher.UIThread.InvokeAsync()` |

## 你将获得什么

从 WinUI 迁移到 Avalonia，并不只是为了跨平台。事实上，在若干方面，Avalonia 今天所提供的能力甚至比 WinUI 更多：

- **类 CSS 的样式系统：** 选择器、样式类和伪类让你能以比 `VisualStateManager` 更简洁的方式获得更强的样式控制力。
- **更完善工具链支持的编译绑定：** `x:DataType` 与 `x:CompileBindings` 能在整个项目范围内对绑定路径进行编译期校验。
- **不需要 MSIX / 打包管线：** Avalonia 应用是标准的 .NET 可执行程序，不需要额外 app 清单、打包流程或应用商店依赖。
- **Linux 和 macOS 是一等目标平台：** 不是事后兼容，也不是附加层。

## 另请参阅

- [Avalonia 快速开始](/docs/get-started/create-your-first-project)：创建你的第一个 Avalonia 应用。
- [样式](/docs/styling/styles)：了解 Avalonia 的类 CSS 样式系统。
- [容器查询](/docs/styling/container-queries)：基于尺寸的样式系统，可替代 WinUI 的 AdaptiveTrigger。
- [响应式布局](/docs/layout/responsive-layouts)：使用容器查询和可回流面板构建自适应布局。
- [数据绑定语法](/docs/data-binding/data-binding-syntax)：Avalonia 绑定语法参考。
- [控件参考](/controls)：完整的 Avalonia 控件文档。
