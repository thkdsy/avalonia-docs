---
id: groupbox
title: GroupBox
description: 一个容器控件，通过带边框的标题区域在视觉上对相关内容进行分组。
doc-type: reference
---

[`GroupBox`](/api/avalonia/controls/groupbox) 控件会在一个标题标签下方、通过边框将相关内容在视觉上分组。标题文本会覆盖在边框的上沿，从而形成桌面 UI 框架中常见的经典“分组框”外观。

`GroupBox` 继承自 `HeaderedContentControl`，因此它支持一个 `Header`（显示在边框顶部）以及一个 `Content` 子元素。

## 常见使用场景

在以下情况下可以使用 `GroupBox`：

- 将表单字段组织为逻辑分区（例如“个人信息”和“账单地址”）。
- 在视觉上分隔相关选项组，例如复选框组或单选按钮组。
- 在某段 UI 外围添加带标题的边框，使分组控件的用途一目了然。

## 常用属性

你最常使用的通常是这些属性：

<table><thead><tr><th width="261">属性</th><th>说明</th></tr></thead><tbody><tr><td><code>Header</code></td><td>显示在边框顶部的文本或内容。</td></tr><tr><td><code>Content</code></td><td>承载在分组框内部的子控件或布局。</td></tr><tr><td><code>BorderBrush</code></td><td>外围边框的颜色。</td></tr><tr><td><code>BorderThickness</code></td><td>外围边框的厚度。</td></tr><tr><td><code>CornerRadius</code></td><td>边框圆角半径。</td></tr><tr><td><code>Padding</code></td><td>边框与内容之间的间距。</td></tr></tbody></table>

## 示例

此示例创建了两个分组框来组织一个表单：

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui"
            Spacing="16" Margin="16">
  <GroupBox Header="Personal Details">
    <StackPanel Spacing="8">
      <TextBox PlaceholderText="First name" />
      <TextBox PlaceholderText="Last name" />
      <TextBox PlaceholderText="Email" />
    </StackPanel>
  </GroupBox>
  <GroupBox Header="Preferences">
    <StackPanel Spacing="8">
      <CheckBox Content="Receive notifications" />
      <CheckBox Content="Dark mode" />
    </StackPanel>
  </GroupBox>
</StackPanel>
```

</XamlPreview>

## 自定义标题内容

`Header` 属性接受任意内容，而不仅仅是文本。你可以用它来显示图标、格式化文本，或任何其他控件：

```xml
<GroupBox>
  <GroupBox.Header>
    <StackPanel Orientation="Horizontal" Spacing="6">
      <PathIcon Data="{StaticResource SettingsIcon}" />
      <TextBlock Text="Advanced Settings" FontWeight="Bold" />
    </StackPanel>
  </GroupBox.Header>
  <StackPanel Spacing="8">
    <CheckBox Content="Enable logging" />
    <CheckBox Content="Verbose output" />
  </StackPanel>
</GroupBox>
```

## 绑定标题

你可以将 `Header` 属性绑定到视图模型属性，这在分区标题需要运行时更新时会很有用：

```xml
<GroupBox Header="{Binding SectionTitle}">
  <TextBlock Text="{Binding SectionContent}" />
</GroupBox>
```

```csharp
public class MyViewModel : ViewModelBase
{
    private string _sectionTitle = "Details";

    public string SectionTitle
    {
        get => _sectionTitle;
        set => this.RaiseAndSetIfChanged(ref _sectionTitle, value);
    }
}
```

## 嵌套分组框

你可以将 `GroupBox` 嵌套使用，以创建子分区。建议保持较浅的嵌套层级（一到两层），以避免布局显得过于拥挤：

```xml
<GroupBox Header="Account">
  <StackPanel Spacing="12">
    <GroupBox Header="Login credentials">
      <StackPanel Spacing="8">
        <TextBox PlaceholderText="Username" />
        <TextBox PlaceholderText="Password" PasswordChar="*" />
      </StackPanel>
    </GroupBox>
    <GroupBox Header="Profile">
      <StackPanel Spacing="8">
        <TextBox PlaceholderText="Display name" />
        <TextBox PlaceholderText="Bio" />
      </StackPanel>
    </GroupBox>
  </StackPanel>
</GroupBox>
```

## 样式设置

你可以通过主题资源来自定义 `GroupBox` 的外观：

| 资源 | 默认值 | 说明 |
|---|---|---|
| `GroupBoxPadding` | `4` | 内容区域周围的内部内边距。 |
| `GroupBoxHeaderFontSize` | `16` | 标题文本的字体大小。 |
| `GroupBoxHeaderMargin` | `0,4,0,12` | 标题周围的外边距。 |
| `GroupBoxBorderThickness` | `1` | 外围边框的厚度。 |
| `GroupBoxBackground` | Transparent | 内容区域的背景填充。 |
| `GroupBoxBorderBrush` | `SystemControlForegroundBaseMediumBrush` | 边框颜色。 |
| `GroupBoxHeaderForeground` | `SystemBaseHighColor` | 标题文本颜色。 |

## 另请参阅

- [Border](/controls/layout/containers/border)
- [Expander](/controls/layout/containers/expander)
- [GroupBox API reference](/api/avalonia/controls/groupbox)
- [`GroupBox.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/GroupBox.cs)
