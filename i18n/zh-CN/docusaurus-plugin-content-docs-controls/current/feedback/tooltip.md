---
id: tooltip
title: ToolTip
---

import ToolTipTextHoverScreenshot from '/img/reference/controls/tooltip/tooltip-text-hover.gif';
import ToolTipContentScreenshot from '/img/reference/controls/tooltip/tooltip-content-hover.gif';

`ToolTip` 是一种弹出层，当用户将鼠标悬停在其附加的“宿主”控件上时，会显示其内容。

## 常用属性

你最常使用的通常是这些属性：

<table>
    <thead>
        <tr>
            <th width="298">属性</th>
            <th>说明</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>ToolTip.Tip</code></td>
            <td>用于设置工具提示内容的附加属性。</td>
        </tr>
        <tr>
            <td><code>ToolTip.Placement</code></td>
            <td>定义工具提示相对于宿主控件或指针的位置。可选值包括 top、bottom、left、right、anchor and gravity、pointer。默认值为 pointer，即在指针停止移动的位置显示提示内容。</td>
        </tr>
        <tr>
            <td><code>ToolTip.HorizontalOffset</code></td>
            <td>工具提示相对于放置位置的水平偏移量（默认 0）。</td>
        </tr>
        <tr>
            <td><code>ToolTip.VerticalOffset</code></td>
            <td>工具提示相对于放置位置的垂直偏移量（默认 20）。</td>
        </tr>
        <tr>
            <td><code>ToolTip.ShowDelay</code></td>
            <td>工具提示出现前，指针需要保持静止的时间，单位为毫秒（默认 400）。</td>
        </tr>
        <tr>
            <td><code>ToolTip.BetweenShowDelay</code></td>
            <td>上次显示后，在该时长内工具提示不会再次出现，单位为毫秒（默认 100）。</td>
        </tr>
        <tr>
            <td><code>ToolTip.ShowOnDisabled</code></td>
            <td>决定是否为禁用元素显示工具提示（默认 false）。</td>
        </tr>
        <tr>
            <td><code>ToolTip.ServiceEnabled</code></td>
            <td>决定工具提示服务是否启用（默认 true）。</td>
        </tr>
    </tbody>
</table>

## 事件

| 事件 | 类型 | 说明 |
|---|---|---|
| `ToolTip.ToolTipOpening` | `CancelRoutedEventArgs` | 当工具提示即将打开时触发。将 `Cancel = true` 可阻止工具提示显示。 |
| `ToolTip.ToolTipClosing` | `RoutedEventArgs` | 当工具提示即将关闭时触发。 |

这些都是附加路由事件。你可以在 XAML 或代码中订阅它们：

```xml
<Button Content="Hover me"
        ToolTip.Tip="Dynamic tooltip"
        ToolTip.ToolTipOpening="OnToolTipOpening" />
```

```csharp
private void OnToolTipOpening(object? sender, CancelRoutedEventArgs e)
{
    // 可选：阻止工具提示显示
    if (ShouldSuppressTooltip)
    {
        e.Cancel = true;
    }
}
```

## 示例

这是一个简单的文本工具提示示例，使用了放置位置和延迟属性的默认值。将鼠标悬停在预览中的矩形上即可看到工具提示。

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui">
  <Rectangle Fill="Aqua" Height="150" Width="200"
             ToolTip.Tip="This is a rectangle" />
</UserControl>
```

</XamlPreview>

<Image light={ToolTipTextHoverScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

如果你希望工具提示拥有更丰富的展示形式，请使用 `<ToolTip.Tip>` 元素。将鼠标悬停在预览中的矩形上即可看到工具提示。

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui">
  <Rectangle Fill="Aqua" Height="150" Width="200"
             ToolTip.Placement="Bottom">
      <ToolTip.Tip>
        <StackPanel>
          <TextBlock FontSize="16">Rectangle</TextBlock>
          <TextBlock>Some explanation here.</TextBlock>
        </StackPanel>
      </ToolTip.Tip>
  </Rectangle>
</UserControl>
```

</XamlPreview>

<Image light={ToolTipContentScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## 另请参阅

- [ToolTip API reference](/api/avalonia/controls/tooltip)
- [`ToolTip.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/ToolTip.cs)
