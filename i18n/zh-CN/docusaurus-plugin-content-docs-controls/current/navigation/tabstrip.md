---
id: tabstrip
title: TabStrip
description: 一个不内置内容切换功能的标签头条带，让你能够完全控制所选标签内容的显示方式。
doc-type: reference
---

显示一排标签页头。你可以将此控件用作一个水平菜单。

[`TabStrip`](/api/avalonia/controls/primitives/tabstrip) 由 [`TabStripItem`](/api/avalonia/controls/primitives/tabstripitem) 子项组成。`TabStripItem` 会按出现顺序显示，并且可以通过以下两种方式创建：1. 直接提供一组 `TabStripItem`；2. 由 `ItemsSource` 自动生成。

与 [`TabControl`](/api/avalonia/controls/tabcontrol) 不同，`TabStrip` 不负责显示所选标签页的内容。相反，你需要响应 `SelectionChanged` 事件或 `SelectedItem` 属性的变化，然后通过单独的控件（例如 `ContentControl`）来显示所需内容。这使你能够完全控制显示内容，并支持诸如自定义视图缓存等用例。

## 示例：带 `TabStripItem` 的 `TabStrip`

<XamlPreview>

```xml
<TabStrip xmlns="https://github.com/avaloniaui" Margin="5">
  <TabStripItem>Tab 1</TabStripItem>
  <TabStripItem>Tab 2</TabStripItem>
</TabStrip>
```

</XamlPreview>

## 示例：带 `ItemsSource` 的 `TabStrip`

```xml
<TabStrip ItemsSource="{Binding MyTabs}" SelectedItem="{Binding MySelectedItem}">
    <TabStrip.ItemTemplate>
        <DataTemplate x:DataType="vm:MyViewModel">
            <TextBlock Text="{Binding Header}"/>
        </DataTemplate>
    </TabStrip.ItemTemplate>
</TabStrip>
```

## 响应选择变化

由于 `TabStrip` 不会显示所选标签页的内容，因此你需要将它与诸如 `ContentControl` 这样的独立控件搭配使用。将 `SelectedIndex`（或 `SelectedItem`）绑定到视图模型，然后在其变化时切换要显示的内容。

```xml
<DockPanel>
    <TabStrip DockPanel.Dock="Top"
              SelectedIndex="{Binding SelectedIndex}">
        <TabStripItem>Home</TabStripItem>
        <TabStripItem>Settings</TabStripItem>
    </TabStrip>
    <ContentControl Content="{Binding CurrentPage}" />
</DockPanel>
```

```csharp
public class MainViewModel : ViewModelBase
{
    private int _selectedIndex;
    private object? _currentPage;

    public int SelectedIndex
    {
        get => _selectedIndex;
        set
        {
            if (_selectedIndex != value)
            {
                _selectedIndex = value;
                OnPropertyChanged();
                UpdateCurrentPage();
            }
        }
    }

    public object? CurrentPage
    {
        get => _currentPage;
        set
        {
            _currentPage = value;
            OnPropertyChanged();
        }
    }

    private void UpdateCurrentPage()
    {
        CurrentPage = SelectedIndex switch
        {
            0 => new HomeViewModel(),
            1 => new SettingsViewModel(),
            _ => null
        };
    }
}
```

## 何时使用 `TabStrip`，何时使用 `TabControl`

当你需要自定义内容切换逻辑时，例如视图缓存或延迟加载，应使用 `TabStrip`。由于 `TabStrip` 不管理内容显示，因此你可以精确控制视图何时创建、缓存或释放。

当你希望使用内置内容显示功能时，应使用 `TabControl`。`TabControl` 同时处理标签头和内容区域，因此你无需自己编写内容切换逻辑。

## 另请参阅

- [TabControl](/controls/navigation/tabcontrol)
- [TabStrip API reference](/api/avalonia/controls/primitives/tabstrip)
- [`TabStrip.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/Primitives/TabStrip.cs)
