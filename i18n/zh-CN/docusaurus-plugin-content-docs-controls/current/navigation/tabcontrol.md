---
id: tabcontrol
title: TabControl
description: Avalonia TabControl 指南，用于将内容组织为可切换的标签页。
doc-type: reference
---

[`TabControl`](/api/avalonia/controls/tabcontrol) 允许你将一个视图划分为多个标签项。

每个标签项都包含一个标题和一个内容区域。标题会以条带形式显示，并按照它们在 XAML 中出现的顺序排列。当你点击某个标签标题时，该标签的内容会变为可见，并显示在标签条下方的 TabControl 内容区域中。

你可以根据 Avalonia 应用的需求，自由组合标签标题区域和内容区域中的 UI。

:::info
如果你只需要这个控件的标签标题部分功能，可以考虑改用 [TabStrip](/controls/navigation/tabstrip)。
:::

## 常用属性

| 属性 | 类型 | 说明 |
|---|---|---|
| `TabStripPlacement` | `Dock` | 标签条的位置：`Top`、`Bottom`、`Left`、`Right`。默认值为 `Top`。 |
| `SelectedIndex` | `int` | 当前选中标签的从零开始索引。 |
| `SelectedItem` | `object` | 当前选中的标签项。 |
| `ItemsSource` | `IEnumerable` | 用于动态生成标签页的集合。 |
| `ItemTemplate` | `IDataTemplate` | 使用 `ItemsSource` 时用于标签标题的模板。 |
| `ContentTemplate` | `IDataTemplate` | 使用 `ItemsSource` 时用于标签内容的模板。 |

## 示例

这是一个简单的标签页示例，标签内容只是一些文本：

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui">
  <TabControl Margin="5">
    <TabItem Header="标签 1">
      <TextBlock Margin="5">这是标签 1 的内容</TextBlock>
    </TabItem>
    <TabItem Header="标签 2">
      <TextBlock Margin="5">这是标签 2 的内容</TextBlock>
    </TabItem>
  </TabControl>
</UserControl>
```

</XamlPreview>

## 标签位置

你可以通过设置 `TabStripPlacement` 属性，将标签放置到内容区域的任意一侧。默认值为 `Top`。下面的示例将标签放置在左侧：

```xml
<TabControl TabStripPlacement="Left">
    <TabItem Header="页面 1"><TextBlock Text="内容 1" Margin="8" /></TabItem>
    <TabItem Header="页面 2"><TextBlock Text="内容 2" Margin="8" /></TabItem>
</TabControl>
```

你也可以使用 `Bottom` 或 `Right`，将标签放置在内容区域的下方或右侧。

## 从集合动态生成标签页

你可以通过 `ItemsSource` 将 `TabControl` 绑定到视图模型中的某个集合。定义 `ItemTemplate` 用于控制标签标题的渲染方式，定义 `ContentTemplate` 用于控制内容区域：

```xml
<TabControl ItemsSource="{Binding Tabs}"
            SelectedItem="{Binding SelectedTab}">
    <TabControl.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Header}" />
        </DataTemplate>
    </TabControl.ItemTemplate>
    <TabControl.ContentTemplate>
        <DataTemplate>
            <ContentControl Content="{Binding Content}" />
        </DataTemplate>
    </TabControl.ContentTemplate>
</TabControl>
```

对应的视图模型大致可以写成这样：

```csharp
public class MainViewModel : ViewModelBase
{
    public ObservableCollection<TabItemViewModel> Tabs { get; } = new()
    {
        new TabItemViewModel("设置", "这里显示设置内容。"),
        new TabItemViewModel("账户", "这里显示账户内容。"),
    };

    public TabItemViewModel? SelectedTab { get; set; }
}

public class TabItemViewModel
{
    public string Header { get; }
    public string Content { get; }

    public TabItemViewModel(string header, string content)
    {
        Header = header;
        Content = content;
    }
}
```

## 延迟加载内容

默认情况下，`TabControl` 会在首次加载时为每个标签创建内容。如果你的标签页中包含较复杂的视图，你可以将每个标签内容包装在 `UserControl` 中，并通过 `DataTemplate` 按需加载，从而延迟到标签被选中时才创建内容：

```xml
<TabControl ItemsSource="{Binding Tabs}"
            SelectedItem="{Binding SelectedTab}">
    <TabControl.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Header}" />
        </DataTemplate>
    </TabControl.ItemTemplate>
    <TabControl.ContentTemplate>
        <DataTemplate DataType="vm:TabItemViewModel">
            <views:TabContentView />
        </DataTemplate>
    </TabControl.ContentTemplate>
</TabControl>
```

由于 `ContentTemplate` 会在每次选中标签时创建一个新的视图实例，因此可视树中只会存在当前可见标签页的内容。当你拥有很多标签页时，这可以减少内存占用并改善启动性能。

## 响应标签变化

你可以通过将 `SelectedIndex` 或 `SelectedItem` 绑定到视图模型来响应标签选择变化：

```xml
<TabControl SelectedIndex="{Binding ActiveTabIndex}">
    <TabItem Header="常规"><TextBlock Text="常规设置" Margin="8" /></TabItem>
    <TabItem Header="高级"><TextBlock Text="高级设置" Margin="8" /></TabItem>
</TabControl>
```

```csharp
public class SettingsViewModel : ViewModelBase
{
    private int _activeTabIndex;

    public int ActiveTabIndex
    {
        get => _activeTabIndex;
        set => this.RaiseAndSetIfChanged(ref _activeTabIndex, value);
    }
}
```

## 另请参阅

- [TabStrip](/controls/navigation/tabstrip)
- [Carousel](/controls/data-display/collections/carousel)
- [TabControl API 参考](/api/avalonia/controls/tabcontrol)
- [`TabControl.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/TabControl.cs)
