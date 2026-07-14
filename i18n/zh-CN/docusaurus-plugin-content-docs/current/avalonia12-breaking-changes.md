---
id: avalonia12-breaking-changes
title: Avalonia 12 中的破坏性变更
description: Avalonia 11 与 12 之间破坏性变更的完整列表，并提供每项变更的迁移指南和代码示例。
doc-type: reference
sidebar_label: Breaking changes
toc_max_heading_level: 2
---

本页列出了 Avalonia 11 与 12 之间的所有破坏性变更，并提供迁移指导和可替代方案。


## .NET 支持已更新

Avalonia 12 已停止支持 .NET Framework 和 .NET Standard。现在仅支持 .NET 8 及更高版本，推荐目标框架为 .NET 10。

如果你的项目面向 Android 或 iOS，则只支持 .NET 10。这是为了与 Microsoft 对底层 .NET SDK 的支持策略保持一致。

请将 Avalonia 项目升级到受支持的 .NET 版本。

**示例：**

```diff
-<TargetFramework>netstandard2.0</TargetFramework>
+<TargetFramework>net10.0</TargetFramework>
```

PR: [#19869](https://github.com/AvaloniaUI/Avalonia/pull/19869)


## Avalonia 版本 12

Avalonia 的主版本号现已从 11 升级为 12。

请通过你常用的 IDE 或直接编辑项目文件，将所有 Avalonia 包引用升级到最新的 12.x 补丁版本。  
最新版本可在官方 [GitHub Releases 页面](https://github.com/AvaloniaUI/Avalonia/releases) 中查看。

**示例：**

```diff
-<PackageReference Include="Avalonia" Version="11.3.12" />
+<PackageReference Include="Avalonia" Version="12.0.0" />
-<PackageReference Include="Avalonia.Themes.Fluent" Version="11.3.12" />
+<PackageReference Include="Avalonia.Themes.Fluent" Version="12.0.0" />
```


## `Avalonia.Diagnostics` 包已移除

`Avalonia.Diagnostics` 包已被移除。
请改用 [Avalonia Plus](https://avaloniaui.net/pricing) 或更高版本中包含的 Dev Tools。

请从项目中移除 `Avalonia.Diagnostics` 包，并改用 `AvaloniaUI.DiagnosticsSupport`。  
如需安装 Avalonia Plus Dev Tools，请参阅 [Dev Tools 文档](/tools/developer-tools/installation)。

**示例：**

项目文件：

```diff
-<PackageReference Include="Avalonia.Diagnostics" Version="11.3.12" />
+<PackageReference Include="AvaloniaUI.DiagnosticsSupport" Version="2.2.0" />
```

应用程序文件：

```diff
-AttachDevTools();
+AttachDeveloperTools();
```

PR: [#20332](https://github.com/AvaloniaUI/Avalonia/pull/20332)


## 绑定类层级已变更

绑定类层级已经调整。定义在 XAML 文件中的绑定（例如 `{Binding}`）不受影响，但你需要更新 C# 代码中的绑定用法。

`IBinding` 接口已被移除，其替代类型为 [`BindingBase`](/api/avalonia/data/bindingbase) 类。

所有绑定类型现在都继承自 `BindingBase`：包括 [`ReflectionBinding`](/api/avalonia/data/reflectionbinding)、[`CompiledBinding`](/api/avalonia/data/compiledbinding)、`TemplateBinding` 和 `IndexerBinding`。请相应调整你的代码，不要再假设 `BindingBase` 实例只代表“标准”绑定。

`Binding` 类出于兼容性仍被保留，但它始终对应 `ReflectionBinding`。如果要在代码中创建绑定，请直接使用 `CompiledBinding` 和 `ReflectionBinding` 类。

`InstancedBinding` 类也已移除。它的直接对应类型是 `BindingExpressionBase`，表示应用到特定对象和属性上的绑定。

**示例：**
```diff
 public record Item(string Value);

-var reflectionBinding = new Binding("SomeProperty");
 var reflectionBinding = new ReflectionBinding(nameof(Item.Value));

+var compiledBinding = CompiledBinding.Create((Item item) => item.Value);
```

PR: [#19589](https://github.com/AvaloniaUI/Avalonia/pull/19589), [#20439](https://github.com/AvaloniaUI/Avalonia/pull/20439)


## 编译绑定默认启用

`<AvaloniaUseCompiledBindingsByDefault>` 现在默认值为 `true`。  
XAML 中的 `Binding` 用法现在会默认映射为 `CompiledBinding`。

Avalonia 官方模板在过去几年中创建的项目里都显式启用了该标志，因此较新的代码库通常不会受影响。不过，如果你的项目之前没有定义这个标志，那么在旧版本中它的默认值是 `false`。

只要条件允许，推荐优先使用编译绑定，因为它比反射绑定性能更好，并且会在构建时检查正确性。

PR: [#19712](https://github.com/AvaloniaUI/Avalonia/pull/19712)


## 绑定插件已移除

绑定插件原本是为绑定功能扩展预留的扩展点。但在实践中，它们无法与编译绑定良好协作，而且大多数人实际上并未使用它们。 

更糟的是，用户还经常需要关闭默认的数据注解验证插件，因为它会与 `CommunityToolkit.Mvvm` 等常用框架发生冲突。

从 Avalonia v12 开始，插件不再可配置。数据注解插件现在默认关闭。

PR: [#20623](https://github.com/AvaloniaUI/Avalonia/pull/20623)


## 可配置的文本 shaping 系统

文本 shaping 系统现在可以独立于渲染引擎进行配置。对于大多数应用来说，这项变更应该是透明的，因为在 `AppBuilder` 上调用 `UsePlatformDetect` 时，默认会使用 HarfBuzz。

不过，如果你的项目显式配置了渲染引擎，例如通过 `UseSkia` 使用 Skia，那么启动时可能会抛出 `InvalidOperationException`，提示 *No text shaping system configured*。此时你需要在 `AppBuilder` 上额外调用 `UseHarfBuzz`。

**示例：**

项目文件：

```diff
+<PackageReference Include="Avalonia.HarfBuzz" Version="12.0.0" />
```

应用程序文件：

```diff
 AppBuilder.Configure<App>()
     .UseSkia()
+    .UseHarfBuzz()
```

PR: [#19852](https://github.com/AvaloniaUI/Avalonia/pull/19852)


## 触控笔/触摸的焦点与选择行为已改进

[`SelectingItemsControl`](/api/avalonia/controls/primitives/selectingitemscontrol)（以及 `ListBox`、`ComboBox`、`TabControl` 等派生控件）和 [`TreeView`](/api/avalonia/controls/treeview) 的选择处理方式现已统一，以便在不同输入设备上提供一致且符合平台习惯的行为：

- **触摸和触控笔输入** 现在会在指针释放时而不是按下时触发选择，这与原生平台习惯一致。这样就可以在可选项上开始滑动或滚动手势，而不会误改当前选择。
- **容器类型**（例如 `ListBoxItem`、`TreeViewItem`）现在会直接处理选择输入，而不是让事件冒泡到其父级 `ItemsControl`。如果你的自定义逻辑依赖于在 `ItemsControl` 层级拦截选择事件，那么必须将其迁移到容器本身，或迁移到 `UpdateSelectionFromEvent` 的重写中。
- **焦点** 现在仅会在触摸和触控笔设备释放时发生变化。

### 已弃用的 API

`SelectingItemsControl` 上的以下方法已被标记为弃用，并将在未来版本中移除：

| 已弃用方法 | 替代方案 |
|---|---|
| `UpdateSelection` | `UpdateSelectionFromEvent` |
| `UpdateSelectionFromEventSource` | `UpdateSelectionFromEvent` |

### 自定义选择行为

你可以在 `SelectingItemsControl` 或 `TreeView` 上重写 `ShouldTriggerSelection`，以控制哪些指针或按键事件会启动选择。也可以重写 `UpdateSelectionFromEvent`，以自定义选择的应用方式。

静态 `ItemSelectionEventTriggers` 类提供了用于检查修饰键的辅助方法：

| 方法 | 说明 |
|---|---|
| `ShouldTriggerSelection(InputElement, PointerEventArgs)` | 判断某个指针事件是否应触发选择。 |
| `ShouldTriggerSelection(InputElement, KeyEventArgs)` | 判断某个按键事件是否应触发选择。 |
| `HasRangeSelectionModifier(InputElement, RoutedEventArgs)` | 检查是否按下 Shift 修饰键（范围选择）。 |
| `HasToggleSelectionModifier(InputElement, RoutedEventArgs)` | 检查是否按下 Ctrl 修饰键（切换选择）。 |

PR: [#19203](https://github.com/AvaloniaUI/Avalonia/pull/19203), [#19753](https://github.com/AvaloniaUI/Avalonia/pull/19753)


## [`TopLevel`](/api/avalonia/controls/toplevel) 变更

为了给高级窗口特性打基础，尤其是模板化绘制窗口装饰和虚拟窗口场景，Avalonia v12 对 `TopLevel` 做了多项调整：

- 尽管名称仍叫 `TopLevel`（包括 `Window`），但它不再一定处于可视树的根部。将顶部 `Visual` 强制转换为 `TopLevel` 的代码已不再可靠。
如需访问 `TopLevel` 实例，请始终调用 `TopLevel.GetTopLevel(Visual)`。
- 过去只由 `TopLevel` 类实现的接口，例如 `IInputRoot`、`IRenderRoot`、`ILayoutRoot`、`ITextInputMethodRoot` 和 `IEmbeddableLayoutRoot`，现在要么已被移除，要么不再可访问。请改用 `TopLevel`。
- 新增了 `IPresentationSource` 接口，它表示可视树的任意宿主（自身并不是 visual）。如果需要获取该实例，请调用新的 `GetPresentationSource(Visual)` 扩展方法。

PR: [#20624](https://github.com/AvaloniaUI/Avalonia/pull/20624)


## 窗口装饰变更

Avalonia v12 彻底调整了在不使用系统窗口边框时，窗口装饰（标题栏、标题按钮、调整大小抓手等）的绘制方式：

- 新增了 [`WindowDrawnDecorations`](/api/avalonia/controls/chrome/windowdrawndecorations) 类，它通过单个控件提供完整的窗口装饰。
- 有了这个新类型后，`TitleBar`、`CaptionButtons` 和 `ChromeOverlayLayer` 类型已经不再需要，因此被移除。
- `Window.ExtendClientAreaChromeHints` 属性由多个行为并不总是符合预期的标志组成，因此现已移除。请改用 [`WindowDecorations`](/api/avalonia/controls/windowdecorations) 属性，并结合 `ExtendClientAreaToDecorationsHint` 使用。

PR: [#20770](https://github.com/AvaloniaUI/Avalonia/pull/20770), [#20732](https://github.com/AvaloniaUI/Avalonia/pull/20732), [#20796](https://github.com/AvaloniaUI/Avalonia/pull/20796)


## 剪贴板变更

在较早的版本中，剪贴板支持已经重写为基于新的 [`IAsyncDataTransfer`](/api/avalonia/input/iasyncdatatransfer) 接口及相关类型。为了避免立即破坏兼容性，旧的 `IDataObject` 接口曾作为新系统的兼容层被保留并标记为弃用。

在 Avalonia v12 中，`IDataObject` 接口以及所有接受该类型的方法都已被移除。它的实现 `DataObject` 也不再执行任何实际功能。

`IClipboard` 接口已经简化，读取特定格式的方法现在以扩展方法形式提供（例如 `TryGetTextAsync` 和 `TryGetFile`）。

有关如何使用 `IAsyncDataTransfer`，请参阅官方的 [剪贴板文档](/docs/services/clipboard)。

**示例：**
```diff
// 将数据对象写入剪贴板
-var data = new DataObject();
-data.Set(DataFormats.Text, "some text");
+var item = new DataTransferItem();
+item.Set(DataFormat.Text, "some text");
+var data = new DataTransfer();
+data.Add(item);

-await clipboard.SetDataObjectAsync(data);
+await clipboard.SetDataAsync(data);
```

**示例：**
```diff
// 从剪贴板读取文本
-var text = await clipboard.GetTextAsync();
+var text = await clipboard.TryGetTextAsync();
```

PR: [#20521](https://github.com/AvaloniaUI/Avalonia/pull/20521)


## 支持多个 Dispatcher

严格来说这不算破坏性变更，但 Avalonia v12 现在支持多个 Dispatcher，每个线程一个。

对应用程序来说，继续使用 `Dispatcher.UIThread` 依然完全可行。不过，对库和控件作者而言，应开始使用 `AvaloniaObject.Dispatcher` 和 `Dispatcher.CurrentDispatcher`，以正确支持多 Dispatcher 场景。

`DispatcherTimer` 和 `AvaloniaSynchronizationContext` 现在默认使用当前 Dispatcher，而不是像以前那样默认使用 UI 线程。请确保这些类型是在正确的线程中实例化，或者将目标 Dispatcher 显式传入对应构造函数。

目前仍然不支持多个 UI 线程。

PR: [#18686](https://github.com/AvaloniaUI/Avalonia/pull/18686)


## `Dispatcher.InvokeAsync` 会捕获执行上下文

这项变更会让大多数异步调用的行为更符合开发者预期。`AsyncLocal` 将按预期工作，模拟身份和区域性上下文也会从调用方流转过来。  

PR: [#19163](https://github.com/AvaloniaUI/Avalonia/pull/19163)


## `FuncMultiValueConverter` 接受 `IReadOnlyList`

转换器函数的参数类型已从 `IEnumerable<TIn>` 改为 `IReadOnlyList<TIn>`，这样你就可以直接按索引访问值（例如 `values[0]`），而无需先转换成中间集合。由于 `IReadOnlyList<T>` 本身继承自 `IEnumerable<T>`，因此大多数只是遍历或使用 LINQ 的现有代码不会受影响。只有显式将 lambda 参数类型声明为 `IEnumerable<TIn>` 的代码需要更新。

PR: [#19936](https://github.com/AvaloniaUI/Avalonia/pull/19936)


## `Window.WindowState` 现在是直接属性

`Window.WindowState` 过去是一个样式属性。但由于它在窗口状态管理中引发了多种问题，因此现已改为直接属性。

因此，现在不能再通过样式来设置 `WindowState`。


## 自定义控件默认启用数据验证

在 v12 之前，对于启用了 `enableDataValidation: true` 的 Avalonia 属性，必须重写 `UpdateDataValidation` 方法，数据验证错误才会报告到该属性。现在这一过程会自动完成。

如果你的 `UpdateDataValidation` 重写仅仅是调用 `DataValidationErrors.SetError`，那么应将其删除。

**示例：**
```diff
 public class CustomControl
 {

     public static readonly StyledProperty<string?> CustomProperty = 
         AvaloniaProperty.Register<CustomControl, string?>("Custom", enableDataValidation: true)); 

-    protected override void UpdateDataValidation(AvaloniaProperty property, BindingValueType state, Exception? error)
-    {
-        if (property == CustomProperty)
-        {
-            DataValidationErrors.SetError(this, error);
-        }
-    } 
 }
```

如果你确实需要为某个属性恢复旧行为，可以继续重写 `UpdateDataValidation`，但不要对该属性调用基类方法。

PR: [#20067](https://github.com/AvaloniaUI/Avalonia/pull/20067)


## 渲染目标与平台表面接口已重构

多个内部渲染接口已经重组：

- `IRenderTarget.CreateDrawingContext` 现在接收一个 `RenderTargetSceneInfo` 参数，而不是多个重载。
- `IRenderTargetBitmapImpl` 不再继承 `IRenderTarget`，而是继承 `IReadableBitmapImpl`，并提供更简单的 `CreateDrawingContext()` 方法。
- `IDrawingContextLayerImpl` 不再继承 `IRenderTargetBitmapImpl`，而是直接继承 `IBitmapImpl`。
- 平台表面现在使用强类型的 `IPlatformRenderSurface` 接口，而不是 `IEnumerable<object>`。
- `ISkiaGpu` 现已转为内部类型。

多个带版本区分或被拆分的接口已合并回其基础类型：

- `IRenderTarget2` 和 `IRenderTargetWithProperties` 已合并到 `IRenderTarget`。
- `IGlPlatformSurfaceRenderTarget2` 和 `IGlPlatformSurfaceRenderTargetWithCorruptionInfo` 已合并到 `IGlPlatformSurfaceRenderTarget`。
- `ISkiaGpuRenderTarget2` 已合并到 `ISkiaGpuRenderTarget`。
- `ISkiaGpuWithPlatformGraphicsContext` 已合并到 `ISkiaGpu`。

此外，alpha 格式的处理也已统一：

- `ILockedFramebuffer` 现在包含 [`AlphaFormat`](/api/avalonia/platform/alphaformat) 属性。
- `IReadableBitmapImpl` 现在包含 `AlphaFormat?` 属性，替代了单独存在的 `IReadableBitmapWithAlphaImpl` 接口。
- `Bitmap.CopyPixels()` 不再接收 `AlphaFormat` 参数，alpha 格式现在会直接从 `ILockedFramebuffer` 读取。
- `LockedFramebuffer` 构造函数现在要求传入 `AlphaFormat` 参数。

`IPlatformRenderInterfaceContext.CreateOffscreenRenderTarget` 方法签名已从 `(PixelSize, double)` 改为 `(PixelSize, Vector, bool)`，其中原本的 `double` 缩放因子被替换为表示各轴缩放的 `Vector`，并新增一个用于控制文本抗锯齿的 `bool` 参数。接口中还新增了 `MaxOffscreenRenderTargetPixelSize` 属性。

这些变更只会影响实现自定义渲染后端或直接使用平台级渲染接口的代码。使用 `RenderTargetBitmap` 或标准绘图 API 的应用程序代码不会受影响。

PRs: [#20811](https://github.com/AvaloniaUI/Avalonia/pull/20811), [#20556](https://github.com/AvaloniaUI/Avalonia/pull/20556), [#20557](https://github.com/AvaloniaUI/Avalonia/pull/20557), [#20497](https://github.com/AvaloniaUI/Avalonia/pull/20497)

## 文本格式化构造函数已调整

`GenericTextRunProperties`、`TextCollapsingProperties` 和 `TextShaperOptions` 过去都各自提供两个构造函数：一个带 `FontFeatureCollection` 参数，一个不带。

现在它们已经合并为一个带可选参数的构造函数。
根据你原先使用的重载不同，可能需要调整参数顺序，因为 `FontFeatureCollection` 现在位于最后一个参数位置。

PR: [#20527](https://github.com/AvaloniaUI/Avalonia/pull/20527)


## 访问键现在按符号触发

过去访问键是由底层虚拟键触发的，因此带重音字符或数字无法作为访问键使用。

为了与其他主流 UI 框架及用户预期保持一致，访问键现在由按键上实际印刷的符号触发。这意味着带重音字符（例如 `_é`）和数字（例如 `_2`）现在都可以作为访问键使用。

`AccessText.AccessKey` 属性类型已从 `char` 改为 `string?`，以支持多字节 Unicode 字符。读取该属性的代码需要相应更新。

PR: [#20662](https://github.com/AvaloniaUI/Avalonia/pull/20662)


## 字体支持已更新

为了确保跨平台字体加载行为一致，Avalonia v12 现在内置了自己的字体解析器。

某些既非 TrueType（.ttf）也非 OpenType（.otf）的旧字体格式已不再受支持，其中包括古老的 Type 1 字体格式（.pfb/.pfm）。

PR: [#19852](https://github.com/AvaloniaUI/Avalonia/pull/19852)


## [`Screen`](/api/avalonia/platform/screen) 现为抽象类

`Screen` 类存在多个内部实现，而其基类之所以可被实例化，仅仅是出于历史兼容原因。

在 Avalonia v12 中，它现在已经变为抽象类。请不要自行构造 `Screen`，而是通过其成员获取现有实例（例如 `Screens.All`、`Screens.Primary`、`Screens.ScreenFromWindow`）。

PR: [#20529](https://github.com/AvaloniaUI/Avalonia/pull/20529)


## [`ResourcesChangedEventArgs`](/api/avalonia/controls/resourceschangedeventargs) 现为结构体

出于性能考虑，`ResourcesChangedEventArgs` 现在改为了结构体。

大多数项目其实不会主动构造这个类型的实例，因为它主要通过 `StyledElement.ResourcesChanged` 事件使用。
如果你过去确实手动构造过 `ResourcesChangedEventArgs`，现在请改用 `ResourceChangedEventArgs.Create`。

PR: [#20576](https://github.com/AvaloniaUI/Avalonia/pull/20576)


## 手势事件已迁移

此前声明在 `Gestures` 类中的所有附加事件（例如 `ScrollGesture`、`Pinch` 等）现在都迁移到了 `InputElement` 上，因此默认可用于所有元素。请从你的 XAML 中移除 `Gestures.` 前缀。

`Gestures` 类现在也不再对外公开。

示例：

```diff
-<Button Gestures.Pinch="Button_Pinch" />
+<Button Pinch="Button_Pinch" />
```

PR: [#20789](https://github.com/AvaloniaUI/Avalonia/pull/20789)


## 焦点处理已改进

`InputElement.GotFocus` 和 `InputElement.LostFocus` 事件参数类型现在已改为新的 `FocusChangedEventArgs` 类。该类型会提供关于先前焦点元素和当前焦点元素的更多信息。

请相应更新你的事件处理器。

示例：
```diff
-private void TextBox_GotFocus(object? sender, GotFocusEventArgs e)
+private void TextBox_GotFocus(object? sender, FocusChangedEventArgs e)

-private void TextBox_LostFocus(object? sender, RoutedEventArgs e)
+private void TextBox_LostFocus(object? sender, FocusChangedEventArgs e)
```

所有焦点处理现在统一由 `FocusManager` 类和 `IFocusManager` 接口管理。对于 `KeyboardNavigationHandler.GetNext` 的调用，需要替换为 `FocusManager.GetNextElement`。

PR: [#20859](https://github.com/AvaloniaUI/Avalonia/pull/20859), [#18647](https://github.com/AvaloniaUI/Avalonia/pull/18647), [#20930](https://github.com/AvaloniaUI/Avalonia/pull/20930)


## 不可见控件上的动画现在会停止

为了提高效率，由样式触发的动画在对应控件隐藏后将不再继续运行，这在多种场景下都能降低 CPU 占用。

如果确实需要恢复以前的行为，可以将新的 `Animation.PlaybackBehavior` 设置为 `Always`。

PR: [#20820](https://github.com/AvaloniaUI/Avalonia/pull/20820)


## Windows

### Direct2D1 支持已移除

Avalonia 不再提供 Direct2D 渲染后端。该包长期缺乏维护，且在功能和性能上都无法与 Skia 后端保持一致。Skia 现在是所有场景下推荐使用的渲染后端。

**示例：**

项目文件：

```diff
-<PackageReference Include="Avalonia.Direct2D1" Version="11.3.12" />
+<PackageReference Include="Avalonia.Skia" Version="12.0.0" />
```

应用程序文件：

```diff
 AppBuilder.Configure<App>()
-    .UseDirect2D1()
+    .UseSkia()
```

PR: [#20132](https://github.com/AvaloniaUI/Avalonia/pull/20132)

### `BinaryFormatter` 已移除

在以前的版本中，Avalonia 在 Windows 上使用 .NET 的 `BinaryFormatter`，隐式序列化和反序列化放入剪贴板的任意对象。

Microsoft 已经在多个 .NET 版本中持续建议 [迁移 away from the binary formatter](https://learn.microsoft.com/en-us/dotnet/standard/serialization/binaryformatter-migration-guide/)。Avalonia 也遵循了这一建议，不再继续使用它。

你的项目应当使用自己选择的机制显式进行对象的序列化与反序列化，JSON 是一种常见选择。

PR: [#20455](https://github.com/AvaloniaUI/Avalonia/pull/20455)

### `Window.ExtendClientAreaToDecorationsHint` 已改进

在 Windows 上，`ExtendClientAreaToDecorationsHint` 属性的多个问题已经修复，现在它可以在所有受支持场景中正常工作。过去曾建议使用各种变通方案，例如在受影响的窗口中添加或移除边距（尤其是窗口最大化时）。这些旧的变通方案现在都应当删除。

PR: [#20217](https://github.com/AvaloniaUI/Avalonia/pull/20217)


## Android

### 应用初始化方式已变更

在 Avalonia 12 中，Android 应用及其 Activity 的初始化方式已经调整，从而使后续创建的 Activity 能够正确使用你项目中定义的 `Application` 类。

1. 将主 Activity 的基类从泛型 `AvaloniaMainActivity<TApp>` 改为非泛型 [`AvaloniaMainActivity`](/api/avalonia/android/avaloniamainactivity)。
2. 新增一个继承自 `AvaloniaAndroidApplication<TApp>` 的类，并为其添加 `[Android.App.Application]` 特性。

**示例：**

```diff
[Activity]
public class MainActivity :
-AvaloniaMainActivity<App>
+AvaloniaMainActivity
{
}

+[Application]
+public class AndroidApp : AvaloniaAndroidApplication<App>
+{
+    protected AndroidApp(IntPtr javaReference, JniHandleOwnership transfer)
+        : base(javaReference, transfer)
+    {
+    }
+}
```

PR: [#18756](https://github.com/AvaloniaUI/Avalonia/pull/18756)

### [`IActivityApplicationLifetime`](/api/avalonia/controls/applicationlifetimes/iactivityapplicationlifetime) 取代 [`ISingleViewApplicationLifetime`](/api/avalonia/controls/applicationlifetimes/isingleviewapplicationlifetime)

Android 现在使用 `IActivityApplicationLifetime`，不再使用 `ISingleViewApplicationLifetime`。这个新接口提供的是 `MainViewFactory` 属性（类型为 `Func<Control>`），而不是单一的 `MainView` 实例，因为 Android 在应用生命周期内可能会创建多个 Activity 实例。

请更新你的 `App.axaml.cs`，优先检查 `IActivityApplicationLifetime`，再检查 `ISingleViewApplicationLifetime`：

```diff
 public override void OnFrameworkInitializationCompleted()
 {
     if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktop)
         desktop.MainWindow = new MainWindow();
+    else if (ApplicationLifetime is IActivityApplicationLifetime activityLifetime)
+        activityLifetime.MainViewFactory = () => new MainView();
     else if (ApplicationLifetime is ISingleViewApplicationLifetime singleView)
         singleView.MainView = new MainView();
     base.OnFrameworkInitializationCompleted();
 }
```

对 iOS、浏览器和嵌入式 Linux 平台来说，仍然需要保留 `ISingleViewApplicationLifetime` 的判断。

PR: [#18893](https://github.com/AvaloniaUI/Avalonia/pull/18893)

### `AvaloniaMainActivity` 中的 `CreateAppBuilder` 与 `CustomizeAppBuilder` 已移除

`CreateAppBuilder()` 和 `CustomizeAppBuilder(AppBuilder)` 这两个虚方法已从 `AvaloniaMainActivity` 中移除。它们此前已经被标记为弃用，框架也不再调用它们。应用初始化现在完全通过 `AvaloniaAndroidApplication<TApp>` 处理，详见上文。

如果你之前重写了这些方法，请将相关逻辑迁移到 `AvaloniaAndroidApplication<TApp>` 子类或 `App` 类中。

PR: [#20715](https://github.com/AvaloniaUI/Avalonia/pull/20715)


## iOS

### 应用初始化方式已变更

在 Avalonia 12 中，iOS 应用的初始化方式已改为采用 [“scenes” 概念](https://developer.apple.com/documentation/technotes/tn3187-migrating-to-the-uikit-scene-based-life-cycle)。Apple 预计会在不久的将来强制要求这一模式。

虽然大多数应用无需修改代码就能继续运行，但 `AvaloniaAppDelegate.Window` 在应用初始化后现在会保持为 `null`，因为窗口已改由 scene delegate 在内部管理。

如果你需要访问 `UIWindow`，请重写 `AvaloniaView.MovedToWindow` 方法，以便在视图附加到窗口时进行检测。

PR: [#20454](https://github.com/AvaloniaUI/Avalonia/pull/20454)


## Browser

### `Avalonia.Browser.Blazor` 包已移除

`Avalonia.Browser.Blazor` 包原本是 Avalonia 从旧的基于 Blazor 的浏览器后端迁移过程中的临时过渡方案。如今 Avalonia 已完全迁移到基于 WASM 的新后端（`Avalonia.Browser`），该包也因不再维护而被移除。

如果你仍想在 Blazor 之上运行 Avalonia，可以使用受支持的 `Avalonia.Browser` 包及其 `AvaloniaView` 类。

PR: [#20105](https://github.com/AvaloniaUI/Avalonia/pull/20105)


## Tizen

### `Avalonia.Tizen` 包已移除

由于缺少维护者，Tizen 平台已不再提供开箱即用支持。
详情请参阅 [Moving Tizen Support Out of Main Repository](https://github.com/AvaloniaUI/Avalonia/discussions/19721)。

PR: [#19722](https://github.com/AvaloniaUI/Avalonia/pull/19722)


## Headless

### xUnit.net 与 NUnit 的支持版本已更新

Avalonia 无头平台所支持的底层测试框架已经升级到它们的最新版本，因此单元测试代码可能也需要调整。
- xUnit.net 现在支持版本 3（原为版本 2）。如何迁移单元测试，请参阅 xUnit.net 的[官方迁移指南](https://xunit.net/docs/getting-started/v3/migration)。
- NUnit 现在支持版本 4（原为版本 3）。如何迁移单元测试，请参阅 NUnit 的[官方迁移指南](https://docs.nunit.org/articles/nunit/release-notes/Nunit4.0-MigrationGuide.html)。

PR: [#20372](https://github.com/AvaloniaUI/Avalonia/pull/20372), [#20481](https://github.com/AvaloniaUI/Avalonia/pull/20481)


## 已移除的成员

以下成员已在 Avalonia 12 中移除。它们按触发移除的功能领域进行分组。

### 绑定类层级

上下文请参阅 [绑定类层级已变更](#binding-class-hierarchy-changes)。

| 已移除成员 | 替代方案 | PR |
|---|---|---|
| `IBinding` interface | `BindingBase` | [#19589](https://github.com/AvaloniaUI/Avalonia/pull/19589) |
| `InstancedBinding` class | `BindingExpressionBase` | [#19589](https://github.com/AvaloniaUI/Avalonia/pull/19589) |

### 绑定插件

上下文请参阅 [绑定插件已移除](#binding-plugins-removed)。请移除这些类型的所有用法。

| 已移除成员 | PR |
|---|---|
| `Avalonia.Data.Core.Plugins.BindingPlugins` | [#20623](https://github.com/AvaloniaUI/Avalonia/pull/20623) |
| `Avalonia.Data.Core.Plugins.DataValidationBase` | [#20623](https://github.com/AvaloniaUI/Avalonia/pull/20623) |
| `Avalonia.Data.Core.Plugins.ExceptionValidationPlugin` | [#20623](https://github.com/AvaloniaUI/Avalonia/pull/20623) |
| `Avalonia.Data.Core.Plugins.IDataValidationPlugin` | [#20623](https://github.com/AvaloniaUI/Avalonia/pull/20623) |
| `Avalonia.Data.Core.Plugins.IndeiValidationPlugin` | [#20623](https://github.com/AvaloniaUI/Avalonia/pull/20623) |
| `Avalonia.Data.Core.Plugins.IPropertyAccessorPlugin` | [#20623](https://github.com/AvaloniaUI/Avalonia/pull/20623) |
| `Avalonia.Data.Core.Plugins.IStreamPlugin` | [#20623](https://github.com/AvaloniaUI/Avalonia/pull/20623) |
| `Avalonia.Data.Core.Plugins.PropertyAccessorBase` | [#20623](https://github.com/AvaloniaUI/Avalonia/pull/20623) |
| `Avalonia.Data.Core.Plugins.PropertyError` | [#20623](https://github.com/AvaloniaUI/Avalonia/pull/20623) |

### 剪贴板与拖放

上下文请参阅 [剪贴板变更](#clipboard-changes)。

| 已移除成员 | 替代方案 | PR |
|---|---|---|
| `DataFormats.*` members | `DataFormat.*` | [#20521](https://github.com/AvaloniaUI/Avalonia/pull/20521) |
| `DataObject.*` members | `DataTransfer` | [#20521](https://github.com/AvaloniaUI/Avalonia/pull/20521) |
| `DataObjectExtensions` class | `AsyncDataTransferExtensions` | [#20521](https://github.com/AvaloniaUI/Avalonia/pull/20521) |
| `DragDrop.DoDragDrop` method | `DragDrop.DoDragDropAsync` | [#20521](https://github.com/AvaloniaUI/Avalonia/pull/20521) |
| `DragEventArgs.Data` property | `DragEventArgs.DataTransfer` | [#20521](https://github.com/AvaloniaUI/Avalonia/pull/20521) |
| `IDataObject` interface | `IAsyncDataTransfer` | [#20521](https://github.com/AvaloniaUI/Avalonia/pull/20521) |
| `IClipboard.GetDataAsync` | `IClipboard.TryGetDataAsync` | [#20521](https://github.com/AvaloniaUI/Avalonia/pull/20521) |
| `IClipboard.GetFormatsAsync` | `ClipboardExtensions.GetDataFormatsAsync` | [#20521](https://github.com/AvaloniaUI/Avalonia/pull/20521) |
| `IClipboard.GetTextAsync` | `ClipboardExtensions.TryGetTextAsync` | [#20521](https://github.com/AvaloniaUI/Avalonia/pull/20521) |
| `IClipboard.SetTextAsync` | `ClipboardExtensions.SetTextAsync` | [#20521](https://github.com/AvaloniaUI/Avalonia/pull/20521) |
| `IClipboard.TryGetInProcessDataObjectAsync` | `IClipboard.TryGetInProcessDataAsync` | [#20521](https://github.com/AvaloniaUI/Avalonia/pull/20521) |

### TopLevel

上下文请参阅 [`TopLevel` 变更](#toplevel-changes)。

| 已移除成员 | 替代方案 | PR |
|---|---|---|
| `IInputRoot` interface | `TopLevel` | [#20624](https://github.com/AvaloniaUI/Avalonia/pull/20624) |
| `ITextInputMethodRoot` interface | `TopLevel` | [#20624](https://github.com/AvaloniaUI/Avalonia/pull/20624) |
| `IEmbeddedLayoutRoot` interface | `TopLevel` | [#20624](https://github.com/AvaloniaUI/Avalonia/pull/20624) |
| `ILayoutRoot` interface | `TopLevel` | [#20624](https://github.com/AvaloniaUI/Avalonia/pull/20624) |
| `IRenderRoot` interface | `TopLevel` | [#20624](https://github.com/AvaloniaUI/Avalonia/pull/20624) |
| `LayoutManager` class | `ILayoutManager` | [#20624](https://github.com/AvaloniaUI/Avalonia/pull/20624) |
| `TopLevel.PlatformSettings` property | `VisualExtensions.GetPlatformSettings` | [#20624](https://github.com/AvaloniaUI/Avalonia/pull/20624) |
| `TopLevel.PointerOverElement` property | 移除相关用法 | [#20624](https://github.com/AvaloniaUI/Avalonia/pull/20624) |
| `TopLevel.StartRendering/StopRendering` | `EmbeddableControlRoot.StartRendering/StopRendering` | [#20624](https://github.com/AvaloniaUI/Avalonia/pull/20624) |
| `VisualExtensions.GetVisualRoot` method | `GetPresentationSource` + `IPresentationSource.RootVisual` | [#20624](https://github.com/AvaloniaUI/Avalonia/pull/20624) |

### 窗口装饰

上下文请参阅 [窗口装饰变更](#window-decoration-changes)。

| 已移除成员 | 替代方案 | PR |
|---|---|---|
| `Chrome.CaptionButtons` class | `WindowDrawnDecorations` | [#20770](https://github.com/AvaloniaUI/Avalonia/pull/20770) |
| `Chrome.TitleBar` class | `WindowDrawnDecorations` | [#20770](https://github.com/AvaloniaUI/Avalonia/pull/20770) |
| `ChromeOverlayLayer` class | `WindowDrawnDecorations` | [#20732](https://github.com/AvaloniaUI/Avalonia/pull/20732) |
| `VisualLayerManager.AdornerLayer` property | `AdornerLayer.GetAdornerLayer` | [#20732](https://github.com/AvaloniaUI/Avalonia/pull/20732) |
| `VisualLayerManager.ChromeOverlayLayer` property | `WindowDrawnDecorations` | [#20732](https://github.com/AvaloniaUI/Avalonia/pull/20732) |
| `VisualLayerManager.LightDismissOverlayLayer` property | Remove usages | [#20732](https://github.com/AvaloniaUI/Avalonia/pull/20732) |
| `VisualLayerManager.OverlayLayer` property | `OverlayLayer.GetOverlayLayer` | [#20732](https://github.com/AvaloniaUI/Avalonia/pull/20732) |
| `Window.ExtendClientAreaChromeHints` property | `Window.WindowDecorations` | [#20770](https://github.com/AvaloniaUI/Avalonia/pull/20770) |
| `SystemDecorations` enum | `WindowDecorations` | [#20796](https://github.com/AvaloniaUI/Avalonia/pull/20796) |
| `ExtendClientAreaChromeHints` enum | `WindowDecorations` | [#20770](https://github.com/AvaloniaUI/Avalonia/pull/20770) |
| `IPopupHostProvider` interface | [`Popup`](/api/avalonia/controls/primitives/popup) | [#20732](https://github.com/AvaloniaUI/Avalonia/pull/20732) |
| `IPopupHost` interface | `Popup` | [#20597](https://github.com/AvaloniaUI/Avalonia/pull/20597) |
| `LightDismissOverlayLayer` class | `VisualLayerManager` | [#20732](https://github.com/AvaloniaUI/Avalonia/pull/20732) |
| `OverlayPopupHost.CreatePopupHost` method | `Popup` | [#20597](https://github.com/AvaloniaUI/Avalonia/pull/20597) |

### 手势事件

上下文请参阅 [手势事件已迁移](#gesture-events-moved)。

| 已移除成员 | 替代方案 | PR |
|---|---|---|
| `Gestures` class (all attached events) | Events on `InputElement` | [#20789](https://github.com/AvaloniaUI/Avalonia/pull/20789) |

### 焦点处理

上下文请参阅 [焦点处理已改进](#focus-improvements)。

| 已移除成员 | 替代方案 | PR |
|---|---|---|
| `GotFocusEventArgs` class | `FocusChangedEventArgs` | [#20859](https://github.com/AvaloniaUI/Avalonia/pull/20859) |
| `KeyboardNavigationHandler` class | `FocusManager` | [#18647](https://github.com/AvaloniaUI/Avalonia/pull/18647) |

### 其他特定移除项

| 已移除成员 | 替代方案 | 说明 | PR |
|---|---|---|---|
| `ResourcesChangedEventArgs.Empty` | `ResourcesChangedEventArgs.Create` | 参见 [`ResourcesChangedEventArgs` 现为结构体](#resourceschangedeventargs-is-a-struct) | [#20576](https://github.com/AvaloniaUI/Avalonia/pull/20576) |
| `TextInputMethodClient.ShowInputPanel` | `InputPaneActivationRequested` event | 直接显示输入面板在所有平台上都不正确 | [#20544](https://github.com/AvaloniaUI/Avalonia/pull/20544) |
| `NativeMenuBar.EnableMenuItemClickForwarding` | 移除相关用法 | 该属性没有实际作用 | [#20577](https://github.com/AvaloniaUI/Avalonia/pull/20577) |
| `NativeMenuItemToggleType` enum | `MenuItemToggleType` | 已与 `MenuItemToggleType` 合并 | [#20577](https://github.com/AvaloniaUI/Avalonia/pull/20577) |
| `IGeometryContext2` interface | `IGeometryContext` | `isStroked`/`isFilled` 现在是 `IGeometryContext` 方法上的可选参数 | [#20528](https://github.com/AvaloniaUI/Avalonia/pull/20528) |
| `IWindowImpl.GetWindowsZOrder` | `IWindowingPlatform.GetWindowsZOrder` | 参数类型已从 `Span<Window>` 改为 `ReadOnlySpan<IWindowImpl>` | [#20633](https://github.com/AvaloniaUI/Avalonia/pull/20633) |
| `AutoCompleteBox.BindingEvaluator` | 自行提供实现 | 这是一个被公开暴露的实现细节 | [#20596](https://github.com/AvaloniaUI/Avalonia/pull/20596) |
| `CharacterReader` struct | 自行提供实现 | 这是一个被公开暴露的实现细节 | [#19123](https://github.com/AvaloniaUI/Avalonia/pull/19123) |
| `StringTokenizer` struct | 自行提供实现 | 这是一个被公开暴露的实现细节 | [#20544](https://github.com/AvaloniaUI/Avalonia/pull/20544) |
| `Data.Core.PropertyPath` class | 移除相关用法 | 这是旧版本遗留且未被使用的类型 | [#19133](https://github.com/AvaloniaUI/Avalonia/pull/19133) |
| `Remote.RemoteServer` class | 移除相关用法 | 遗留类型，且工作不正确 | [#20767](https://github.com/AvaloniaUI/Avalonia/pull/20767) |
| `Remote.RemoteWidget` class | 移除相关用法 | 遗留类型，且工作不正确 | [#20767](https://github.com/AvaloniaUI/Avalonia/pull/20767) |

### 自 Avalonia 11 起已弃用

以下成员在 Avalonia 11 中已被标记为弃用，并已在当前版本中移除。

| 已移除成员 | 替代方案 | PR |
|---|---|---|
| `IInsetsManager.DisplayEdgeToEdge` | `IInsetsManager.DisplayEdgeToEdgePreference` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `CustomAnimatorBase` / `CustomAnimatorBase<T>` | `InterpolatingAnimator<T>` | [#20613](https://github.com/AvaloniaUI/Avalonia/pull/20613) |
| `CubicBezierEasing` | `SplineEasing` | [#20613](https://github.com/AvaloniaUI/Avalonia/pull/20613) |
| `AppBuilder.LifetimeOverride` | 使用任意预定义 lifetime | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `AvaloniaObjectExtensions.Bind` | `AvaloniaObject.Bind` | [#20613](https://github.com/AvaloniaUI/Avalonia/pull/20613) |
| `IActivatableApplicationLifetime` | `Application.Current.TryGetFeature<IActivatableLifetime>` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `ContextMenu.PlacementMode` | `ContextMenu.Placement` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `FileDialog` / `FileSystemDialog` / `SystemDialog` | `IStorageProvider` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `OpenFileDialog` | `IStorageProvider.OpenFilePickerAsync` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `OpenFolderDialog` | `IStorageProvider.OpenFolderPickerAsync` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `SaveFileDialog` | `IStorageProvider.SaveFilePickerAsync` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `ManagedFileDialogExtensions.ShowManagedAsync` | `IStorageProvider.OpenFilePickerAsync` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `ItemContainerGenerator.ContainerFromIndex` | `ItemsControl.ContainerFromIndex` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `ItemContainerGenerator.IndexFromContainer` | `ItemsControl.IndexFromContainer` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `TreeContainerIndex` | `TreeView` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `TreeItemContainerGenerator` | `ItemContainerGenerator` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `ItemsControl.ItemsControlFromItemContaner` | `ItemsControl.ItemsControlFromItemContainer` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `ToggleButton.Checked/Unchecked/Indeterminate` events | `ToggleButton.IsCheckedChanged` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `TabItem.SubscribeToOwnerProperties` | 移除相关用法 | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `BindingPriority.TemplatedParent` | `BindingPriority.Template` | [#20613](https://github.com/AvaloniaUI/Avalonia/pull/20613) |
| `CompiledBindingPathBuilder.SetRawSource` | `CompiledBinding.Source` | [#20613](https://github.com/AvaloniaUI/Avalonia/pull/20613) |
| `Color.ToUint32` | `Color.ToUInt32` | [#20613](https://github.com/AvaloniaUI/Avalonia/pull/20613) |
| `DrawingContext.PushPreTransform/PushPostTransform/PushTransformContainer` | `DrawingContext.PushTransform` | [#20613](https://github.com/AvaloniaUI/Avalonia/pull/20613) |
| `ImmutableRadialGradientBrush.Radius` | `RadiusX` and `RadiusY` | [#20613](https://github.com/AvaloniaUI/Avalonia/pull/20613) |
| `IRadialGradientBrush.Radius` | `RadiusX` and `RadiusY` | [#20613](https://github.com/AvaloniaUI/Avalonia/pull/20613) |
| `RadialGradientBrush.Radius` | `RadiusX` and `RadiusY` | [#20613](https://github.com/AvaloniaUI/Avalonia/pull/20613) |
| `IApplicationPlatformEvents` | `Application.Current.TryGetFeature<IActivatableLifetime>` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `Popup.PlacementMode` | `Popup.Placement` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `Screen.PixelDensity` | `Screen.Scaling` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `Screen.Primary` | `Screen.IsPrimary` | [#20617](https://github.com/AvaloniaUI/Avalonia/pull/20617) |
| `ICompositionGpuImportedObject.ImportCompeted` | `ImportCompleted` | [#20613](https://github.com/AvaloniaUI/Avalonia/pull/20613) |
| `IStyleable` interface | `StyledElement` | [#20613](https://github.com/AvaloniaUI/Avalonia/pull/20613) |


## 已重命名的成员

| 旧名称 | 新名称 | 说明 | PR |
|---|---|---|---|
| `PseudolassesExtensions` | `PseudoClassesExtensions` | 修正拼写错误。由于此类型通常在 XAML 中隐式使用，或作为 C# 扩展方法调用，因此大多数代码库不会受影响。 | [#18717](https://github.com/AvaloniaUI/Avalonia/pull/18717) |
| `X11PlatformOptions.ExterinalGLibMainLoopExceptionLogger` | `ExternalGLibMainLoopExceptionLogger` | 修正拼写错误。 | [#19128](https://github.com/AvaloniaUI/Avalonia/pull/19128) |
| `TextBox.Watermark` | `TextBox.PlaceholderText` | 旧属性仍保留为弃用状态。 | [#20303](https://github.com/AvaloniaUI/Avalonia/pull/20303) |
| `TextBox.UseFloatingWatermark` | `TextBox.UseFloatingPlaceholder` | 旧属性仍保留为弃用状态。 | [#20303](https://github.com/AvaloniaUI/Avalonia/pull/20303) |
| `Window.SystemDecorations` | `Window.WindowDecorations` | 旧属性仍保留为弃用状态。参见 [窗口装饰变更](#window-decoration-changes)。 | [#20796](https://github.com/AvaloniaUI/Avalonia/pull/20796) |
| `RenderOptions.TextRenderingMode` | `TextOptions.TextRenderingMode` | `TextOptions` 还包含 `TextHintingMode` 和 `BaselinePixelAlignment`。参见 [文本选项](/docs/graphics-animation/text-options)。 | [#20107](https://github.com/AvaloniaUI/Avalonia/pull/20107) |
| `TextBlock.LetterSpacing` | `TextElement.LetterSpacing` | 现在它是一个可继承的附加属性，可用于所有模板化控件。`TextBlock` 上的 XAML 用法依然源码兼容；但代码中若引用 `TextBlock.LetterSpacingProperty`，应改为 `TextElement.LetterSpacingProperty`。 | [#20141](https://github.com/AvaloniaUI/Avalonia/pull/20141) |

## 另请参阅

- [Avalonia 12 发布说明](https://github.com/AvaloniaUI/Avalonia/releases)
