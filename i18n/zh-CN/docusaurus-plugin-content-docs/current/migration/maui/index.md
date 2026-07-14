---
title: .NET MAUI
description: 从 .NET MAUI 迁移到 Avalonia，或使用 Avalonia MAUI Backend 扩展 MAUI 应用。
doc-type: migration
---

如果你是 .NET MAUI 开发者，那么通往 Avalonia 有两条路径。你可以保留现有的 MAUI 代码库，并使用 Avalonia MAUI Backend 将应用扩展到新的平台；也可以直接把应用迁移到 Avalonia，以获得对 UI 框架的完整控制权。本页会介绍这两种方案。

:::tip[需要帮助？]
Avalonia 团队拥有处理 MAUI 代码库的实战经验。无论你是希望接入 Avalonia MAUI Backend，还是想彻底迁移到 Avalonia，我们都可以提供这项服务。更多信息请参阅 [Avalonia Services](https://avaloniaui.net/services)。
:::

## 方案 1：Avalonia MAUI 后端

Avalonia MAUI Backend 允许你保留现有的 .NET MAUI 代码库，并将渲染层替换为 Avalonia。你已有的 MAUI 代码、控件、handler 和布局仍然可以继续工作，但它们会通过 Avalonia 的跨平台引擎来渲染，而不是依赖各平台的原生控件。

这会让你的 MAUI 应用获得 MAUI 自身尚不支持的平台能力：

- **桌面 Linux：** 在 Ubuntu、Debian、Fedora 等发行版上提供一等公民级别的桌面支持，使用的正是今天已在生产环境中支撑高要求桌面应用的 Avalonia 渲染器。
- **嵌入式 Linux：** Avalonia 已经可以运行在嵌入式 Linux 设备上，从 Raspberry Pi 面板到工业 HMI。MAUI Backend 会把这些能力带给你的 MAUI 应用。
- **WebAssembly：** 将你的 MAUI 应用部署到浏览器中，客户端无需任何原生依赖。
- **更好的桌面性能：** 在 Windows 和 macOS 上，Avalonia 后端可直接接入 Avalonia 成熟的桌面能力。早期在 macOS 上的测试显示，相比 Mac Catalyst 方案，性能有显著提升。

由于 Avalonia 会自行绘制每一个控件，因此你的 MAUI 应用无论运行在 Windows、macOS、Linux、移动设备还是浏览器标签页中，外观和行为都会保持一致。

### 工作原理

从本质上说，Avalonia MAUI Backend 构建了一套统一的 handler，用于把 MAUI 控件映射到 Avalonia 控件上。当你在 MAUI 中创建一个 [`Button`](/api/avalonia/controls/button) 时，它在所有平台上都会渲染为 Avalonia 的 `Button`，而不再是特定平台的原生按钮控件。

MAUI 的布局系统也是类似的。MAUI 自己负责位置和约束的计算，而 Avalonia 后端则会严格按照 MAUI 的结果来摆放控件。实际效果上，这意味着许多标准 MAUI 布局控件无需修改即可工作。

依赖 `SkiaSharp` 和 `Microsoft.Maui.Graphics` 的库也能够工作，因为 Avalonia 本身就包含基于 SkiaSharp 的渲染器。这使得许多自绘控件只需极少修改就能完成映射。

### 这对你的代码意味着什么

你不需要重写整个应用。你只需把 Avalonia MAUI Backend 库添加到现有项目中，并把目标平台扩展到新的平台即可。你的 MAUI XAML、视图模型、服务和业务逻辑都可以保持不变。

像 Resizetizer 这样的构建期工具也依然可以使用。在构建过程中，Resizetizer 会把图片、SVG 和字体转换为资源，而 Avalonia 后端会自动将这些资源映射为 Avalonia 资源。

### 当前状态

Avalonia 团队正在与 MAUI 生态中的工程师协作开发 MAUI Backend。目标是在 .NET 11 发布时同步推出稳定版本。预览版会跟随 .NET MAUI 的发布节奏推进，并通过 CI 提供 nightly 构建。

当前的首要重点是 Linux 和 WebAssembly，因为 MAUI 目前并不支持这两个平台。该后端同样可以运行在 Windows 和 macOS 上，并计划逐步支持 Avalonia 的所有目标平台。

这个项目不会分叉 .NET MAUI。所有为支持集成所需的改动，都会尽量向上游贡献到官方 .NET MAUI 仓库中，从而让整个生态都能受益。

:::note
Avalonia MAUI Backend 仍处于活跃开发中。你可以在 [avaloniaui.net](https://avaloniaui.net) 登记关注，以获取更新和抢先体验资格。
:::

## 方案 2：迁移到 Avalonia

如果你希望完全掌控 UI 框架，或者你的应用需要超出 MAUI 能力范围的特性（例如类 CSS 样式、自定义渲染、高级桌面功能），那么你可以直接把 MAUI 应用迁移到 Avalonia。

### 前置条件

开始之前，请确保你已经具备以下条件：

- 已安装 **.NET 8 或更高版本**。Avalonia 11+ 最低要求是 .NET 8。
- 已安装 **Avalonia 模板**。请在命令行运行 `dotnet new install Avalonia.Templates`。
- **你现有的 MAUI 项目可以成功编译和运行。** 在开始之前先修复已有构建问题。有一个可正常运行的基线，能让你更容易验证每一步迁移结果。
- 熟悉 XAML 和 MVVM 模式。如果你本来就在使用 MAUI，那么这部分通常已经具备。

### 不同的渲染模型

MAUI 和 Avalonia 之间最重要的区别，在于它们的渲染方式。

**MAUI** 会把控件映射到平台原生控件上。iOS 上的 `Button` 实际是 `UIButton`，Android 上的 `Button` 则是 `Android.Widget.Button`。这意味着各个平台的外观可能会有细微甚至明显差异，而平台特定的 bug 也比较常见。

**Avalonia** 会使用 Skia 或 Direct2D 自行绘制每一个控件。在 Avalonia 中，一个 `Button` 在 Windows、macOS、Linux、iOS、Android 和 WebAssembly 上看起来都是一致的。决定外观的是你选择的主题，而不是目标平台。

这种差异会影响很多方面：样式系统、布局精度、调试方式，以及你最终需要编写多少平台特定代码。

### 关键差异

#### XAML 方言

这两个框架都使用 XAML，但它们的方言并不相同。MAUI 的 XAML 延续自 Xamarin.Forms 的约定，而 Avalonia 的 XAML 则更接近 WPF。

| .NET MAUI | Avalonia | 说明 |
|---|---|---|
| `xmlns="http://schemas.microsoft.com/dotnet/2021/maui"` | `xmlns="https://github.com/avaloniaui"` | |
| 用于编译绑定的 `x:DataType` | `x:DataType` + `x:CompileBindings` | 概念相同，但配置方式略有不同 |
| `{Binding Path}` | `{Binding Path}` | 语法相同 |
| `{Binding Source={RelativeSource Self}}` | `{Binding $self.Property}` | 简写语法 |
| `{Binding Source={x:Reference myControl}, Path=Text}` | `{Binding #myControl.Text}` | `#name` 简写 |

#### 布局

MAUI 和 Avalonia 都使用面板来组织布局，但名称和具体行为有所不同。

| .NET MAUI | Avalonia | 说明 |
|---|---|---|
| `StackLayout` / `VerticalStackLayout` | `StackPanel` | Avalonia 使用 `Orientation` 属性控制方向 |
| `HorizontalStackLayout` | `StackPanel Orientation="Horizontal"` | |
| `Grid` | `Grid` | 概念相同。Avalonia 支持 `ColumnDefinitions="Auto,*"` 这类简写 |
| `FlexLayout` | `WrapPanel` | 不是严格的一一对应，但可覆盖大多数使用场景 |
| `AbsoluteLayout` | `Canvas` | |
| `ScrollView` | `ScrollViewer` | |
| `Frame` | `Border` | |
| `ContentView` | `UserControl` or `ContentControl` | |
| `Padding`, `Margin` | `Padding`, `Margin` | 相同 |
| 布局上的 `Spacing` | `StackPanel` 上的 `Spacing` | 概念相同 |

#### 控件

| .NET MAUI | Avalonia | 说明 |
|---|---|---|
| `Entry` | [`TextBox`](/api/avalonia/controls/textbox) | |
| `Editor` | 设置了 `AcceptsReturn="True"` 的 `TextBox` | |
| `Label` | `TextBlock` | |
| `Button` | `Button` | 相同 |
| `ImageButton` | 带图像内容的 `Button` | |
| `CheckBox` | `CheckBox` | 相同 |
| `Switch` | `ToggleSwitch` | |
| `Slider` | `Slider` | 相同 |
| `Stepper` | `NumericUpDown` | |
| `Picker` | `ComboBox` | |
| `DatePicker` | `DatePicker` | 相同 |
| `TimePicker` | `TimePicker` | 相同 |
| `ActivityIndicator` | `ProgressBar IsIndeterminate="True"` | |
| `ProgressBar` | `ProgressBar` | 相同 |
| `ListView` / `CollectionView` | `ListBox` 或 `ItemsRepeater` | 若需要支持虚拟化的自定义布局，可使用 `ItemsRepeater` |
| `CarouselView` | `Carousel` | |
| `TableView` | 无直接等价项 | 可使用 `DataGrid`，或通过面板自行组合 |
| `WebView` | 无内置等价项 | 可使用第三方控件 |
| `RefreshView` | `RefreshContainer` | |
| `SearchBar` | 带自定义样式的 `TextBox` | 若需要建议列表，也可以使用 `AutoCompleteBox` |
| `Shell` | 无等价项 | Avalonia 不强制规定一套固定导航框架 |
| `FlyoutPage` | `SplitView` | |
| `TabbedPage` | `TabControl` | |
| [`NavigationPage`](/api/avalonia/controls/navigationpage) | `NavigationPage` | 参阅 [NavigationPage](/controls/navigation/navigationpage) |
| `ContentPage` | `ContentPage` | 通常用于 `NavigationPage` 内部 |
| `BoxView` | `Border` 或 `Rectangle` | |

#### 样式系统

MAUI 使用资源字典配合 `Style` 元素并以类型为目标来定义样式。Avalonia 则使用类 CSS 的选择器。

**MAUI：**

```xml
<Style TargetType="Button">
    <Setter Property="BackgroundColor" Value="SteelBlue" />
    <Setter Property="TextColor" Value="White" />
</Style>
```

**Avalonia：**

```xml
<Style Selector="Button">
    <Setter Property="Background" Value="SteelBlue" />
    <Setter Property="Foreground" Value="White" />
</Style>
```

表面上看语法很相似，但 Avalonia 的选择器支持更多能力。你可以按类名、名称、状态、嵌套关系等条件来匹配控件：

```xml
<Style Selector="Button.primary:pointerover">
    <Setter Property="Background" Value="LightBlue" />
</Style>
```

MAUI 使用 `VisualStateManager` 处理交互状态。Avalonia 则将伪类（如 `:pointerover`、`:pressed`、`:disabled`、`:checked`）直接作为选择器的一部分，表达方式更简洁。完整参考请参阅 [Styles](/docs/styling/styles)。

#### 导航

MAUI 内置了 `Shell`、`NavigationPage`、`FlyoutPage` 和 `TabbedPage` 等导航模式。Avalonia 提供了 `NavigationPage`，这是一套基于栈的导航系统。如果你用过 MAUI 的 `NavigationPage`，会觉得它非常熟悉。它支持带动画的页面入栈和出栈、内置带返回按钮的导航栏，以及模态展示。

```xml
<NavigationPage xmlns="https://github.com/avaloniaui">
    <ContentPage Header="主页">
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

如果你的应用更偏好轻量方案，也可以通过视图模型组合的方式来处理导航，即根据应用状态切换 `ContentControl` 的内容：

```xml
<ContentControl Content="{Binding CurrentPage}" />
```

如果你为每种视图模型类型都注册了数据模板，Avalonia 就会自动解析正确的视图。这种方式非常适合那些不需要导航栏或动画页面切换的应用。

完整说明请参阅 [NavigationPage](/controls/navigation/navigationpage)。

#### 平台特定代码

在 MAUI 中，不同平台之间的差异会频繁暴露出来。你最终往往需要编写 handler、自定义 renderer，或者使用 `#if` 条件编译块来修复特定平台上的行为问题。

在 Avalonia 中，平台特定代码通常很少见。由于 Avalonia 自己掌控渲染过程，同一份代码在各个平台上通常会产生一致结果。而当你确实需要平台特定行为（例如访问原生 API）时，Avalonia 也会提供更清晰的抽象层，而无需你去继承 renderer。

#### 文件结构

| .NET MAUI | Avalonia |
|---|---|
| `.xaml` 扩展名 | `.axaml` 扩展名 |
| `App.xaml` | `App.axaml` |
| `MainPage.xaml` | `MainWindow.axaml` |
| `.xaml.cs` 代码后置 | `.axaml.cs` 代码后置 |
| 含平台专属代码的 `Platforms/` 文件夹 | 通常不需要平台目录 |
| `MauiProgram.cs` builder | 带有 `AppBuilder` 的 `Program.cs` |

#### 线程模型

| .NET MAUI | Avalonia |
|---|---|
| `MainThread.BeginInvokeOnMainThread()` | `Dispatcher.UIThread.Post()` |
| `MainThread.InvokeOnMainThreadAsync()` | `Dispatcher.UIThread.InvokeAsync()` |
| `MainThread.IsMainThread` | `Dispatcher.UIThread.CheckAccess()` |

### 迁移步骤

目前并没有从 MAUI 自动转换到 Avalonia 的工具。迁移过程需要手动完成，但由于两者之间存在很多相似点，这个过程仍然是可控的。建议按层次逐步推进你的应用迁移。

#### 1. 创建新的 Avalonia 项目

先使用模板创建一个全新的 Avalonia 项目：

```bash
dotnet new avalonia.mvvm -n MyApp
```

这样你会得到一个可正常工作的项目结构，其中包含 `App.axaml`、`MainWindow.axaml` 和视图模型基类。不要尝试直接在原地改造 MAUI 的 `.csproj`。

#### 2. 迁移模型和服务

把你的模型类、服务以及任何与平台无关的业务逻辑复制到新项目中。这些部分通常不需要修改，因为它们本身并不依赖 UI 框架。

#### 3. 迁移视图模型

复制你的视图模型。如果你使用的是 `CommunityToolkit.Mvvm`，那么它通常可以直接在 Avalonia 中使用，无需修改。如果你的视图模型引用了 MAUI 特有类型（例如 `Microsoft.Maui.Controls.Application`），则需要将这些引用替换为 Avalonia 对应实现。

#### 4. 转换 XAML 文件

对于 MAUI 项目中的每一个 `.xaml` 页面，都在 Avalonia 项目中创建对应的 `.axaml` 文件。你可以参考上文[关键差异](#key-differences)中的控件与布局映射表，来转换控件名称、属性和绑定语法。

每个文件通常都要做这些关键改动：
- 将根 XML 命名空间替换为 `xmlns="https://github.com/avaloniaui"`
- 重命名控件（例如 `Entry` 改为 `TextBox`，`Label` 改为 `TextBlock`）
- 用 Avalonia 的样式选择器和伪类替换 `VisualStateManager` 代码块
- 在需要时调整绑定语法（例如 `{x:Reference}` 改为 `#name`，`RelativeSource Self` 改为 `$self`）

#### 5. 转换样式与资源

迁移你的资源字典。把基于 `TargetType` 的样式替换为基于 Avalonia 选择器的样式。示例可参阅上面的[样式系统](#styling)章节。

#### 6. 替换导航实现

如果你的 MAUI 应用使用了 `Shell` 或 `NavigationPage`，则需要将其替换为 Avalonia 的 `NavigationPage`，或者改用视图模型组合模式。详见上面的[导航](#navigation)章节。

#### 7. 处理平台特定代码

检查 MAUI `Platforms/` 文件夹中的所有代码。大多数为修复平台渲染差异而写的变通逻辑，在 Avalonia 中通常都不再需要。对于真正的平台 API 访问（如摄像头、文件系统、传感器），可以继续把 .NET MAUI Essentials 当作独立包使用，或者改为直接调用平台 API。

### 验证

迁移完成后，请验证应用是否工作正常：

1. **构建项目。** 修复所有由于类型重命名或命名空间调整遗漏导致的编译错误。
2. **在主平台上运行。** 确认主窗口能够正常加载，导航也能正常工作。
3. **在第二个平台上测试。** 在不同操作系统上运行一次（例如你在 Windows 开发，就再到 macOS 上试一次），确认跨平台渲染一致性。
4. **走查核心用户流程。** 验证数据绑定、命令和输入处理是否符合预期。
5. **检查样式。** 确认视觉效果符合你的设计意图，尤其要关注以前依赖 `VisualStateManager` 的悬停和按下状态。

### 迁移后你会获得什么

从 MAUI 迁移到 Avalonia 之后，你日常开发方式会发生这些变化：

- **像素级一致性：** 你的 UI 会在所有平台上以一致方式渲染。不必再追着只在 Android 上出现的布局 bug，或只在 iOS 上出现的样式问题跑。
- **一等公民级桌面支持：** Avalonia 从一开始就是为桌面而构建的。窗口管理、菜单、键盘导航和多窗口支持都能按预期工作。
- **Linux 支持：** Avalonia 可以原生运行在 Linux 上，而 MAUI 完全不支持 Linux。
- **没有原生控件包装层：** 你不再需要穿透一层层平台抽象去调试。XAML 中写出来的内容，就是最终渲染出来的内容。
- **WebAssembly：** Avalonia 支持通过 WebAssembly 部署到浏览器，这是 MAUI 不具备的目标平台。

## 另请参阅

- [Avalonia 快速开始](/docs/get-started/create-your-first-project)：创建你的第一个 Avalonia 应用。
- [样式](/docs/styling/styles)：了解 Avalonia 的类 CSS 样式系统。
- [数据绑定语法](/docs/data-binding/data-binding-syntax)：Avalonia 绑定语法参考。
- [控件参考](/controls)：完整的 Avalonia 控件文档。
- [NavigationPage](/controls/navigation/navigationpage)：Avalonia 中基于栈的页面导航。
