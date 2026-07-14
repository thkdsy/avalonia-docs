---
id: cheat-sheet
title: WPF 到 Avalonia 速查表
description: 快速对照 WPF 概念、控件和 API 与其在 Avalonia 中的对应实现。
doc-type: migration
---

面向从 WPF 迁移到 Avalonia 的开发者的快速参考。每一项都展示了 WPF 概念及其在 Avalonia 中的对应实现。

## XAML 命名空间

| WPF | Avalonia |
|---|---|
| `xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"` | `xmlns="https://github.com/avaloniaui"` |
| `xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"` | 相同 |
| `xmlns:local="clr-namespace:MyApp"` | `xmlns:local="using:MyApp"`（推荐）或 `clr-namespace:` |

## 属性系统

| WPF | Avalonia | Notes |
|---|---|---|
| `DependencyProperty` | `StyledProperty` | 支持样式、动画和继承 |
| `DependencyProperty`（性能关键场景） | `DirectProperty` | 更快，但不支持样式和动画 |
| `DependencyProperty.Register()` | `AvaloniaProperty.Register<TOwner, TValue>()` | 泛型注册方式 |
| `DependencyProperty.RegisterAttached()` | `AvaloniaProperty.RegisterAttached<TValue>()` | 附加属性 |
| `PropertyMetadata` | `StyledPropertyMetadata<T>` | 类型安全的元数据 |
| `CoerceValueCallback` | 通过元数据提供的 `CoerceValueCallback` | 概念相同 |
| 元数据中的 `PropertyChanged` 回调 | `Register` 中的 `propertyChanged` 回调 | 直接传入 |

## 样式

| WPF | Avalonia | Notes |
|---|---|---|
| `<Style TargetType="Button">` | `<Style Selector="Button">` | CSS-like selectors |
| `Style.Triggers` | 伪类（`:pointerover`、`:pressed`） | Avalonia 中没有 triggers |
| `DataTrigger` | 绑定 + 伪类或转换器 | 见下文 |
| `EventTrigger` | Animations on pseudo-classes | |
| `VisualStateManager` | 伪类 | `:pointerover`、`:pressed`、`:disabled`、`:checked` |
| `BasedOn="{StaticResource ...}"` | 通常不需要；选择器可组合 | |
| `Style x:Key="..."` | Style classes: `<Style Selector="Button.primary">` | |
| `Style="{StaticResource ButtonStyle}"` | `Classes="primary"` | |
| `ControlTemplate.Triggers` | Pseudo-class selectors | |
| `TemplateBinding` | `TemplateBinding` | 概念相同（仅 OneWay） |
| `{RelativeSource TemplatedParent}` | `{TemplateBinding}` 或 `$parent[ControlType]` | |

### DataTrigger 的等价写法

WPF：
```xml
<DataTrigger Binding="{Binding IsActive}" Value="True">
    <Setter Property="Background" Value="Green" />
</DataTrigger>
```

Avalonia（使用转换器或自定义伪类）：
```xml
<Style Selector="Border.status">
    <Setter Property="Background" Value="Red" />
</Style>
<Style Selector="Border.status[(vm:MyViewModel.IsActive)]">
    <Setter Property="Background" Value="Green" />
</Style>
```

或者使用绑定转换器：
```xml
<Border Background="{Binding IsActive, Converter={StaticResource BoolToColorConverter}}" />
```

## 数据绑定

| WPF | Avalonia | Notes |
|---|---|---|
| `{Binding Path}` | `{Binding Path}` | 语法相同 |
| `{Binding Path, Mode=TwoWay}` | `{Binding Path, Mode=TwoWay}` | 相同 |
| `{Binding RelativeSource={RelativeSource Self}}` | `{Binding $self.Property}` | 简写语法 |
| `{Binding RelativeSource={RelativeSource AncestorType=Grid}}` | `{Binding $parent[Grid].Property}` | |
| `{Binding RelativeSource={RelativeSource TemplatedParent}}` | `{TemplateBinding Property}` | |
| `{Binding ElementName=myControl, Path=Text}` | `{Binding #myControl.Text}` | `#name` 语法 |
| `CompiledBinding` | `{CompiledBinding}` 或 `x:CompileBindings="True"` | 编译期校验 |
| `MultiBinding` | `MultiBinding` | 概念相同 |
| `IValueConverter` | `IValueConverter` | 接口相同 |
| `IMultiValueConverter` | `IMultiValueConverter` | 接口相同 |
| `FallbackValue` | `FallbackValue` | 相同 |
| `TargetNullValue` | `TargetNullValue` | 相同 |
| `StringFormat` | `StringFormat` | 相同 |
| `UpdateSourceTrigger` | 通常不需要；默认行为已较合理 | `TextBox` 会在文本变化时更新 |

## 控件

| WPF | Avalonia | 说明 |
|---|---|---|
| `Window` | `Window` | 相同 |
| `UserControl` | `UserControl` | 相同 |
| `Button` | `Button` | 相同 |
| `TextBlock` | `TextBlock` | 相同 |
| `TextBox` | `TextBox` | 相同 |
| `CheckBox` | `CheckBox` | 相同 |
| `RadioButton` | `RadioButton` | 相同 |
| `ComboBox` | `ComboBox` | 相同 |
| [`ListBox`](/api/avalonia/controls/listbox) | `ListBox` | 相同 |
| `ListView` | `ListBox` | 使用带 `ItemTemplate` 的 ListBox |
| `TreeView` | `TreeView` | 相同 |
| `DataGrid` | `DataGrid`（NuGet 包） | 需要单独安装 |
| `TabControl` | `TabControl` | 相同 |
| `Expander` | `Expander` | 相同 |
| `Slider` | `Slider` | 相同 |
| `ProgressBar` | `ProgressBar` | 相同 |
| `ToolTip` | `ToolTip` | 通过附加属性 `ToolTip.Tip` 使用 |
| `StatusBar` | 无直接等价控件 | 使用带样式的面板替代 |
| `Menu` | `Menu` | 相同 |
| `ContextMenu` | `ContextMenu` | 相同 |
| `Popup` | `Popup` | 相同 |
| `ScrollViewer` | `ScrollViewer` | 相同 |
| `Image` | `Image` | 相同 |
| [`Border`](/api/avalonia/controls/border) | `Border` | 相同；支持 `BoxShadow` |
| `Viewbox` | `Viewbox` | 相同 |
| `ContentControl` | `ContentControl` | 相同 |
| `ItemsControl` | `ItemsControl` | 相同 |
| `StackPanel` | `StackPanel` | 相同 |
| `Grid` | `Grid` | 相同；支持 `ColumnDefinitions="Auto,*"` 简写 |
| `DockPanel` | `DockPanel` | 相同 |
| `WrapPanel` | `WrapPanel` | 相同 |
| `Canvas` | `Canvas` | 相同 |
| `UniformGrid` | `UniformGrid` | 相同 |
| `GroupBox` | [`GroupBox`](/controls/layout/containers/groupbox) | 相同 |
| `RichTextBox` | 无内置等价控件 | 使用第三方编辑器 |

## 布局

| WPF | Avalonia | 说明 |
|---|---|---|
| `Grid.RowDefinitions="Auto,*"` | 相同简写 | 两者都支持内联语法 |
| `DockPanel.LastChildFill` | `DockPanel.LastChildFill` | 相同 |
| `HorizontalAlignment` | `HorizontalAlignment` | 相同 |
| `VerticalAlignment` | `VerticalAlignment` | 相同 |
| `Margin="10,5"` | `Margin="10,5"` | 相同 |
| `SharedSizeGroup` | `SharedSizeGroup` | 相同 |
| `Visibility`（`Visible`/`Collapsed`/`Hidden`） | `IsVisible`（`bool`） | `IsVisible="False"` 等价于 WPF 的 `Collapsed`（会从布局中移除）。若需要 WPF 的 `Hidden` 行为（不可见但仍占位），请改用 `Opacity="0"`。 |

## 资源

| WPF | Avalonia | 说明 |
|---|---|---|
| `StaticResource` | `StaticResource` | 相同（解析一次） |
| `DynamicResource` | `DynamicResource` | 相同（会跟踪变化） |
| `MergedDictionaries` | `MergedDictionaries` | 相同 |
| `ResourceDictionary` | `ResourceDictionary` | 相同 |
| `ThemeDictionaries` | `ResourceDictionary.ThemeDictionaries` | 浅色/深色变体 |

## 事件

| WPF | Avalonia | 说明 |
|---|---|---|
| [`RoutedEvent`](/api/avalonia/interactivity/routedevent)（Bubble） | `RoutedEvent`（Bubble） | 相同 |
| `RoutedEvent`（Tunnel） | `RoutedEvent`（Tunnel） | 相同 |
| `Preview*` 事件 | Tunnel 路由策略 | 使用 `AddHandler` 搭配 `RoutingStrategies.Tunnel` |
| `EventManager.RegisterRoutedEvent` | `RoutedEvent.Register<T, TArgs>` | 泛型注册方式 |
| `e.Handled = true` | `e.Handled = true` | 相同 |
| `AddHandler(event, handler, handledEventsToo)` | 签名相同 | 相同 |
| 类处理器 | `Event.AddClassHandler<T>()` | 概念相同 |

## 命令

| WPF | Avalonia | 说明 |
|---|---|---|
| `ICommand` | `ICommand` | 接口相同 |
| `RoutedCommand` | 无内置等价实现 | 使用 `ICommand` 实现 |
| `CommandBinding` | 无对应机制 | 直接绑定命令 |
| `InputBinding` / `KeyBinding` | `KeyBinding` | 概念相同 |
| `RelayCommand`（MVVM toolkit） | `RelayCommand`（CommunityToolkit.Mvvm） | 同一套库可直接使用 |

## 模板

| WPF | Avalonia | 说明 |
|---|---|---|
| [`DataTemplate`](/api/avalonia/markup/xaml/templates/datatemplate) | `DataTemplate` | 相同 |
| `HierarchicalDataTemplate` | `TreeDataTemplate` | 名称不同 |
| `ControlTemplate` | `ControlTemplate` | 相同 |
| `DataTemplateSelector` | 使用带 `DataType` 的 `DataTemplate` | 通过 `DataType` 匹配 |
| `ContentPresenter` | `ContentPresenter` | 相同 |
| `ItemsPresenter` | `ItemsPresenter` | 相同 |
| `PART_` 命名约定 | `PART_` 命名约定 | 相同 |

## 线程

| WPF | Avalonia | 说明 |
|---|---|---|
| `Dispatcher.Invoke()` | `Dispatcher.UIThread.InvokeAsync()` | 默认异步 |
| `Dispatcher.BeginInvoke()` | `Dispatcher.UIThread.Post()` | 即发即忘 |
| `Dispatcher.CurrentDispatcher` | `Dispatcher.CurrentDispatcher` | API 相同 |
| `Dispatcher.FromThread()` | `Dispatcher.FromThread()` | API 相同 |
| `DependencyObject.Dispatcher` | `AvaloniaObject.Dispatcher` | 每对象调度器 |
| `Dispatcher.CheckAccess()` | `Dispatcher.UIThread.CheckAccess()` | 相同 |
| `Dispatcher.Yield()` | `Dispatcher.Yield()` | API 相同 |
| `DispatcherPriority` | `DispatcherPriority` | 枚举相同 |

## 动画

| WPF | Avalonia | 说明 |
|---|---|---|
| `Storyboard` | `Animation` | API 不同 |
| `DoubleAnimation` | 关键帧动画 | 使用带 `Cue` 的 `KeyFrame` |
| `BeginStoryboard` | 在 `Style.Animations` 中声明动画 | 由伪类触发 |
| `EasingFunction` | `Easing` 属性 | 可用的缓动类型相同 |
| `Transitions`（UWP） | `Transitions` | 属性变化动画 |
| `CompositionTarget.Rendering` | `TopLevel.RequestAnimationFrame()` | UI 线程上的逐帧回调；渲染线程回调可参见 `CompositionCustomVisualHandler` |

## 窗口

| WPF | Avalonia | 说明 |
|---|---|---|
| `AllowsTransparency="True"` | `TransparencyLevelHint="Transparent"` | 若要实现[点击穿透行为](/docs/how-to/window-how-to#transparent-click-through-window)，请设置 `Background="{x:Null}"`，而不是 `Transparent` |
| `WindowStyle="None"` | `SystemDecorations="None"` | 移除标题栏和边框 |
| `ResizeMode` | `CanResize` | 使用布尔值而不是枚举 |

## 图形

| WPF | Avalonia | 说明 |
|---|---|---|
| `SolidColorBrush` | `SolidColorBrush` | 相同 |
| `LinearGradientBrush` | `LinearGradientBrush` | 相同 |
| `RadialGradientBrush` | `RadialGradientBrush` | 相同 |
| `ImageBrush` | `ImageBrush` | 相同 |
| [`VisualBrush`](/api/avalonia/media/visualbrush) | `VisualBrush` | 相同 |
| `DrawingBrush` | 不可用 | 使用 `VisualBrush` |
| `BitmapEffect` | `Effect` 属性 | 如 `BlurEffect`、`DropShadowEffect` |
| `DropShadowEffect` | `Border` 上的 `BoxShadow` | API 不同，更接近 CSS |
| `RenderTransform` | `RenderTransform` | 相同；也支持 CSS 简写 |
| `LayoutTransform` | `LayoutTransformControl` | 通过包装控件实现 |
| `Clip` | `Clip` | 相同 |
| `OpacityMask` | `OpacityMask` | 相同 |
| `Path` | `Path` | 相同；使用同样的迷你语言 |

## 平台服务

| WPF | Avalonia | 说明 |
|---|---|---|
| `SystemParameters.PrimaryScreenWidth` | `TopLevel.GetTopLevel(this).Screens.Primary.Bounds.Width` | 可通过任意 `TopLevel` 上的 [`Screens`](/api/avalonia/controls/screens) 访问 |
| `System.Windows.Forms.Screen.AllScreens` | `TopLevel.GetTopLevel(this).Screens.All` | 返回所有已连接显示器 |
| `System.Windows.Forms.Screen.PrimaryScreen.WorkingArea` | `TopLevel.GetTopLevel(this).Screens.Primary.WorkingArea` | 不包含任务栏或程序坞 |
| `PresentationSource.FromVisual().CompositionTarget.TransformToDevice` | `TopLevel.GetTopLevel(this).Screens.Primary.Scaling` | DPI 缩放因子 |

完整用法请参阅 [使用屏幕](/docs/app-development/window-management#working-with-screens)。

## 文件结构

| WPF | Avalonia |
|---|---|
| `.xaml` extension | `.axaml` extension |
| `App.xaml` | `App.axaml` |
| `MainWindow.xaml` | `MainWindow.axaml` |
| `.xaml.cs` code-behind | `.axaml.cs` code-behind |
| `.csproj` WPF SDK | `.csproj` Avalonia SDK |

## 常见陷阱

1. **没有 triggers**：Avalonia 使用伪类和类似 CSS 的选择器，而不是 WPF 的 triggers。参见 [伪类](/docs/styling/pseudoclasses)。

2. **样式使用选择器而非 TargetType**：样式通过类似 CSS 的选择器（`Button.primary:pointerover`）表达，而不是 `TargetType` + `Triggers`。

3. **使用 x:Name 而不是 Name**：在 Avalonia XAML 中优先使用 `x:Name`。虽然 `Name` 属性也存在，但 `x:Name` 才是标准写法。

4. **绑定路径语法不同**：使用 `#elementName.Property` 替代 `ElementName=elementName, Path=Property`，使用 `$parent[Type]` 替代 `RelativeSource AncestorType`。

5. **没有 RoutedCommand**：Avalonia 不提供 WPF 的 `RoutedCommand` 基础设施。请使用 `ICommand` 实现，推荐搭配 CommunityToolkit.Mvvm。

6. **DataGrid 是独立包**：需要从 NuGet 安装 `Avalonia.Controls.DataGrid`。

7. **使用 TreeDataTemplate 而不是 HierarchicalDataTemplate**：名称不同，但概念完全一致。

8. **布局变换方式不同**：请使用 `LayoutTransformControl` 包装器，而不是 `LayoutTransform` 属性。

9. **资源使用 avares://**：资源 URI 使用 `avares://AssemblyName/path`，而不是 `pack://application:,,,/`。

10. **默认绑定模式可能不同**：某些控件的默认绑定模式与 WPF 不同。如果绑定更新不符合预期，请查看控件文档。

## 另请参阅

- [从 WPF 迁移](/docs/migration/wpf)：更详细的迁移指南。
- [Avalonia 架构](/docs/fundamentals/architecture)：Avalonia 的内部工作原理。
- [样式选择器](/docs/styling/style-selectors)：类似 CSS 的选择器语法。
- [数据绑定语法](/docs/data-binding/data-binding-syntax)：Avalonia 的绑定语法。
