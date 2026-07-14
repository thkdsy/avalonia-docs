---
id: menu-how-to
title: "如何：使用菜单"
description: 学习如何在 Avalonia 中使用 Menu、ContextMenu 和 NativeMenu，包括命令、键盘快捷键、动态项、勾选项、子菜单以及右键菜单。
doc-type: how-to
---

本指南介绍 Avalonia 中 [`Menu`](/api/avalonia/controls/menu) 和 [`ContextMenu`](/api/avalonia/controls/contextmenu) 的常见使用模式，包括命令、键盘快捷键、动态菜单、勾选项、子菜单以及右键上下文菜单。

## 基础菜单栏

将 `Menu` 放在窗口顶部停靠的 `DockPanel` 中：

```xml
<DockPanel>
    <Menu DockPanel.Dock="Top">
        <MenuItem Header="_File">
            <MenuItem Header="_New" Command="{Binding NewCommand}" />
            <MenuItem Header="_Open..." Command="{Binding OpenCommand}" />
            <MenuItem Header="_Save" Command="{Binding SaveCommand}" />
            <Separator />
            <MenuItem Header="E_xit" Command="{Binding ExitCommand}" />
        </MenuItem>
        <MenuItem Header="_Edit">
            <MenuItem Header="_Undo" Command="{Binding UndoCommand}" />
            <Separator />
            <MenuItem Header="Cu_t" Command="{Binding CutCommand}" />
            <MenuItem Header="_Copy" Command="{Binding CopyCommand}" />
            <MenuItem Header="_Paste" Command="{Binding PasteCommand}" />
        </MenuItem>
    </Menu>
    <ContentControl Content="{Binding CurrentView}" />
</DockPanel>
```

字母前面的下划线用于定义访问键（Alt+字母）。例如 `_File` 表示用户可以按 Alt+F 打开 File 菜单。

:::tip
你可以在 [`MenuItem`](/api/avalonia/controls/menuitem) 之间插入 `Separator` 元素，作为视觉分隔线来归类相关操作。
:::

## 带键盘快捷键的菜单

使用 `InputGesture` 可以在菜单项旁边显示快捷键提示。需要注意，`InputGesture` 只负责显示文本，真正的快捷键功能仍然需要你单独注册按键绑定。

```xml
<MenuItem Header="_Save" Command="{Binding SaveCommand}"
          InputGesture="Ctrl+S" />
<MenuItem Header="Save _As..." Command="{Binding SaveAsCommand}"
          InputGesture="Ctrl+Shift+S" />
```

请把按键绑定注册到窗口或父级控件上，这样即使菜单没有展开，快捷键也仍然可以生效：

```xml
<Window.KeyBindings>
    <KeyBinding Gesture="Ctrl+S" Command="{Binding SaveCommand}" />
    <KeyBinding Gesture="Ctrl+Shift+S" Command="{Binding SaveAsCommand}" />
</Window.KeyBindings>
```

:::caution
如果你只设置了 `InputGesture`，却没有对应的 `KeyBinding`，那么菜单中虽然会显示快捷键文字，但用户按下组合键时不会有任何反应。因此这两者必须成对出现。
:::

### 平台适配的快捷键

在 macOS 上，你通常希望使用 Cmd 而不是 Ctrl。你可以通过为不同平台定义不同按键绑定，或者在代码中使用 Avalonia 的 `KeyModifiers.Meta` 来处理：

```csharp
var gesture = RuntimeInformation.IsOSPlatform(OSPlatform.OSX)
    ? new KeyGesture(Key.S, KeyModifiers.Meta)
    : new KeyGesture(Key.S, KeyModifiers.Control);
```

## 带图标的菜单

使用 `MenuItem.Icon` 属性为菜单项添加图标。你可以结合几何资源使用 `PathIcon`：

```xml
<MenuItem Header="Copy">
    <MenuItem.Icon>
        <PathIcon Data="{StaticResource copy_regular}" />
    </MenuItem.Icon>
</MenuItem>
```

或者在位图图标场景下使用 `Image`：

```xml
<MenuItem Header="Open">
    <MenuItem.Icon>
        <Image Source="/Assets/open.png" Width="16" Height="16" />
    </MenuItem.Icon>
</MenuItem>
```

:::tip
`PathIcon` 在任意 DPI 下都能保持清晰缩放，因此是大多数图标场景的推荐做法。只有当图标是复杂位图、无法表示为矢量路径时，才建议使用 `Image`。
:::

## 可勾选菜单项

你可以通过在 `MenuItem.Icon` 中放置一个 `CheckBox`，来模拟可勾选菜单项：

```xml
<MenuItem Header="Word Wrap"
          Command="{Binding ToggleWordWrapCommand}">
    <MenuItem.Icon>
        <CheckBox IsChecked="{Binding IsWordWrapEnabled}"
                  BorderThickness="0" Background="Transparent"
                  Content="" />
    </MenuItem.Icon>
</MenuItem>
```

更简单的做法，是在视图模型中维护一个布尔属性，再通过勾号图标的可见性来表示勾选状态：

```csharp
[ObservableProperty]
private bool _isWordWrapEnabled;

[RelayCommand]
private void ToggleWordWrap()
{
    IsWordWrapEnabled = !IsWordWrapEnabled;
}
```

```xml
<MenuItem Header="Word Wrap" Command="{Binding ToggleWordWrapCommand}">
    <MenuItem.Icon>
        <PathIcon Data="{StaticResource checkmark_regular}"
                  IsVisible="{Binding IsWordWrapEnabled}" />
    </MenuItem.Icon>
</MenuItem>
```

:::note
如果你的菜单项需要表现得像单选按钮（即同一组中始终只有一个激活项），那么应在视图模型中管理状态：当某一项被选中时，主动取消其他项的选中状态。
:::

## 子菜单

通过嵌套 `MenuItem` 元素可以创建子菜单。Avalonia 会自动显示一个展开箭头，并在悬停时打开子级弹出菜单：

```xml
<MenuItem Header="View">
    <MenuItem Header="Zoom">
        <MenuItem Header="Zoom In" Command="{Binding ZoomInCommand}" />
        <MenuItem Header="Zoom Out" Command="{Binding ZoomOutCommand}" />
        <MenuItem Header="Reset Zoom" Command="{Binding ResetZoomCommand}" />
    </MenuItem>
    <MenuItem Header="Panels">
        <MenuItem Header="Explorer" Command="{Binding ToggleExplorerCommand}" />
        <MenuItem Header="Output" Command="{Binding ToggleOutputCommand}" />
    </MenuItem>
</MenuItem>
```

建议避免把子菜单嵌套得超过两层，因为层级过深的菜单通常不利于用户操作。

## 从集合生成动态菜单

你可以通过绑定 `ItemsSource`，根据数据集合动态生成菜单项。这对于“最近文件”“窗口列表”或“插件注入操作”等场景特别有用。

先定义你的数据模型和集合：

```csharp
public ObservableCollection<RecentFile> RecentFiles { get; } = new();

public class RecentFile
{
    public string Name { get; set; }
    public string Path { get; set; }
    public ICommand OpenCommand { get; set; }
}
```

然后使用 `ItemContainerTheme` 把集合项映射为菜单项属性：

```xml
<MenuItem Header="Recent Files" ItemsSource="{Binding RecentFiles}">
    <MenuItem.ItemContainerTheme>
        <ControlTheme TargetType="MenuItem" BasedOn="{StaticResource {x:Type MenuItem}}">
            <Setter Property="Header" Value="{Binding Name}" />
            <Setter Property="Command" Value="{Binding OpenCommand}" />
            <Setter Property="CommandParameter" Value="{Binding Path}" />
        </ControlTheme>
    </MenuItem.ItemContainerTheme>
</MenuItem>
```

如果你只是想自定义显示内容，也可以使用 `DataTemplates`：

```xml
<MenuItem Header="Recent Files" ItemsSource="{Binding RecentFiles}">
    <MenuItem.DataTemplates>
        <DataTemplate DataType="vm:RecentFile">
            <TextBlock Text="{Binding Name}" />
        </DataTemplate>
    </MenuItem.DataTemplates>
</MenuItem>
```

### 显示空状态提示

当集合可能为空时，可以替换为一个禁用状态的占位菜单项：

```csharp
public IEnumerable<object> RecentFilesOrPlaceholder =>
    RecentFiles.Any()
        ? RecentFiles
        : new object[] { new MenuItem { Header = "(No recent files)", IsEnabled = false } };
```

### 更新动态菜单

由于 `RecentFiles` 是 `ObservableCollection`，因此当你添加或删除项时，菜单会自动更新。如果你直接替换了整个集合，则需要触发 `PropertyChanged`，让绑定刷新。

## 上下文菜单

通过 `ContextMenu` 属性，你可以给任意控件附加右键菜单。用户右击（或执行平台对应手势）时，该菜单就会打开：

```xml
<ListBox ItemsSource="{Binding Items}" SelectedItem="{Binding SelectedItem}">
    <ListBox.ContextMenu>
        <ContextMenu>
            <MenuItem Header="Edit" Command="{Binding EditCommand}" />
            <MenuItem Header="Delete" Command="{Binding DeleteCommand}" />
            <Separator />
            <MenuItem Header="Properties" Command="{Binding PropertiesCommand}" />
        </ContextMenu>
    </ListBox.ContextMenu>
</ListBox>
```

### 传递被点击的项目

可以使用 `CommandParameter` 配合绑定，把相关项目传递给命令：

```xml
<ListBox.ContextMenu>
    <ContextMenu>
        <MenuItem Header="Delete"
                  Command="{Binding DeleteCommand}"
                  CommandParameter="{Binding $parent[ListBox].SelectedItem}" />
    </ContextMenu>
</ListBox.ContextMenu>
```

:::tip
`$parent[ListBox]` 语法会沿着视觉树向上查找最近的 `ListBox` 祖先。之所以需要这样做，是因为 `ContextMenu` 与它的放置目标并不在同一棵视觉树中。
:::

### Disabling items based on state

如果你的命令实现了 `ICommand.CanExecute`，那么当 `CanExecute` 返回 `false` 时，`MenuItem` 会自动被禁用。使用 CommunityToolkit MVVM 源生成器时，你可以使用 `[RelayCommand(CanExecute = ...)]` 特性：

```csharp
[RelayCommand(CanExecute = nameof(CanDelete))]
private void Delete(object item)
{
    // delete logic
}

private bool CanDelete(object item) => item is not null;
```

## 在代码中创建上下文菜单

你也可以在 code-behind 中创建或修改上下文菜单：

```csharp
var contextMenu = new ContextMenu
{
    ItemsSource = new[]
    {
        new MenuItem { Header = "Cut", Command = CutCommand },
        new MenuItem { Header = "Copy", Command = CopyCommand },
        new MenuItem { Header = "Paste", Command = PasteCommand },
    }
};
myControl.ContextMenu = contextMenu;
```

## 打开与关闭事件

你可以处理菜单生命周期事件，从而在运行时动态定制菜单项，或者有条件地阻止菜单打开：

```xml
<ContextMenu Opening="ContextMenu_Opening" Closing="ContextMenu_Closing">
```

```csharp
private void ContextMenu_Opening(object? sender, CancelEventArgs e)
{
    // 根据当前状态动态定制菜单项。
    // 将 e.Cancel = true 可阻止菜单打开。
    if (sender is ContextMenu menu)
    {
        var canPaste = CheckClipboardContent();
        // 动态启用或禁用菜单项
    }
}

private void ContextMenu_Closing(object? sender, EventArgs e)
{
    // 菜单关闭时执行清理或记录日志。
}
```

## `ContextFlyout` 替代方案

当你需要的内容比普通菜单项列表更丰富时，可以使用 `ContextFlyout`。Flyout 中可以容纳任意控件布局：

```xml
<Border Background="LightGray" Padding="20">
    <Border.ContextFlyout>
        <Flyout>
            <StackPanel Spacing="8" Width="200">
                <TextBlock Text="Custom flyout content" FontWeight="Bold" />
                <TextBox PlaceholderText="Enter value..." />
                <Button Content="Apply" />
            </StackPanel>
        </Flyout>
    </Border.ContextFlyout>
    <TextBlock Text="Right-click for flyout" />
</Border>
```

:::caution
一个控件不能同时拥有 `ContextMenu` 和 `ContextFlyout`。如果你两个都设置了，最终只会有一个生效。标准菜单项列表请使用 `ContextMenu`，自定义复杂布局则使用 `ContextFlyout`。
:::

## [`NativeMenu`](/api/avalonia/controls/nativemenu) (macOS)

在 macOS 上，可以使用 `NativeMenu` 与屏幕顶部的系统菜单栏集成。这能提供符合 macOS 用户预期的原生体验：

```xml
<NativeMenu.Menu>
    <NativeMenu>
        <NativeMenuItem Header="About MyApp" Command="{Binding AboutCommand}" />
        <NativeMenuItemSeparator />
        <NativeMenuItem Header="Preferences..." Command="{Binding PreferencesCommand}" />
    </NativeMenu>
</NativeMenu.Menu>
```

:::note
`NativeMenu` 在 macOS 以外的平台上会被忽略。因此你可以放心地把它直接写进 AXAML，而无需条件编译。在 Windows 和 Linux 上，请改用标准 `Menu` 控件。
:::

## 另请参阅

- [Hotkeys](/docs/input-interaction/keyboard-and-hotkeys)：注册键盘快捷键和按键绑定。
- [Commanding](/docs/input-interaction/commanding)：在控件中使用命令。
- [Data binding to commands](/docs/data-binding/binding-to-commands)：在 MVVM 模式中绑定命令。
