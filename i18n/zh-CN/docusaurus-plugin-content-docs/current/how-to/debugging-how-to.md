---
id: debugging-how-to
title: "如何：调试常见的 Avalonia 问题"
description: 调试 Avalonia 应用程序中的绑定、布局、样式和渲染问题。
doc-type: how-to
---

本指南介绍如何调试 Avalonia 应用程序中的绑定、布局、样式和渲染问题。

## 调试数据绑定

### 启用绑定错误日志

Avalonia 会把绑定错误记录到 trace 输出中。你可以在 `Program.cs` 中这样配置日志：

```csharp
public static AppBuilder BuildAvaloniaApp()
    => AppBuilder.Configure<App>()
        .UsePlatformDetect()
        .LogToTrace(LogEventLevel.Warning);
```

绑定错误通常会以如下警告形式出现：
```text
[Binding] Error in binding to 'MyProperty' on 'MyControl': Could not find a matching property accessor...
```

### 常见绑定错误

| 现象 | 常见原因 |
|---|---|
| 控件什么都不显示 | 属性名拼错、`DataContext` 未设置，或 `DataContext` 类型不正确。 |
| 输出中出现 “Binding error” | `DataContext` 上不存在该属性。请检查大小写和拼写。 |
| 单向绑定正常，双向绑定无效 | 缺少属性 setter，或者属性没有触发 `PropertyChanged`。 |
| 集合更新后 UI 不变 | 使用了 `List<T>` 而不是 `ObservableCollection<T>`。 |
| 自定义控件忽略绑定值 | 在控件构造函数里设置了 `DataContext = this`。这会覆盖继承而来的 `DataContext`，导致父级设置的绑定失效。应删除这段赋值，并在控件模板中改用 `TemplateBinding`。 |

### 验证 DataContext

你可以在运行时检查某个控件实际拿到的 `DataContext`：

```csharp
// 在 code-behind 中用于调试
protected override void OnLoaded(RoutedEventArgs e)
{
    base.OnLoaded(e);
    System.Diagnostics.Debug.WriteLine($"DataContext: {DataContext?.GetType().Name}");
}
```

### 使用编译绑定在构建阶段捕获错误

```xml
<UserControl xmlns:vm="using:MyApp.ViewModels"
             x:DataType="vm:MainViewModel">
    <!-- 绑定错误会直接变成编译错误 -->
    <TextBlock Text="{Binding Name}" />
</UserControl>
```

## 调试布局问题

### 让布局可视化

可以临时加上有颜色的背景，用来观察空间是如何分配的：

```xml
<Grid ColumnDefinitions="200,*">
    <Border Grid.Column="0" Background="#10FF0000">
        <!-- 侧边栏内容 -->
    </Border>
    <Border Grid.Column="1" Background="#100000FF">
        <!-- 主内容 -->
    </Border>
</Grid>
```

### 检查零尺寸元素

宽度或高度为 0 的控件是不可见的。常见原因包括：

- `Canvas` 的子元素没有设置 `Width` / `Height`。
- `StackPanel` 的方向与期望的拉伸方向相反。
- `Image` 没有 `Source`，或者 URI 已失效。

### 使用 DevTools

在运行时按 **F12**（调试构建）可以打开 Avalonia DevTools：

- **Visual Tree** 标签页：查看渲染后的树结构以及每个控件的边界。
- **Properties** 标签页：检查实际属性值，包括尺寸。
- **Styles** 标签页：查看哪些样式已生效，以及它们的优先级。

## 调试样式

### 样式没有生效

| 现象 | 检查点 |
|---|---|
| 样式完全没效果 | 确认选择器是否匹配。可在 DevTools > Styles 中查看匹配到的样式。 |
| 本地值覆盖了样式 | 本地值（内联值）的优先级高于样式。请移除内联值。 |
| 文件中的样式没加载 | 确认该 `.axaml` 文件已经通过 `StyleInclude` 引入到 `App.axaml`。 |

### 验证样式选择器

可以在 DevTools 中测试你的选择器。如果无法匹配，常见问题包括：

- 缺少样式类（忘记给控件添加 `Classes="myClass"`）。
- 选择器中的控件类型写错了。
- 模板部件的选择器遗漏了 `/template/`。

## 调试渲染

### 控件不可见

建议按下面顺序检查：

1. **IsVisible** 是否为 `True`。
2. **Opacity** 是否大于 0。
3. 控件是否具有非零的 **Width** 和 **Height**（或位于会提供尺寸的面板中）。
4. 控件是否被父元素的 `ClipToBounds="True"` **裁剪** 掉了。
5. 控件是否处于 **可视区域** 内（没有被滚动出视图）。

### 渲染异常

- **文字或图标模糊**：检查图片的 `RenderOptions.BitmapInterpolationMode` 以及文本的 `TextOptions.TextHintingMode`。同时确认 `UseLayoutRounding="True"`。
- **闪烁**：这可能意味着出现了布局循环。检查是否有绑定在布局过程中再次触发布局。

## 调试性能问题

### 识别布局抖动

如果 UI 很卡，常见原因之一是布局遍历过于频繁。DevTools 的性能标签页可以看到布局次数。

### 检查虚拟化

对于大列表，请确认你使用的是支持虚拟化的面板：

```xml
<ListBox ItemsSource="{Binding LargeList}" />
<!-- ListBox 默认启用虚拟化 -->
```

像 `ItemsControl` 中的 `StackPanel` 这类不支持虚拟化的面板，会一次性创建所有项：

```xml
<!-- 不推荐：没有虚拟化 -->
<ItemsControl ItemsSource="{Binding LargeList}" />

<!-- 推荐：使用 ListBox 或配置 VirtualizingStackPanel -->
<ListBox ItemsSource="{Binding LargeList}" />
```

## 常用调试工具

| 工具 | 用途 |
|---|---|
| **DevTools (F12)** | 在运行时检查视觉树、属性、样式和事件。 |
| **Compiled Bindings** | 在编译期而不是运行时捕获绑定错误。 |
| **LogToTrace** | 在调试输出中查看绑定错误和其他警告。 |
| **Conditional breakpoints** | 在视图模型属性 setter 变化时中断。 |

## 另请参阅

- [Binding Debugging](/docs/data-binding/binding-debugging): Detailed binding diagnostics.
- [Compiled Bindings](/docs/data-binding/compiled-bindings): Catch binding errors at build time.
- [Performance](/docs/app-development/performance): Performance optimization tips.
