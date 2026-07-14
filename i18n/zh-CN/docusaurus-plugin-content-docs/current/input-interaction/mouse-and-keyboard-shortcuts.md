---
id: mouse-and-keyboard-shortcuts
title: 创建鼠标和键盘快捷方式
description: 了解如何在 Avalonia 控件中使用 KeyBindings、手势和上下文菜单来绑定键盘快捷键，并处理双击等鼠标事件。
doc-type: how-to
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

import KeyMouseScreenshot from '/img/guides/ui-development/binding-key-mouse-test.gif';

Avalonia 允许你为控件配置键盘快捷键和鼠标操作，从而让用户无需依赖工具栏或菜单也能完成交互。本页将带你了解最常见的几种模式：使用 `KeyBindings` 将按键绑定到命令、通过 `DoubleTapped` 事件处理双击，以及添加右键上下文菜单。

## 键盘绑定

你可以把一个或多个 [`KeyBinding`](/api/avalonia/input/keybinding) 元素附加到任意控件的 `KeyBindings` 集合中。每个 `KeyBinding` 都会将一个 `Gesture`（某个按键，外加可选修饰键）映射到视图模型中的某个命令。

```xml
<ListBox.KeyBindings>
    <KeyBinding Command="{Binding PrintItem}" Gesture="Enter" />
    <KeyBinding Command="{Binding DeleteItem}" Gesture="Delete" />
    <KeyBinding Command="{Binding SelectAll}" Gesture="Ctrl+A" />
</ListBox.KeyBindings>
```

`Gesture` 字符串会被解析为 `KeyGesture`。你可以使用 `Ctrl`、`Shift`、`Alt` 和 `Cmd` 等修饰键简写。支持的按键和修饰键完整列表，请参阅 [键盘与热键](/docs/input-interaction/keyboard-and-hotkeys) 参考页。

:::tip
`KeyBinding` 只有在该控件（或其子元素）拥有键盘焦点时才会触发。如果你需要一个无论焦点在哪里都能工作的应用级快捷键，请改用 `MenuItem` 或其他 `ICommandSource` 上的 `HotKey`。
:::

## 处理双击

Avalonia 没有提供与 `MouseBinding` 等价的机制。若要响应双击，请在 code-behind 中处理 `DoubleTapped` 事件，并将该动作转发给视图模型：

```csharp
private void ListBox_DoubleTapped(object? sender, Avalonia.Input.TappedEventArgs e)
{
    if (DataContext is MainViewModel vm)
    {
        vm.PrintItem.Execute(null);
    }
}
```

在 XAML 中，可以使用 `DoubleTapped` 属性附加该处理器：

```xml
<ListBox DoubleTapped="ListBox_DoubleTapped"
         ItemsSource="{Binding OperatingSystems}"
         SelectedItem="{Binding OS}" />
```

## 添加上下文菜单

你可以把 `ContextMenu` 附加到任意控件上。当用户右键点击（或在触摸设备上长按）时，菜单就会出现。将每个 `MenuItem` 绑定到视图模型中的命令即可：

```xml
<TextBlock Text="{Binding Result}">
    <TextBlock.ContextMenu>
        <ContextMenu>
            <MenuItem Command="{Binding Clear}" Header="清除" />
        </ContextMenu>
    </TextBlock.ContextMenu>
</TextBlock>
```

## 完整示例

下面的示例将这三种技术组合在同一个视图中。一个 `ListBox` 用于显示操作系统列表。按下 Enter 或双击某一项会将其输出到 `TextBlock` 中，而右键点击 `TextBlock` 则会清除结果。

<Tabs
  defaultValue="xaml"
  values={[
      { label: 'XAML', value: 'xaml', },
      { label: 'Code-behind', value: 'code-behind', },
      { label: 'ViewModel', value: 'ViewModel', },
  ]}
>
<TabItem value="xaml">

```xml
<UserControl ..>
    <StackPanel>
        <ListBox
            DoubleTapped="ListBox_DoubleTapped"
            ItemsSource="{Binding OperatingSystems}"
            SelectedItem="{Binding OS}">
            <ListBox.KeyBindings>
                <!--  Enter  -->
                <KeyBinding Command="{Binding PrintItem}" Gesture="Enter" />
                <!--
                    MouseBindings 不受支持。
                    请改为在视图的 code-behind 中处理。（DoubleTapped 事件）
                -->
            </ListBox.KeyBindings>
        </ListBox>
        <TextBlock Text="{Binding Result}">
            <TextBlock.ContextMenu>
                <ContextMenu>
                    <!--  右键点击  -->
                    <MenuItem Command="{Binding Clear}" Header="清除" />
                </ContextMenu>
            </TextBlock.ContextMenu>
        </TextBlock>
    </StackPanel>
</UserControl>
```

</TabItem>
<TabItem value="code-behind">

```csharp
public partial class MainView : UserControl
{
    public MainView()
    {
        InitializeComponent();
    }

    private void ListBox_DoubleTapped(object? sender, Avalonia.Input.TappedEventArgs e)
    {
        if (DataContext is MainViewModel vm)
        {
            vm.PrintItem.Execute(null);
        }
    }
}
```
</TabItem>

<TabItem value="ViewModel">

```csharp
public class MainViewModel : ViewModelBase
{
    public List<string> OperatingSystems =>
    [
        "Windows",
        "Linux",
        "Mac",
    ];
    public string OS { get; set; } = string.Empty;

    [Reactive]
    public string Result { get; set; } = string.Empty;

    public ICommand PrintItem { get; }
    public ICommand Clear { get; }

    public MainViewModel()
    {
        PrintItem = ReactiveCommand.Create(() => Result = OS);
        Clear = ReactiveCommand.Create(() => Result = string.Empty);
    }
}
```
</TabItem>
</Tabs>

<Image light={KeyMouseScreenshot} alt="Demo showing keyboard and mouse shortcut interactions with a ListBox" position="center" maxWidth={400} cornerRadius="true"/>

## 平台差异说明

| 平台 | 行为 |
|---|---|
| **macOS** | 对于符合平台习惯的快捷键，请使用 `Cmd` 代替 `Ctrl`（例如用 `Cmd+S` 保存）。你也可以同时绑定 `Ctrl` 和 `Cmd` 两种变体到同一个命令，以覆盖所有平台。 |
| **Linux / X11** | 默认情况下，上下文菜单会通过右键点击打开。由于 X11 不提供触摸长按事件，因此不支持通过长按弹出上下文菜单。 |
| **移动端（Android / iOS）** | 如果没有连接物理键盘，`KeyBinding` 不会生效。对于以触摸为主的交互，请使用手势识别器和 `ContextMenu`（通过长按激活）。 |
| **浏览器（WASM）** | 大多数按键手势都能工作，但某些被浏览器保留的快捷键（例如 `Ctrl+T` 或 `Ctrl+W`）无法被应用拦截。 |

## 另请参阅

- [键盘与热键](/docs/input-interaction/keyboard-and-hotkeys)：按键绑定与热键配置。
- [命令系统](/docs/input-interaction/commanding)：`ICommand` 接口与命令绑定。
- [手势](/docs/input-interaction/gestures)：点击、双击与多指手势识别器。
- [添加交互性](/docs/input-interaction/adding-interactivity)：事件与命令概览。
