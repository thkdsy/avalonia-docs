---
id: windowdrawndecorations
title: WindowDrawnDecorations
description: 一个逻辑元素，用于管理窗口装饰（例如标题栏和边框）在客户端侧的呈现方式，并定义标题按钮的交互行为。
doc-type: reference
---

`WindowDrawnDecorations` 不是一个可视控件，而是一个逻辑元素。它保存了窗口装饰（例如标题栏和边框）的模板与属性，用于管理这些装饰在客户端侧的呈现方式。

此外，`WindowDrawnDecorations` 还定义了标题按钮的交互逻辑。

该控件取代了 Avalonia 早期版本（12 之前）中的 `TitleBar`、`CaptionButtons` 和 `ChromeOverlayLayer` 类。

## 何时使用

当你需要创建自定义窗口装饰（例如标题栏、边框、标题按钮、缩放抓手等）时，可使用 `WindowDrawnDecorations`。

## 命名空间

位于 `Avalonia.Controls.Chrome`。

## 可视树结构

`WindowDrawnDecorations` 中的可视元素被划分为 **underlay**、**overlay** 和 **popover** 三层。它们会按照如下结构构建进可视树中：

```
Visual root
├── Underlay layer        （来自模板：边框、背景、阴影区域内容）
├── TopLevel/Window       （客户端区域：Window.Bounds 与此区域一致）
├── Overlay layer         （来自模板：标题栏、标题按钮）
├── FullscreenPopover     （来自模板：全屏时悬停触发的标题栏）
└── Resize hit-test zones （自动生成）
```

有关这些层在代码中的实现方式，请参阅 [`WindowDrawnDecorationsContent`](#windowdrawndecorationscontent)。

## WindowDrawnDecorationsTemplate

这是一种自定义模板类型，用于构建 `WindowDrawnDecorationsContent`。（参见 [WindowDrawnDecorationsContent](#windowdrawndecorationscontent)）

## WindowDrawnDecorationsContent

它承载 `WindowDrawnDecorationsTemplate` 使用的三个模板槽位。逻辑子元素会按照上文[可视树结构](#可视树结构)中描述的方式，被分配到不同的可视树层中。

```csharp
public class WindowDrawnDecorationsContent : StyledElement
{
    public Control? Overlay { get; set; }           // 标题栏、标题按钮
    public Control? Underlay { get; set; }          // 边框、背景、阴影
    public Control? FullscreenPopover { get; set; } // 悬停触发的全屏标题栏
}
```

## 属性

| 属性 | 类型 | 可见性 | 说明 |
| --- | --- | --- | --- |
| `Template` | `WindowDrawnDecorationsTemplate` | Styled | 装饰模板。 |
| `DefaultTitleBarHeight` | `double` | Styled | 默认标题栏高度。未设置时由主题决定。 |
| `DefaultFrameThickness` | `Thickness` | Styled | 默认边框厚度。未设置时由主题决定。 |
| `DefaultShadowThickness` | `Thickness` | Styled | 默认阴影厚度。未设置时由主题决定。 |
| `TitleBarHeight` | `double` | Styled | 实际生效的标题栏高度。若 `Window` 上设置了本地值，会覆盖这里。 |
| `FrameThickness` | `Thickness` | Styled | 实际生效的边框厚度。若 `Window` 上设置了本地值，会覆盖这里。 |
| `ShadowThickness` | `Thickness` | Styled | 实际生效的阴影厚度。若 `Window` 上设置了本地值，会覆盖这里。 |
| `Content` | `WindowDrawnDecorationsContent?` | Read-only | 构建后的模板内容。 |

## 装饰部件

可用的装饰部件包括：

- 阴影
- 边框
- 标题栏
- 缩放抓手

可用的装饰部件会因平台而异，例如 macOS 会自行处理缩放抓手。

## 伪类

当窗口状态变化时（例如 `Window` 从普通状态切换到全屏），会应用相应的伪类。当[装饰部件](#装饰部件)启用或禁用时，也会应用伪类，例如进入全屏可能会导致 `Shadow` 被禁用。

- `:normal`
- `:maximized`
- `:minimized`
- `:fullscreen`
- `:has-shadow`
- `:has-border`
- `:has-titlebar`

## 模板部件

`WindowDrawnDecorations` 会在模板应用过程中获取模板部件，并解析其中已指定的部分。所有模板部件都是可选的，例如你可以省略全屏按钮。

这部分功能取代了 Avalonia 早期版本中的 `CaptionButtons` 类。

| 部件 | 类型 | 说明 |
| --- | --- | --- |
| `PART_CloseButton` | `Button?` | 关闭按钮。 |
| `PART_MinimizeButton` | `Button?` | 最小化按钮。 |
| `PART_MaximizeButton` | `Button?` | 最大化切换按钮。 |
| `PART_FullScreenButton` | `Button?` | 全屏切换按钮。 |

## 元素角色

`ElementRole` 是一个[附加属性](/docs/properties#attached-properties)，用于为每个可视元素标记特定角色，以支持跨平台、非客户区命中测试。它可以应用到可视树中的任意元素上，而不仅限于装饰子元素。

可用的元素角色如下：

| 角色 | 说明 |
| --- | --- |
| `None` | 无角色。元素对命中测试不可见。 |
| `DecorationsElement` | 设置在装饰模板元素上的交互元素。输入会传递给该元素。 |
| `User` | 由用户代码设置的交互元素。输入会传递给该元素。 |
| `TitleBar` | 标题栏拖拽区域。点击并拖动该元素会移动窗口。 |
| `ResizeN` | 顶边（北）缩放抓手。 |
| `ResizeS` | 底边（南）缩放抓手。 |
| `ResizeE` | 右边（东）缩放抓手。 |
| `ResizeW` | 左边（西）缩放抓手。 |
| `ResizeNE` | 右上角（东北）缩放抓手。 |
| `ResizeSE` | 右下角（东南）缩放抓手。 |
| `ResizeNW` | 左上角（西北）缩放抓手。 |
| `ResizeSW` | 左下角（西南）缩放抓手。 |
| `CloseButton` | 元素执行窗口关闭行为。 |
| `MaximizeButton` | 元素执行窗口最大化行为。 |
| `MinimizeButton` | 元素执行窗口最小化行为。 |
| `FullScreenButton` | 元素执行窗口全屏切换行为。 |

在[下面的示例](#示例)中，一个充当标题栏的 `TextBlock` 被标记了元素角色：`ElementRole="TitleBar"`。

## 示例

```xml
<ControlTheme x:Key="{x:Type WindowDrawnDecorations}" TargetType="WindowDrawnDecorations">
    <Setter Property="DefaultTitleBarHeight" Value="32"/>
    <Setter Property="DefaultFrameThickness" Value="0,0,0,0"/>
    <Setter Property="DefaultShadowThickness" Value="16"/>
    <Setter Property="Template">
        <WindowDrawnDecorationsTemplate>
            <WindowDrawnDecorationsContent>

                <WindowDrawnDecorationsContent.Underlay>
                    <!-- 全尺寸：覆盖阴影区域 + 边框 + 客户区后方 -->
                    <Panel>
                        <Border Margin="{TemplateBinding ShadowThickness}"
                                Background="{DynamicResource WindowBackground}"
                                BorderThickness="1"
                                BorderBrush="{DynamicResource WindowBorderBrush}"
                                BoxShadow="0 8 32 0 #40000000"/>
                    </Panel>
                </WindowDrawnDecorationsContent.Underlay>

                <WindowDrawnDecorationsContent.Overlay>
                    <!-- 放置在标题栏区域上方 -->
                    <DockPanel Height="{TemplateBinding TitleBarHeight}"
                               VerticalAlignment="Top"
                               Margin="{TemplateBinding ShadowThickness}">
                        <StackPanel DockPanel.Dock="Right" Orientation="Horizontal">
                            <Button x:Name="PART_MinimizeButton" Content="─"/>
                            <Button x:Name="PART_MaximizeButton" Content="□"/>
                            <Button x:Name="PART_CloseButton" Content="✕"/>
                        </StackPanel>
                        <TextBlock Text="{Binding Title}"
                                   VerticalAlignment="Center"
                                   Margin="12,0"
                                   WindowDecorationProperties.ElementRole="TitleBar"/>
                    </DockPanel>
                </WindowDrawnDecorationsContent.Overlay>

                <WindowDrawnDecorationsContent.FullscreenPopover>
                    <!-- 全屏时在顶部边缘悬停显示 -->
                    <DockPanel Height="{TemplateBinding TitleBarHeight}"
                               Background="#E0000000">
                        <StackPanel DockPanel.Dock="Right" Orientation="Horizontal">
                            <Button x:Name="PART_MinimizeButton" Content="─"/>
                            <Button x:Name="PART_MaximizeButton" Content="□"/>
                            <Button x:Name="PART_CloseButton" Content="✕"/>
                        </StackPanel>
                        <TextBlock Text="{Binding Title}"
                                   Foreground="White"
                                   VerticalAlignment="Center"
                                   Margin="12,0"/>
                    </DockPanel>
                </WindowDrawnDecorationsContent.FullscreenPopover>

            </WindowDrawnDecorationsContent>
        </WindowDrawnDecorationsTemplate>
    </Setter>

    <!-- 伪类样式 -->
    <Style Selector="^:maximized /template/ Border">
        <Setter Property="BorderThickness" Value="0"/>
    </Style>
    <Style Selector="^:fullscreen /template/ DockPanel">
        <Setter Property="IsVisible" Value="False"/>
    </Style>
</ControlTheme>
```

## 另请参阅

- [Window 控件](/controls/primitives/window)
- [Avalonia v12 破坏性变更](/docs/avalonia12-breaking-changes)
