---
id: hyperlinkbutton
title: HyperlinkButton
description: 一个样式类似文本超链接的按钮，使用平台默认处理程序打开 URI。
doc-type: reference
---

[`HyperlinkButton`](/api/avalonia/controls/hyperlinkbutton) 是一种外观类似文本超链接的按钮，点击时会打开一个 URI。它使用平台默认机制来启动 URI，例如打开浏览器、邮件客户端等。

## 常用属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `NavigateUri` | `Uri` | 点击按钮时要打开的 URI。 |
| `Content` | `object` | 按钮中显示的内容，通常为文本。 |
| `IsVisited` | `bool` | 链接是否已访问。URI 启动后会自动设为 `true`。 |
| `Command` | `ICommand` | 点击按钮时执行的可选命令。 |

## 基本示例

```xml
<HyperlinkButton NavigateUri="https://avaloniaui.net"
                 Content="Visit Avalonia" />
```

## 自定义内容

与 `Button` 一样，`HyperlinkButton` 支持任意内容：

```xml
<HyperlinkButton NavigateUri="https://github.com/AvaloniaUI/Avalonia">
    <StackPanel Orientation="Horizontal" Spacing="8">
        <PathIcon Data="{StaticResource github_icon}" />
        <TextBlock Text="View on GitHub" />
    </StackPanel>
</HyperlinkButton>
```

## 绑定 URI

```xml
<HyperlinkButton NavigateUri="{Binding ProjectUrl}"
                 Content="{Binding ProjectName}" />
```

## 平台行为

点击 `HyperlinkButton` 时，URI 的打开会交给操作系统的默认处理程序。最终打开的浏览器或应用取决于你的平台设置。例如，`https://` 链接会在默认网页浏览器中打开，而 `mailto:` 链接会在默认邮件客户端中打开。

## 伪类

| 伪类 | 说明 |
|---|---|
| `:visited` | 当 `IsVisited` 为 `true` 时应用。 |
| `:pressed` | 当按钮被按下时应用。 |

## 另请参阅

- [Button](/controls/input/buttons/button)：标准按钮。
- [RepeatButton](/controls/input/buttons/repeatbutton)
- [Launcher](/docs/services/launcher)：以编程方式启动 URI 和文件。
