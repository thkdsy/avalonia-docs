---
id: performance
title: 性能优化
description: 通过虚拟化、布局效率、编译绑定和性能分析来优化 Avalonia 应用性能。
doc-type: how-to
---

本指南介绍 Avalonia 应用中常见的性能注意事项，以及保持 UI 响应流畅的实用技巧。

## UI 虚拟化

当显示大型集合时，虚拟化可以确保只创建并渲染可见项。Avalonia 的 `ListBox`、`TreeView`、`DataGrid` 和 `ItemsRepeater` 默认都支持虚拟化。

### 虚拟化如何工作

虚拟化面板不会为集合中的每一项都创建控件，而是只为当前可见项创建控件。当用户滚动时，移出屏幕的控件会被回收，并重新用于即将进入视口的新项。

### 确保虚拟化已启用

虚拟化需要一个受约束的高度。如果项控件位于 `StackPanel` 或其他会给予其无限高度的容器中，虚拟化就会失效：

```xml
<!-- 错误：StackPanel 提供无限高度，会导致虚拟化失效 -->
<StackPanel>
    <ListBox ItemsSource="{Binding LargeCollection}" />
</StackPanel>

<!-- 正确：使用 * 的 Grid 行会约束高度 -->
<Grid RowDefinitions="*">
    <ListBox ItemsSource="{Binding LargeCollection}" />
</Grid>

<!-- 正确：DockPanel 的填充区域会约束高度 -->
<DockPanel>
    <TextBlock DockPanel.Dock="Top" Text="Items" />
    <ListBox ItemsSource="{Binding LargeCollection}" />
</DockPanel>
```

### 用于自定义布局的 ItemsRepeater

`ItemsRepeater` 提供了一个更底层的虚拟化控件，适合自定义布局场景：

```xml
<ScrollViewer>
    <ItemsRepeater ItemsSource="{Binding Items}">
        <ItemsRepeater.Layout>
            <StackLayout Spacing="4" />
        </ItemsRepeater.Layout>
        <ItemsRepeater.ItemTemplate>
            <DataTemplate>
                <TextBlock Text="{Binding Name}" />
            </DataTemplate>
        </ItemsRepeater.ItemTemplate>
    </ItemsRepeater>
</ScrollViewer>
```

### 用缓冲系数实现平滑滚动

`VirtualizingStackPanel` 支持 `BufferFactor` 属性，它会在可见视口之外额外保留一部分已实例化的项。这样可以减少滚动过程中的回收频率，从而缓解因垃圾回收带来的卡顿，尤其是在移动设备上。

```xml
<ListBox ItemsSource="{Binding LargeCollection}">
    <ListBox.ItemsPanel>
        <ItemsPanelTemplate>
            <VirtualizingStackPanel BufferFactor="1" />
        </ItemsPanelTemplate>
    </ListBox.ItemsPanel>
</ListBox>
```

当 `BufferFactor` 为 `1` 时，会在可见区域上下各额外保留一个视口高度范围内的项。默认值是 `0`（不做缓冲）。更高的值会消耗更多内存，但对于复杂项模板通常能带来更平滑的滚动体验。

### 可变高度项

当所有项高度一致时，`VirtualizingStackPanel` 的效果最佳。如果项高度不一致，面板就必须根据已测量项来估算整个滚动范围，这可能导致滚动条跳动和额外的布局重算。如果你的项高度差异很大，可以考虑以下策略：

- **使用统一的估算高度。** 为所有项设置固定的 `Height` 或 `MinHeight`，这样虚拟化面板就能更准确地计算滚动范围。如果内容超过估算尺寸，可以让它裁剪或在内部滚动。
- **拍平层级数据。** 与其在虚拟化列表中嵌套展开器，不如将树结构拍平成一个带缩进层级的列表。这样虚拟化面板就能直接管理所有行。`TreeView` 内部就是这样做的。
- **限制已实例化项的数量。** 如果虚拟化不可行（例如带展开器的复杂属性网格），就应限制同时存在的控件数量。只加载当前可见区域，并在用户展开或滚动时按需创建更多项。

### 降低控件模板复杂度

像 [`TextBox`](/api/avalonia/controls/textbox) 这样的复杂控件，内部包含较深的视觉树（边框、滚动查看器、水印层等）。当你一次创建成千上万个此类控件时，模板实例化与测量过程就会主导启动耗时。

**显示时使用轻量控件，交互时再切换。** 平时用 `TextBlock`（视觉树更轻）显示值，只有在用户点击编辑时再替换成 `TextBox`：

```csharp
// 在 DataTemplate 的代码后置或自定义控件中
var display = new TextBlock { Text = field.Value };
display.PointerPressed += (s, e) =>
{
    var editor = new TextBox { Text = field.Value };
    editor.LostFocus += (s2, e2) =>
    {
        field.Value = editor.Text;
        parent.Children.Remove(editor);
        parent.Children.Add(display);
    };
    parent.Children.Remove(display);
    parent.Children.Add(editor);
};
```

**为重量级控件重新定义模板。** 如果你必须在各处都使用 `TextBox`，可以创建一个精简的控件主题，去掉不必要的视觉元素（如水印、清除按钮、滚动查看器），从而降低视觉树深度：

```xml
<ControlTheme x:Key="LightTextBox" TargetType="TextBox">
    <Setter Property="Template">
        <ControlTemplate>
            <Border Background="{TemplateBinding Background}"
                    BorderBrush="{TemplateBinding BorderBrush}"
                    BorderThickness="{TemplateBinding BorderThickness}">
                <TextPresenter Name="PART_TextPresenter"
                               Text="{TemplateBinding Text}"
                               CaretBrush="{TemplateBinding CaretBrush}" />
            </Border>
        </ControlTemplate>
    </Setter>
</ControlTheme>
```

把它应用到那些不需要完整功能集的控件上：

```xml
<TextBox Theme="{StaticResource LightTextBox}" Text="{Binding Value}" />
```

## 布局性能

### 避免过深嵌套

每多一层嵌套，就会增加一次测量和排列成本。应尽量让布局更扁平：

```xml
<!-- 避免：过深嵌套的布局 -->
<StackPanel>
    <Border>
        <StackPanel>
            <Border>
                <TextBlock Text="Hello" />
            </Border>
        </StackPanel>
    </Border>
</StackPanel>

<!-- 推荐：更扁平的布局 -->
<StackPanel>
    <TextBlock Text="Hello" Margin="8" />
</StackPanel>
```

### 用 Grid 替代嵌套的 StackPanel

一个带行列定义的 `Grid`，通常比多个层层嵌套的 `StackPanel` 更高效：

```xml
<!-- 用来替代嵌套 StackPanel -->
<Grid ColumnDefinitions="Auto,*" RowDefinitions="Auto,Auto,Auto" RowSpacing="4">
    <TextBlock Grid.Row="0" Grid.Column="0" Text="Name:" />
    <TextBox Grid.Row="0" Grid.Column="1" Text="{Binding Name}" />
    <TextBlock Grid.Row="1" Grid.Column="0" Text="Email:" />
    <TextBox Grid.Row="1" Grid.Column="1" Text="{Binding Email}" />
</Grid>
```

### 尽量减少 InvalidateArrange / InvalidateMeasure

会影响布局的属性变更（如 Width、Height、Margin、Padding）会触发布局重算。能批量修改时就尽量批量修改：

```csharp
// 一起设置多个属性；Avalonia 会在单次 dispatcher 操作中
// 自动合并布局处理。
myControl1.Width = 100;
myControl2.Height = 200;
```

## 渲染性能

### 使用 IsVisible 隐藏未使用控件

将 `IsVisible="False"` 设为假，会让控件彻底退出布局和渲染流程。布局系统会跳过该控件及其整棵子树的测量与排列，渲染器也不会再绘制它。因此，对于按条件显示的内容，`IsVisible` 是一种有效的减负手段：

```xml
<Panel>
    <StackPanel IsVisible="{Binding ShowDetails}">
        <!-- 复杂内容：只有可见时才会参与测量和渲染 -->
    </StackPanel>
</Panel>
```

如果你只是想把控件视觉上隐藏，但仍保留它占据的布局空间，请使用 `Opacity="0"`。`Opacity="0"` 的元素依然参与布局，并且仍可能接收输入。

### 谨慎使用 ClipToBounds

`ClipToBounds="True"` 会创建一个裁剪层。只有当子内容确实会超出控件边界时，才值得启用它。

### 降低命中测试成本

当发生指针事件时，Avalonia 会遍历视觉树并测试每个元素。在 `Canvas` 或 `Panel` 中存在成百上千个子元素时，这种线性遍历会在点击与事件响应之间引入明显延迟。对于不需要指针交互的元素，请设置 `IsHitTestVisible="False"`；而对于对象非常多的场景，可以考虑使用基于覆盖层的命中测试策略或自定义渲染。更多模式和代码示例请参阅 [Hit Testing: Performance with many elements](/docs/graphics-animation/hit-testing#performance-with-many-elements)。

透明元素同样会参与命中测试。如果某个控件不需要指针交互，请将 `IsHitTestVisible="False"`：

```xml
<Border Background="Transparent" IsHitTestVisible="False">
    <!-- 不应拦截点击的覆盖层 -->
</Border>
```

### 降低视觉复杂度

- 尽量减少 `BoxShadow` 效果的数量（每个阴影都会增加一次渲染开销）
- 避免大量半透明元素互相重叠
- 优先在父元素上设置 `Opacity`，而不是给每个子元素分别设置

### BitmapCache

对于那些渲染成本高、但变化不频繁的视觉内容，可以使用 `BitmapCache` 将其栅格化到位图表面。控件及其子元素会先被渲染到一张中间位图中，后续帧会重复使用这张位图，直到内容发生变化。

```xml
<Border BoxShadow="0 4 8 0 #40000000" CornerRadius="8">
    <Border.CacheMode>
        <BitmapCache RenderAtScale="1" />
    </Border.CacheMode>
    <!-- 复杂内容只渲染一次并缓存 -->
</Border>
```

`BitmapCache` 的属性如下：

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `RenderAtScale` | `double` | `1` | 缓存位图的分辨率倍数。大于 1 会提升质量（适合将被放大的内容），小于 1 会以降低质量为代价节省内存。设为 0 可禁用缓存。 |
| `SnapsToDevicePixels` | `bool` | `false` | 将缓存位图对齐到设备像素边界，以获得更清晰的文字和线条渲染。 |
| `EnableClearType` | `bool` | `false` | 在缓存表面中启用 ClearType 子像素文本渲染。否则缓存中的文字将使用灰度抗锯齿。 |

对于包含大量文本的缓存内容，同时启用 `SnapsToDevicePixels` 和 `EnableClearType` 通常能得到最佳效果：

```xml
<Border>
    <Border.CacheMode>
        <BitmapCache SnapsToDevicePixels="True" EnableClearType="True" />
    </Border.CacheMode>
    <TextBlock Text="Cached text with ClearType rendering" />
</Border>
```

### BitmapInterpolationMode

对于不需要高质量缩放的图片，可以使用较低的插值模式：

```xml
<Image Source="avares://MyApp/Assets/thumbnail.png"
       RenderOptions.BitmapInterpolationMode="LowQuality" />
```

### GPU 资源缓存大小

Avalonia 默认使用启用 GPU 加速的 Skia。Skia 会为纹理和其他 GPU 支持的表面维护一个 GPU 资源缓存。默认缓存上限大约是 28 MB。如果你的应用处理大图、图块集或大量缓存视觉对象，超出缓存限制的图片就可能在每一帧都重新上传到 GPU，进而造成卡顿。

你可以在启动时通过配置 `SkiaOptions` 来增大缓存：

```csharp
AppBuilder.Configure<App>()
    .UsePlatformDetect()
    .With(new SkiaOptions
    {
        MaxGpuResourceSizeBytes = 256 * 1024 * 1024 // 256 MB
    });
```

请根据目标硬件选择合适的值。大多数集成显卡至少拥有 2 GB 的共享内存，因此 256 MB 或 512 MB 对桌面应用通常是安全的。移动设备则可能需要更低的设置值。

## 数据绑定性能

### 使用编译绑定

编译绑定会在编译时解析属性路径，从而避免运行时反射：

```xml
<UserControl x:CompileBindings="True" x:DataType="vm:MainViewModel">
    <TextBlock Text="{Binding Name}" />
</UserControl>
```

也可以在 `.csproj` 中为整个项目启用：

```xml
<AvaloniaUseCompiledBindingsByDefault>true</AvaloniaUseCompiledBindingsByDefault>
```

### 避免不必要的绑定

对于永远不会变化的属性，使用静态值而不是绑定：

```xml
<!-- 对常量使用绑定是没有必要的 -->
<TextBlock Text="{Binding AppTitle}" />

<!-- 更好：使用静态资源或字面量 -->
<TextBlock Text="{StaticResource AppTitle}" />
<TextBlock Text="My Application" />
```

### 对静态数据使用 OneTime 绑定

如果某个值只设置一次、之后不会变化，就使用 `OneTime` 模式来避免持续的变更跟踪：

```xml
<TextBlock Text="{Binding Version, Mode=OneTime}" />
```

## 集合性能

### 小到中等规模列表使用 ObservableCollection

`ObservableCollection<T>` 能高效地将单个项目的添加和移除通知给 UI。

### 批量处理大更新

当你需要一次添加很多项时，可以考虑直接替换整个集合，而不是逐个添加：

```csharp
// 较慢：每次 Add 都会触发一次 UI 更新
foreach (var item in newItems)
    Items.Add(item);

// 更快：一次性替换整个集合
Items = new ObservableCollection<Item>(newItems);
OnPropertyChanged(nameof(Items));
```

### 增量加载

如果你必须在没有虚拟化的情况下创建大量控件（例如属性网格或检查器面板），一次性全部添加会在测量阶段阻塞 UI 线程。更好的做法是分批添加，并在每批之间向 dispatcher 让出执行权，这样 UI 仍能保持响应：

```csharp
private async Task LoadItemsIncrementally(IList<ItemViewModel> items, Panel container)
{
    const int batchSize = 50;

    for (int i = 0; i < items.Count; i += batchSize)
    {
        var batch = items.Skip(i).Take(batchSize);
        foreach (var item in batch)
        {
            container.Children.Add(CreateControl(item));
        }

        // 向 UI 线程让出执行权，让当前帧有机会渲染
        await Dispatcher.UIThread.Yield(DispatcherPriority.Background);
    }
}
```

批次大小应足够在第一次加载时填满可见区域。这样用户可以立即看到内容，而剩余项则逐步加载。

### 大型响应式集合可使用 DynamicData

对于需要频繁排序、过滤或进行复杂转换的集合，[DynamicData](https://github.com/reactivemarbles/DynamicData) 提供了优化过的响应式管线，能够尽量减少 UI 更新次数。

## 异步与线程

### 让 UI 线程保持空闲

把耗时计算移到后台线程：

```csharp
var data = await Task.Run(() => LoadLargeDataSet());
Items = new ObservableCollection<Item>(data);
```

### 对高频输入进行防抖

对于边输入边搜索这类场景，应对输入进行防抖，以避免每次按键都触发昂贵操作：

```csharp
this.WhenAnyValue(x => x.SearchText)
    .Throttle(TimeSpan.FromMilliseconds(300))
    .Subscribe(text => ApplyFilter(text));
```

### 对延后任务使用 DispatcherPriority.Background

把低优先级更新安排到 UI 线程空闲时执行：

```csharp
Dispatcher.UIThread.Post(() =>
{
    // 低优先级工作
    UpdateStatistics();
}, DispatcherPriority.Background);
```

## 性能分析

### Avalonia DevTools

在调试构建中按 **F12** 可打开 DevTools。**Performance** 选项卡会显示帧耗时信息。

### dotTrace 与 dotMemory

JetBrains 的性能分析工具可用于 Avalonia 应用。你可以用它们识别热点路径和内存泄漏。

### 诊断覆盖层

你可以在 `App.axaml.cs` 中启用 FPS 覆盖层：

```csharp
public override void OnFrameworkInitializationCompleted()
{
    // 在调试构建中添加 FPS 覆盖层
#if DEBUG
    this.AttachDevTools();
#endif
}
```

## 另请参阅

- [Threading Model](/docs/app-development/threading)：UI 线程与 Dispatcher 的使用方式。
- [Compiled Bindings](/docs/data-binding/compiled-bindings)：编译时绑定验证与性能优势。
- [Collection Views](/docs/data-binding/collection-views)：高效的集合过滤与排序。
- [Hit Testing](/docs/graphics-animation/hit-testing)：命中测试机制与大量元素下的性能表现。
