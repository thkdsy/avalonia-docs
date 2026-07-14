---
id: toolbar
title: 工具栏和选择弹窗
doc-type: guide
tags:
 - avalonia pro
 - avalonia enterprise
---

import DefaultToolbar from '/img/controls/richtexteditor/default-toolbar.png';
import CustomToolbarMinimal from '/img/controls/richtexteditor/custom-toolbar-minimal.png';
import WordCountTool from '/img/controls/richtexteditor/word-count-tool.png';
import AreaAware from '/img/controls/richtexteditor/area-aware.gif';
import DefaultMiniBar from '/img/controls/richtexteditor/default-mini-bar.png';
import CustomMiniBarMinimal from '/img/controls/richtexteditor/custom-mini-bar-minimal.png';
import DefaultContextMenu from '/img/controls/richtexteditor/default-context-menu.png';
import CustomContextMenu from '/img/controls/richtexteditor/custom-context-menu.png';
import CustomContextMenuSpecialized from '/img/controls/richtexteditor/custom-context-menu-specialized.png';
import CustomThemeToolbar from '/img/controls/richtexteditor/custom-theme-toolbar.png';

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

默认情况下，`RichTextEditor` 包含一个主工具栏、一个选择迷你栏和一个右键上下文菜单。这些工具栏中的每一个都由相同的基础系统构建，并且可以独立自定义。您可以交换布局、添加自己的工具、调整溢出菜单、重新主题化按钮等。本指南将带您了解从最常见到最高级的自定义选项。

:::info
此控件作为 [Avalonia Pro](https://avaloniaui.net/pricing) 或更高版本的一部分提供。
:::

## 工具栏架构

工具栏系统将 UI 呈现与行为逻辑分离。`EditorToolbar` 是一个 `TemplatedControl`，具有强类型的 `Tools` 集合（`AvaloniaList<EditorTool>`），标记为 `[Content]` 属性——XAML 子元素会自动添加到 `Tools` 中，并且只能是 `EditorTool` 实例。承载操作的工具派生自 `ActionTool`（它添加了 `Action`/`Icon`/`ToolTipText`）；分隔符和组直接派生自 `EditorTool`。

- **`EditorTool`**：`EditorToolbar` 或 `ToolbarGroup` 内托管的任何项目的抽象基类。它携带目标区域可见性、溢出元数据和编辑器主机发现。
- **`ActionTool`**：绑定到 `IEditorAction` 的工具的抽象基类。添加了 `Action`、`Icon` 和 `ToolTipText`，并将 `IsEnabled` 与 `Action.CanExecute(host)` 同步。大多数具体工具（按钮、切换、组合框、弹窗）都派生自此类。
- **`ToolbarGroup`**：本身是一个 `EditorTool`。托管自己的强类型 `Tools` 子工具集合。组共享可见性状态，即如果空间不足，它们会一起折叠到溢出菜单中。不支持嵌套的 `ToolbarGroup` 实例。
- **`EditorToolbar`**：一个 `TemplatedControl`，公开一个标记为 `[Content]` 属性的强类型 `Tools` 集合（`AvaloniaList<EditorTool>`）。将项目连接到编辑器，传播 `ActiveTargetAreas`，并运行溢出折叠布局传递。项目被插入到控件模板中名为 `PART_ItemsHost` 的 `Panel` 中。

`EditorToolbar` 和 `ToolbarGroup` 都使用名为 `PART_ItemsHost` 的 `Panel` 模板部件。默认主题使用 `WrapPanel`；嵌入在 `RichTextEditor` 内部的工具栏使用水平的 `StackPanel`。使用任何面板类型重新模板化任一控件以更改布局。

```
RichTextEditor
  └─ EditorToolbar            (TemplatedControl，[Content] Tools：AvaloniaList<EditorTool>)
       ├─ ToolbarGroup        (EditorTool，带 [Content] Tools——集体可见性)
       │    ├─ ButtonTool     (ActionTool)
       │    ├─ ToggleTool     (ActionTool)
       │    └─ ...
       ├─ SeparatorTool       (EditorTool，无操作界面)
       └─ OverflowTool        (ActionTool——"..." 按钮，托管折叠的工具)
```

### 命名空间

```csharp
using Avalonia.Controls.Documents.Primitives.Toolbar; // EditorToolbar, tools, groups
using Avalonia.Controls.Documents.Primitives.Actions; // EditorActions, IEditorAction
using Avalonia.Controls.Documents.Primitives.Adorners; // ToolbarTargetAreas
using Avalonia.Controls.Documents.Primitives; // EditorSelectionFlyout, EditorContextMenu
```

所有这些类型在默认 Avalonia 命名空间（`https://github.com/avaloniaui`）下在 XAML 中可用。

## 从较早的 `EditorToolbar` API 迁移

如果您有使用旧的基于 `ItemsControl` 的界面的现有代码，请按如下方式更新：

| 之前 | 之后 |
|---|---|
| `EditorToolbar.Items` / `ToolbarGroup.Items` | `Tools`（类型化 `AvaloniaList<EditorTool>`，`[Content]`） |
| `EditorToolbar.ItemsPanel` / `ItemsSource` | 已移除。使用名为 `PART_ItemsHost` 的 `Panel` 重新模板化工具栏。 |
| `EditorToolbar`/`ToolbarGroup` 派生自 `ItemsControl` | 两者现在都是 `TemplatedControl`。`ToolbarGroup` 派生自 `EditorTool`。 |
| `EditorTool.Action` / `Icon` / `ToolTipText` | 已移至 `ActionTool`。自定义操作绑定工具应派生自 `ActionTool`（被动工具保留在 `EditorTool` 上）。 |

隐式 XAML 子元素语法（`<EditorToolbar><ToolbarGroup>…</ToolbarGroup></EditorToolbar>`）保持不变——子元素通过 `[Content]` 属性添加到 `Tools`。只有显式的 `<EditorToolbar.Items>` / `<EditorToolbar.ItemsPanel>` 元素形式需要重命名。在代码中，将 `toolbar.Items.Add(...)` 替换为 `toolbar.Tools.Add(...)`。

## 默认工具栏

内置的 `EditorToolbar`，通过编辑器默认控件主题中的 `RichTextEditor.Toolbar` 填充，包含以下工具，按此顺序出现并分组。

<Image light={DefaultToolbar} position="center" cornerRadius="true" alt="默认的 RichTextEditor 工具栏，包含历史、剪贴板、字体、内联格式、列表、表格、块布局和溢出组。"/>
<br />

1. **历史**——撤销、重做
2. **剪贴板**——剪切、复制、粘贴、全选
3. **字体**——字体族、字号、前景色、背景色
4. **内联格式**——粗体、斜体、下划线、删除线、上标、下标、链接
5. **列表**——项目符号列表、编号列表
6. **表格**——插入表格
7. **块布局**——文本对齐、块边框
8. **溢出**——"..." 按钮，点击时显示折叠的工具

大多数工具是上下文相关的，这意味着它们会在脱离上下文时自动隐藏，例如，列表工具在列表外隐藏，表格工具在表格外隐藏。这是通过声明每个工具或组的 [`ToolbarTargetAreas`](#toolbartargetareas) 来实现的。

## 替换默认工具栏

有两种方式可以自定义默认工具栏，取决于您的 UI 需求。

### 选项 1：设置 `RichTextEditor.Toolbar`

将自定义 `EditorToolbar` 分配给 `RichTextEditor` 的 `Toolbar` 设置器。将其放置在编辑器的控件主题中，以便应用于应用程序中的每个 `RichTextEditor`：

```xml
<Application.Resources>
  <ControlTheme x:Key="{x:Type RichTextEditor}"
                TargetType="RichTextEditor"
                BasedOn="{StaticResource {x:Type RichTextEditor}}">
    <Setter Property="Toolbar">
      <Template>
        <EditorToolbar>
          <ToolbarGroup>
            <ButtonTool Action="{x:Static EditorActions.Undo}" />
            <ButtonTool Action="{x:Static EditorActions.Redo}" />
          </ToolbarGroup>

          <SeparatorTool />

          <ToolbarGroup Classes="AreaAware" TargetAreas="Text">
            <ToggleTool Action="{x:Static EditorActions.Bold}" />
            <ToggleTool Action="{x:Static EditorActions.Italic}" />
            <ToggleTool Action="{x:Static EditorActions.Underline}" />
          </ToolbarGroup>

          <OverflowTool />
        </EditorToolbar>
      </Template>
    </Setter>
  </ControlTheme>
</Application.Resources>
```

### 选项 2：独立于编辑器构建工具栏

如果您需要将工具栏放在编辑器以外的地方（例如，在侧面板、窗口标题栏中，或在多个编辑器之间共享），您可以隐藏内置工具栏并将 `EditorToolbar` 放置在您想要的任何位置。

为此，在 XAML 中定义独立的 `EditorToolbar`，并在相应的代码后置中将其附加到编辑器。

<Tabs>
<TabItem value="xaml" label="XAML">

```xml
<DockPanel>
  <EditorToolbar x:Name="MyToolbar" DockPanel.Dock="Top">
    <ToolbarGroup>
      <ButtonTool Action="{x:Static EditorActions.Undo}" />
      <ButtonTool Action="{x:Static EditorActions.Redo}" />
    </ToolbarGroup>
    <ToolbarGroup Classes="AreaAware" TargetAreas="Text">
      <ToggleTool Action="{x:Static EditorActions.Bold}" />
      <ToggleTool Action="{x:Static EditorActions.Italic}" />
    </ToolbarGroup>
  </EditorToolbar>

  <RichTextEditor x:Name="MyEditor" ShowToolbar="False" />
</DockPanel>
```

</TabItem>
<TabItem value="csharp" label="代码后置">

```csharp
public MainWindow()
{
    InitializeComponent();
    MyToolbar.Editor = MyEditor;
}
```

</TabItem>
</Tabs>

:::tip
您可以通过重新分配 `EditorToolbar.Editor` 在运行时将一个 `EditorToolbar` 连接到不同的编辑器。这是标签式文档界面的常见模式，其中单个共享工具栏跟踪活动标签页。
:::

### 极简示例

一个只包含撤销/重做和粗体/斜体的最小工具栏。注意工具图标是在与工具操作不同的标签中设置的。

<Image light={CustomToolbarMinimal} position="center" cornerRadius="true" alt="一个最小自定义工具栏，包含撤销、重做、粗体和斜体工具，由分隔符分隔。"/>
<br />

```xml
<ControlTheme x:Key="{x:Type RichTextEditor}"
              TargetType="RichTextEditor"
              BasedOn="{StaticResource {x:Type RichTextEditor}}">
  <Setter Property="Toolbar">
    <Template>
      <EditorToolbar Margin="4">

        <ButtonTool Action="{x:Static EditorActions.Undo}">
            <ButtonTool.Icon>
                <ContentPresenter ContentTemplate="{StaticResource EditorIcons.Undo}" />
            </ButtonTool.Icon>
        </ButtonTool>

        <ButtonTool Action="{x:Static EditorActions.Redo}">
            <ButtonTool.Icon>
                <ContentPresenter ContentTemplate="{StaticResource EditorIcons.Redo}" />
            </ButtonTool.Icon>
        </ButtonTool>

        <SeparatorTool />

        <ToggleTool Action="{x:Static EditorActions.Bold}">
            <ToggleTool.Icon>
                <ContentPresenter ContentTemplate="{StaticResource EditorIcons.Bold}" />
            </ToggleTool.Icon>
        </ToggleTool>

        <ToggleTool Action="{x:Static EditorActions.Italic}" >
            <ToggleTool.Icon>
                <ContentPresenter ContentTemplate="{StaticResource EditorIcons.Italic}" />
            </ToggleTool.Icon>
        </ToggleTool>

      </EditorToolbar>
    </Template>
  </Setter>
</ControlTheme>
```

## EditorTool 类型

工具栏上的组件，例如按钮、切换、组合框等，是 `EditorTool` 的子类。层次结构分为两部分：

- `EditorTool` 本身是任何工具栏项目的抽象基类。它处理目标区域可见性、溢出元数据、焦点返回辅助程序和编辑器主机发现。`SeparatorTool` 和 `ToolbarGroup` 直接派生自它，因为它们不绑定到操作。
- `ActionTool` 是添加操作承载属性（`Action`、`Icon`、`ToolTipText`）并与编辑器保持状态同步的抽象中间类。大多数具体控件——按钮、切换、组合框、弹窗——都派生自它。

在实践中，您很少需要直接使用任一基类。下面列出的内置子类被设计为满足大多数用例。

### 内置子类

| 类 | 基类 | 控件 | 典型用途 |
|---|---|---|---|
| `ButtonTool` | `ActionTool` | 按钮 | 一次性命令，例如撤销、剪切、粘贴。 |
| `ToggleTool` | `ActionTool` | 切换按钮 | 格式设置，例如粗体、斜体。 |
| `ListToggleTool` | `ToggleTool` | 拆分切换按钮 | 列表切换（项目符号/编号），同时反映当前标记样式。 |
| `ComboBoxTool` | `ActionTool` | 组合框 | 从列表中选择，例如字体族、字号。 |
| `ColorPickerTool` | `ColorTool` | 拆分按钮 + Avalonia `ColorPicker` 弹窗 | 选择任意颜色，例如前景色、背景色。 |
| `ColorSwatchTool` | `ColorTool` | 拆分按钮 + 色样面板弹窗 | 从固定调色板中选择。 |
| `AlignmentFlyoutTool` | `ActionTool` | 带弹窗的按钮 | 文本对齐——左、右、居中、两端对齐。 |
| `HyperlinkFlyoutTool` | `ActionTool` | 带弹窗的按钮 | 插入/编辑超链接。 |
| `TablePickerTool` | `ActionTool` | 网格选择器 | 通过指定网格大小插入表格。 |
| `BorderFlyoutTool` | `ActionTool` | 带弹窗的按钮 | 块边框配置，例如边、厚度、颜色。 |
| `OverflowTool` | `ActionTool` | 带弹窗的 "..." 按钮 | 呈现折叠工具的菜单弹窗。 |
| `SeparatorTool` | `EditorTool` | 垂直分割线 | 视觉分隔符。 |

项目符号和编号列表标记样式在默认主题中通过 `ButtonTool` 实例暴露，这些实例与驱动 `EditorActions.BulletMarkerStyle` / `EditorActions.NumberedMarkerStyle` 的 `PropertyMenuItem` 溢出表示配对——没有专用的 `BulletMarkerFlyoutTool` / `NumberedMarkerFlyoutTool`。

### 核心属性

`EditorTool` 在每个工具栏项目上暴露以下属性：

| 属性 | 类型 | 描述 |
|---|---|---|
| `TargetAreas` | `ToolbarTargetAreas` | 此工具应出现的上下文。请参阅 [ToolbarTargetAreas](#toolbartargetareas)。 |
| `OverflowMenuItem` | `MenuItem?` | 当此工具折叠到溢出菜单中时显示的菜单项。`null` 表示此工具不能折叠。 |
| `CanCollapseOverride` | `bool?` | 溢出折叠的显式覆盖。 |

`ActionTool` 添加了操作承载界面（由每个交互式工具继承）：

| 属性 | 类型 | 描述 |
|---|---|---|
| `Action` | `IEditorAction?` | 此工具执行的操作。 |
| `Icon` | `object?` | 工具的显示图标。 |
| `ToolTipText` | `string?` | 悬停时显示为工具提示的文本。如果未设置，默认为 `Action.DisplayName`。 |

### XAML 用法

```xml
<!-- 一次性命令 -->
<ButtonTool Action="{x:Static EditorActions.Undo}">
  <ButtonTool.Icon>
    <ContentPresenter ContentTemplate="{StaticResource EditorIcons.Undo}" />
  </ButtonTool.Icon>
</ButtonTool>

<!-- 格式化切换 -->
<ToggleTool Action="{x:Static EditorActions.Bold}">
  <ToggleTool.Icon>
    <ContentPresenter ContentTemplate="{StaticResource EditorIcons.Bold}" />
  </ToggleTool.Icon>
</ToggleTool>

<!-- 带有自定义项目模板的属性组合框 -->
<ComboBoxTool Action="{x:Static EditorActions.FontFamily}" Width="170">
  <ComboBoxTool.ItemTemplate>
    <DataTemplate DataType="FontFamily">
      <TextBlock Text="{Binding Name}" FontFamily="{Binding}" />
    </DataTemplate>
  </ComboBoxTool.ItemTemplate>
</ComboBoxTool>
```

## 创建自定义工具

如果您需要一个内置子类未覆盖的控件，可以派生自定义实现。选择与您的用例匹配的基类：

- **`EditorTool`**——用于不绑定到操作的被动控件（状态显示、装饰芯片）。重写 `OnApplyTemplate` 进行模板部件查找，重写 `OnEditorHostAttached` 以在编辑器主机可用时作出反应。
- **`ActionTool`**——用于执行 `IEditorAction` 的交互式工具。您继承 `Action`、`Icon`、`ToolTipText` 和一个 `UpdateState()` 虚拟方法，该方法在选择/内容/文档更改时运行。

在这两种情况下：

1. 公开一个渲染控件的控件模板。
2. 重写 `OnApplyTemplate` 以获取对模板部件的引用。
3. 对于 `ActionTool`，重写 `UpdateState` 以刷新额外状态（例如，自定义徽章）。对于 `EditorTool`，挂钩 `OnEditorHostAttached` 和您自己的事件订阅。
4. 在处理器内部，在执行操作前调用 `EnsureEditorFocus()`，以便光标返回编辑器。

### 示例：字数统计工具

<Image light={WordCountTool} position="center" cornerRadius="true" alt="一个自定义字数统计工具，停靠在工具栏末尾，显示当前字数。"/>
<br />

字数统计显示是一个被动控件——它不执行操作——因此它直接派生自 `EditorTool`。

实现通过遍历文档的 `DocumentSnapshot` 来统计字数。枚举 `Run` 节点并将块边界和换行符视为单词分隔符，避免分配完整的纯文本字符串，并避免合并一个段落的最后一个单词与下一个段落的第一个单词。

<Tabs>
<TabItem value="class" label="C#">

```csharp
using Avalonia.Controls;
using Avalonia.Controls.Primitives;
using Avalonia.Controls.Documents.TextModel;
using Avalonia.Controls.Documents.Serialization.Snapshot;

public class WordCountTool : EditorTool
{
    private TextBlock? _countText;

    protected override void OnApplyTemplate(TemplateAppliedEventArgs e)
    {
        base.OnApplyTemplate(e);
        _countText = e.NameScope.Find<TextBlock>("PART_Count");
        Refresh();
    }

    protected override void OnEditorHostAttached()
    {
        base.OnEditorHostAttached();
        Refresh();
    }

    protected override void UpdateState()
    {
        base.UpdateState();
        Refresh();
    }

    private void Refresh()
    {
        if (_countText is null) return;

        var doc = EditorHost?.TextDocument;
        if (doc is null)
        {
            _countText.Text = "0 字";
            return;
        }

        var snapshot = doc.CreateSnapshot();
        int words = CountWords(snapshot);
        _countText.Text = $"{words} 字";
    }

    private static int CountWords(DocumentSnapshot snapshot)
    {
        int count = 0;
        bool inWord = false;

        foreach (var node in snapshot.EnumerateNodes())
        {
            var kind = node.Kind;
            if (kind == TextDocumentNodeKind.Run)
            {
                int remaining = node.Length;
                int pos = node.StartOffset;
                while (remaining > 0)
                {
                    var chunk = snapshot.GetTextMemory(pos, remaining);
                    if (chunk.IsEmpty) break;

                    foreach (char c in chunk.Span)
                    {
                        if (char.IsWhiteSpace(c)) inWord = false;
                        else if (!inWord) { inWord = true; count++; }
                    }

                    pos += chunk.Length;
                    remaining -= chunk.Length;
                }
            }
            else if (kind == TextDocumentNodeKind.LineBreak
                || (kind.Flags & NodeKindFlags.Block) != 0)
            {
                // 块边界——绝不合并一个段落的最后一个单词
                // 与下一个段落的第一个单词。
                inWord = false;
            }
        }

        return count;
    }
}
```

</TabItem>
<TabItem value="theme" label="XAML 控件主题">

```xml
<ControlTheme x:Key="{x:Type local:WordCountTool}" TargetType="local:WordCountTool">
  <Setter Property="Template">
    <ControlTemplate>
      <Border Padding="8,4"
              MinWidth="72"
              VerticalAlignment="Center">
        <TextBlock Name="PART_Count"
                   Classes="Caption"
                   Foreground="{DynamicResource TextControlForeground}"
                   VerticalAlignment="Center" />
      </Border>
    </ControlTemplate>
  </Setter>
</ControlTheme>
```

</TabItem>
<TabItem value="usage" label="用法">

```xml
<EditorToolbar>
  <!-- ...其他组... -->
  <SeparatorTool />
  <local:WordCountTool />
</EditorToolbar>
```

</TabItem>
</Tabs>

## 编辑器操作

每个工具绑定到由静态 `EditorActions` 类提供的 `IEditorAction`。该操作告诉工具如何执行命令、何时可用，以及（对于切换和属性操作）显示什么状态。从 XAML 使用 `{x:Static EditorActions.<名称>}` 绑定，或直接从代码调用操作：

```csharp
if (EditorActions.Bold.CanExecute(editorHost))
    EditorActions.Bold.Execute(editorHost);

// 剪贴板操作是异步的
await EditorActions.Paste.ExecuteAsync(editorHost);

// 属性操作获取和设置类型化值
EditorActions.FontSize.SetValue(editorHost, 16.0);
var current = EditorActions.FontSize.GetValue(editorHost);
```

以下是内置的操作。

### 编辑操作

| 操作 | 手势 | 描述 |
|---|---|---|
| `Undo` | Ctrl+Z | 撤销上一步操作。 |
| `Redo` | Ctrl+Y | 重做上一步撤销的操作。 |
| `Cut` | Ctrl+X | 剪切当前选择到剪贴板。 |
| `Copy` | Ctrl+C | 复制当前选择到剪贴板。 |
| `Paste` | Ctrl+V | 在光标处粘贴剪贴板内容。 |
| `SelectAll` | Ctrl+A | 全选内容。 |

### 文本格式

| 操作 | 手势 | 描述 |
|---|---|---|
| `Bold` | Ctrl+B | 切换粗体。 |
| `Italic` | Ctrl+I | 切换斜体。 |
| `Underline` | Ctrl+U | 切换下划线。 |
| `Strikethrough` | Ctrl+- | 切换删除线。 |
| `Superscript` | Ctrl+Shift++ | 切换上标基线对齐。 |
| `Subscript` | Ctrl++ | 切换下标基线对齐。 |
| `FontFamily` | N/A | 获取或设置字体族。 |
| `FontSize` | N/A | 获取或设置字号。 |

### 颜色

| 操作 | 描述 |
|---|---|
| `ForegroundColor` | 获取或设置文本前景色。 |
| `BackgroundColor` | 获取或设置文本背景（高亮）色。 |

### 块对齐

| 操作 | 描述 |
|---|---|
| `AlignLeft` | 左对齐块。 |
| `AlignCenter` | 居中对齐块。 |
| `AlignRight` | 右对齐块。 |
| `AlignJustify` | 两端对齐块。 |

### 块间距和样式

| 操作 | 描述 |
|---|---|
| `LineHeight` | 获取或设置块行高。 |
| `Margin` | 获取或设置块边距。所有边统一。 |
| `Padding` | 获取或设置块内边距。所有边统一。 |
| `BlockBackground` | 获取或设置块背景色。 |
| `BorderThickness` | 获取或设置块边框厚度。 |
| `BorderBrush` | 获取或设置块边框颜色。 |
| `BlockBorder` | 打开或关闭块边框。 |

### 列表

| 操作 | 描述 |
|---|---|
| `ToggleBulletList` | 包装或取消包装为无序列表。 |
| `ToggleNumberedList` | 包装或取消包装为有序列表。 |
| `BulletMarkerStyle` | 设置项目符号标记样式（例如 Disc、Circle、Square）。 |
| `NumberedMarkerStyle` | 设置编号标记样式（例如 Decimal、LowerLatin、UpperRoman）。 |

### 表格

| 操作 | 描述 |
|---|---|
| `InsertTable` | 在光标处插入表格。默认为 3×3。 |
| `InsertRowBefore` | 在当前行上方插入一行。 |
| `InsertRowAfter` | 在当前行下方插入一行。 |
| `DeleteRow` | 删除当前行。 |
| `InsertColumnBefore` | 在当前列左侧插入一列。 |
| `InsertColumnAfter` | 在当前列右侧插入一列。 |
| `DeleteColumn` | 删除当前列。 |
| `DeleteTable` | 删除整个表格。 |

### 按 ID 查找

操作 ID 遵循模式 `"Category.Name"`（例如 `"Format.Bold"`、`"Table.InsertRowAfter"`）。使用 `GetById` 在运行时解析操作。

```csharp
var action = EditorActions.GetById("Format.Bold");
action?.Execute(editorHost);

// 枚举每个内置操作
foreach (var a in EditorActions.All)
    Console.WriteLine($"{a.Id} — {a.DisplayName}");
```

### Springload 行为

当选择为空时，切换和属性操作会设置一个 _springload_，意味着格式将应用于下一个输入的字符。这与用户从常见文字处理器中期望的行为一致。

## ToolbarGroup

`ToolbarGroup` 将一组相关工具分组，以共享集体可见性。如果组的 `TargetAreas` 与当前光标上下文不匹配，则整个组被隐藏。此外，如果布局改变，整个组会一起[溢出](#managing-the-overflow-menu)。

### 应用 `AreaAware` 类

`ToolbarGroup` 默认不响应编辑器的活动上下文。要启用上下文感知，添加 `AreaAware` 类。如果未设置，`AreaAware` 默认为 `All`，组始终可见。

<Image light={AreaAware} position="center" maxWidth={250} cornerRadius="true" alt="动画显示工具栏组在光标在正文、列表和表格之间移动时出现和消失。"/>
<br />

```xml
<!-- 文本格式组：仅在编辑文本时可见 -->
<ToolbarGroup Classes="AreaAware" TargetAreas="Text">
  <ToggleTool Action="{x:Static EditorActions.Bold}" />
  <ToggleTool Action="{x:Static EditorActions.Italic}" />
  <ToggleTool Action="{x:Static EditorActions.Underline}" />
</ToolbarGroup>

<!-- 表格操作组：仅在表格内可见 -->
<ToolbarGroup Classes="AreaAware" TargetAreas="Table">
  <ButtonTool Action="{x:Static EditorActions.InsertRowAfter}" />
  <ButtonTool Action="{x:Static EditorActions.DeleteRow}" />
</ToolbarGroup>

<!-- 始终可见 -->
<ToolbarGroup>
  <ButtonTool Action="{x:Static EditorActions.Undo}" />
  <ButtonTool Action="{x:Static EditorActions.Redo}" />
</ToolbarGroup>
```

## ToolbarTargetAreas

`ToolbarTargetAreas` 是一个 `[Flags]` 枚举，描述工具或组应可见的编辑上下文。`EditorToolbar` 在每次选择更改时检测活动上下文，并将其传播给 `AreaAware` 子级。

| 标志 | 光标上下文 |
|---|---|
| `Text` | 光标在任何可编辑文本位置。 |
| `Block` | 光标在块级上下文中。 |
| `Table` | 光标在表格中。 |
| `List` | 光标在列表中。 |
| `All` | 始终可见。（默认） |

如果需要，您可以组合 `ToolbarTargetAreas` 以使工具在多个上下文中可见。

```xml
<ToolbarGroup Classes="AreaAware" TargetAreas="Text,List">
```

## 管理溢出菜单

溢出菜单是工具栏末尾的 "..." 按钮。当工具栏的水平空间不足时，`EditorToolbar` 会将工具折叠到此菜单中，默认从右到左开始。

### 折叠规则

| `CanCollapseOverride` | `OverflowMenuItem` | 结果 |
|-----------------------|--------------------|---------|
| `null` | 已设置 | 可折叠（默认） |
| `null` | `null` | 不可折叠 |
| `true` | 已设置 | 可折叠 |
| `true` | `null` | 不可折叠（无菜单表示） |
| `false` | 已设置 | 不可折叠（固定） |
| `false` | `null` | 不可折叠 |

### 声明溢出表示

每个可折叠工具必须拥有自己的 `OverflowMenuItem`。在大多数情况下，菜单项可以是一个简单的 `EditorMenuItem`。

```xml
<!-- 折叠为带相同图标的简单菜单项 -->
<ButtonTool Action="{x:Static EditorActions.Cut}">
  <ButtonTool.Icon>
    <ContentPresenter ContentTemplate="{StaticResource EditorIcons.Cut}" />
  </ButtonTool.Icon>
  <ButtonTool.OverflowMenuItem>
    <EditorMenuItem Action="{x:Static EditorActions.Cut}">
      <EditorMenuItem.Icon>
        <ContentPresenter ContentTemplate="{StaticResource EditorIcons.Cut}" />
      </EditorMenuItem.Icon>
    </EditorMenuItem>
  </ButtonTool.OverflowMenuItem>
</ButtonTool>
```

### 专用工具的溢出

以下 `EditorMenuItem` 的专用子类适用于可能在溢出菜单中作为纯菜单项效果不佳的工具。

| 源工具 | 溢出项 | 行为 |
|---|---|---|
| `ComboBoxTool`（字体） | `FontFamilyMenuItem` | 字体子菜单，每个字体以其自身字型渲染。 |
| `ComboBoxTool`（通用） | `PropertyMenuItem` | 从操作自动填充值的子菜单。 |
| `ColorPickerTool` / `ColorSwatchTool` | `ColorMenuItem` | 带色样和可选"无颜色"的子菜单。 |
| `AlignmentFlyoutTool` | `TextAlignmentMenuItem` | 对齐选项的子菜单。 |

#### 示例

```xml
<!-- 字体选择器子菜单 -->
<ComboBoxTool Action="{x:Static EditorActions.FontFamily}" Width="170">
  <ComboBoxTool.OverflowMenuItem>
    <FontFamilyMenuItem Action="{x:Static EditorActions.FontFamily}" />
  </ComboBoxTool.OverflowMenuItem>
</ComboBoxTool>

<!-- 颜色选择器子菜单 -->
<ColorPickerTool Action="{x:Static EditorActions.ForegroundColor}"
                       ToolTipText="文本颜色">
  <ColorPickerTool.OverflowMenuItem>
    <ColorMenuItem Action="{x:Static EditorActions.ForegroundColor}"
                         Header="文本颜色" />
  </ColorPickerTool.OverflowMenuItem>
</ColorPickerTool>

<!-- 文本对齐子菜单 -->
<AlignmentFlyoutTool ToolTipText="文本对齐">
  <AlignmentFlyoutTool.OverflowMenuItem>
    <TextAlignmentMenuItem Header="文本对齐" />
  </AlignmentFlyoutTool.OverflowMenuItem>
</AlignmentFlyoutTool>
```

### 固定重要工具

为确保工具始终保留在工具栏中，即使水平空间紧张，也省略 `OverflowMenuItem`。撤销和重做在默认工具栏中以此方式固定。仅当您希望保留溢出定义用于条件切换时，才使用 `CanCollapseOverride="False"`。

```xml
<!-- 撤销永远不会被折叠 -->
<ButtonTool Action="{x:Static EditorActions.Undo}">
  <ButtonTool.Icon>
    <ContentPresenter ContentTemplate="{StaticResource EditorIcons.Undo}" />
  </ButtonTool.Icon>
</ButtonTool>
```

## 默认迷你栏

当用户选择文本时，一个紧凑的浮动工具栏会出现在选择附近。这就是 `SelectionFlyout`，一个 `EditorSelectionFlyout`（一个 `Flyout` 子类），它托管控件精简版的 `EditorToolbar`。

<Image light={DefaultMiniBar} position="center" cornerRadius="true" alt="默认选择迷你栏浮动在选中文本上方，显示内联格式、列表和块配置工具。"/>
<br />

默认迷你栏包括：

- **内联格式**：粗体、斜体、下划线、删除线、链接
- **列表切换**：项目符号、编号
- **块配置**：背景色、边框、文本对齐

迷你栏锚定到包含选择的块，而不是指针。

## 替换默认迷你栏

在 `<RichTextEditor>` XAML 标签内，添加 `<RichTextEditor.SelectionFlyout>` 并指定自定义的 `EditorSelectionFlyout`。这可以用来托管 `EditorToolbar`。

外部的名为 `PART_Chrome` 的 `Border` 是必需的——`EditorSelectionFlyout` 在处理解散事件时用它来区分 chrome 与周围的阴影边距。

此示例显示一个仅提供基本文本格式选项的最小迷你栏。

<Image light={CustomMiniBarMinimal} position="center" cornerRadius="true" alt="一个仅包含粗体、斜体和下划线切换的自定义选择迷你栏。"/>
<br />

```xml
<RichTextEditor>
  <RichTextEditor.SelectionFlyout>
    <EditorSelectionFlyout Placement="TopEdgeAlignedLeft"
                           OverlayDismissEventPassThrough="True">
      <Border Name="PART_Chrome" Classes="EditorMiniBarChrome">
        <EditorToolbar>

          <ToggleTool Action="{x:Static EditorActions.Bold}">
            <ToggleTool.Icon>
              <ContentPresenter ContentTemplate="{StaticResource EditorIcons.Bold}" />
            </ToggleTool.Icon>
          </ToggleTool>

          <ToggleTool Action="{x:Static EditorActions.Italic}">
            <ToggleTool.Icon>
              <ContentPresenter ContentTemplate="{StaticResource EditorIcons.Italic}" />
            </ToggleTool.Icon>
          </ToggleTool>

          <ToggleTool Action="{x:Static EditorActions.Underline}">
            <ToggleTool.Icon>
              <ContentPresenter ContentTemplate="{StaticResource EditorIcons.Underline}" />
            </ToggleTool.Icon>
          </ToggleTool>

        </EditorToolbar>
      </Border>
    </EditorSelectionFlyout>
  </RichTextEditor.SelectionFlyout>
</RichTextEditor>
```

一些建议：

- 对外部 `Border` 应用 `EditorMiniBarChrome` 样式类，以获得默认的圆角阴影外观。
- 保持工具集小巧。迷你栏应补充主工具栏，而不是重复它。

### 禁用迷你栏

在 `<RichTextEditor>` XAML 标签内，将 `SelectionFlyout` 设置为 `{x:Null}` 以完全关闭它。

```xml
<RichTextEditor SelectionFlyout="{x:Null}" />
```

## 默认上下文菜单

在文本编辑器中右键单击时打开的上下文菜单是 `EditorContextMenu`——一个附加到 `RichTextEditor.ContextFlyout` 的 `MenuFlyout` 子类。

<Image light={DefaultContextMenu} position="center" maxWidth={250} cornerRadius="true" alt="默认右键上下文菜单，包含剪切、复制、粘贴、全选和表格操作。"/>
<br />

默认上下文菜单包含：

- 剪切、复制、粘贴
- 全选
- 表格操作（插入/删除行、插入/删除列、删除表格）——仅在光标位于表格内时可见。

`EditorContextMenu` 可以读取每个 `EditorMenuItem` 上的 `TargetAreas`，以隐藏不适用于当前上下文的项目。被隐藏组留下的未使用的分隔线也会自动隐藏。

## 替换上下文菜单

在 `<RichTextEditor>` XAML 标签内，添加 `<RichTextEditor.ContextFlyout>` 并指定自定义的 `EditorContextMenu`。要向菜单添加操作，为 `<EditorMenuItem>` 添加单独的标签。

<Image light={CustomContextMenu} position="center" maxWidth={250} cornerRadius="true" alt="一个包含剪贴板命令和上下文相关表格操作的自定义上下文菜单。"/>
<br />

```xml
<RichTextEditor>
  <RichTextEditor.ContextFlyout>
    <EditorContextMenu Placement="Pointer">
      <EditorMenuItem Action="{x:Static EditorActions.Cut}" />
      <EditorMenuItem Action="{x:Static EditorActions.Copy}" />
      <EditorMenuItem Action="{x:Static EditorActions.Paste}" />

      <Separator />

      <EditorMenuItem Action="{x:Static EditorActions.SelectAll}" />

      <Separator />

      <!-- 仅在表格中出现 -->
      <EditorMenuItem Action="{x:Static EditorActions.InsertRowAfter}"
                      TargetAreas="Table" />
      <EditorMenuItem Action="{x:Static EditorActions.DeleteRow}"
                      TargetAreas="Table" />
      <EditorMenuItem Action="{x:Static EditorActions.DeleteTable}"
                      TargetAreas="Table" />
    </EditorContextMenu>
  </RichTextEditor.ContextFlyout>
</RichTextEditor>
```

### 添加属性子菜单

用于[溢出的相同专用菜单项](#overflow-for-specialized-tools)也适用于上下文菜单。

<Image light={CustomContextMenuSpecialized} position="center" maxWidth={250} cornerRadius="true" alt="一个包含字体族、文本颜色和对齐等专用子菜单的自定义上下文菜单。"/>
<br />

```xml
<EditorContextMenu Placement="Pointer">
  <EditorMenuItem Action="{x:Static EditorActions.Cut}" />
  <EditorMenuItem Action="{x:Static EditorActions.Copy}" />
  <EditorMenuItem Action="{x:Static EditorActions.Paste}" />

  <Separator />

  <FontFamilyMenuItem Action="{x:Static EditorActions.FontFamily}"
                            Header="字体" />
  <ColorMenuItem Action="{x:Static EditorActions.ForegroundColor}"
                       Header="文本颜色" />
  <TextAlignmentMenuItem Header="对齐" TargetAreas="Text,Block" />
</EditorContextMenu>
```

### 禁用上下文菜单

在 `<RichTextEditor>` XAML 标签内，将 `ContextFlyout` 设置为 `{x:Null}` 以完全关闭它。

```xml
<RichTextEditor ContextFlyout="{x:Null}" />
```

## 主题和样式

工具栏视觉效果通过动态资源和样式类控制。您可以在应用程序级别、窗口级别或单个 `EditorToolbar` 上覆盖默认设置。

### 大小和颜色资源

| 资源 | 默认值 | 用途 |
|---|---|---|
| `EditorToolbarToolHeight` | 30 | 工具按钮高度。 |
| `EditorToolbarToolMinWidth` | 28 | 工具按钮最小宽度。 |
| `EditorToolbarToolPadding` | 6,4 | 工具按钮的内边距。 |
| `EditorToolbarButtonCornerRadius` | 6 | 工具按钮的圆角半径。 |
| `EditorToolbarToolOpacity` | 0.9 | 工具不透明度。 |
| `EditorToolbarSeparatorHeight` | 18 | 分隔线高度。 |
| `EditorToolbarSubtleBorderBrush` | 主题 | 分隔线和细微边框颜色。 |
| `EditorToolbarPointerOverBackgroundBrush` | 主题 | 悬停背景色。 |
| `EditorToolbarCheckedBackgroundBrush` | 主题 | 活动/选中背景色。 |
| `EditorToolbarDisabledForegroundBrush` | 主题 | 禁用前景色。 |

#### 示例

<Image light={CustomThemeToolbar} position="center" cornerRadius="true" alt="一个覆盖了主题资源的工具栏，显示更大的按钮、减小的圆角半径和选中工具的自定义强调色。"/>
<br />

```xml
<Application.Resources>
  <ResourceDictionary>
    <!-- 使工具栏按钮更大 -->
    <x:Double x:Key="EditorToolbarToolHeight">36</x:Double>
    <x:Double x:Key="EditorToolbarToolMinWidth">36</x:Double>
    <CornerRadius x:Key="EditorToolbarButtonCornerRadius">4</CornerRadius>

    <!-- 为选中的工具使用自定义强调色 -->
    <SolidColorBrush x:Key="EditorToolbarCheckedBackgroundBrush"
                     Color="#4CAF50" Opacity="0.25" />
  </ResourceDictionary>
</Application.Resources>
```

### 样式类

| 类 | 应用于 | 效果 |
|---|---|---|
| `ToolbarTool` | `Button`、`ToggleButton`、`SplitButton` | 标准工具栏按钮大小、透明度、悬停/选中/禁用视觉效果、过渡。将标准按钮嵌入 `EditorToolbar` 时应用，使其与周围工具融合。 |
| `AreaAware` | `ToolbarGroup`、`ToggleTool`、`SeparatorTool`、`TablePickerTool`、`AlignmentFlyoutTool` 和其他 `EditorTool` 子类 | 绑定来自祖先 `EditorToolbar` 的活动目标区域，并可选择驱动 `IsVisible`。在 `ToolbarGroup` 上需要上下文可见性才能工作。 |
| `EditorMiniBarChrome` | `Border` | 默认选择迷你栏使用的紧凑圆角边框、投影外观。 |

#### 示例

向工具栏添加一个普通的 `Button`，使其看起来不突兀。

```xml
<EditorToolbar>
  <ToolbarGroup>
    <Button Classes="ToolbarTool" Click="OnExport_Click">
      <PathIcon Data="{StaticResource ExportGeometry}" />
    </Button>
  </ToolbarGroup>
  <!-- ... -->
</EditorToolbar>
```

### 定向样式

为了精确控制，编写一个针对工具栏中特定元素的选择器。

```xml
<Style Selector="EditorToolbar > ToolbarGroup > ToggleTool">
  <Setter Property="Margin" Value="2,0" />
</Style>
```

## 另请参阅

- [RichTextEditor 控件](/controls/input/text-input/richtexteditor)
- [扩展模式](/controls/input/text-input/richtexteditor/extension-patterns)
- [性能调优](/controls/input/text-input/richtexteditor/performance-tuning)
- [故障排除 RichTextEditor](/troubleshooting/controls/richtexteditor)
