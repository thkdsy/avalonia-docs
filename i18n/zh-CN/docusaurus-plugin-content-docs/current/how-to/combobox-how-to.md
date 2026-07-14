---
id: combobox-how-to
title: "如何：使用 ComboBox"
description: 了解如何在 Avalonia ComboBox 中绑定集合、创建自定义模板、处理可编辑选择框以及绑定枚举值。
doc-type: how-to
---

本指南介绍常见的 [`ComboBox`](/api/avalonia/controls/combobox) 使用场景，包括绑定集合、创建自定义项模板、处理枚举，以及使用 [`AutoCompleteBox`](/api/avalonia/controls/autocompletebox) 实现输入即搜索功能。

## 基础绑定

若要把 `ComboBox` 绑定到一个集合并跟踪当前选中项，可以把 `ItemsSource` 绑定到集合属性，再把 `SelectedItem` 绑定到视图模型中的某个属性。同时可使用 `PlaceholderText` 在未选中任何项时显示提示信息：

```xml
<ComboBox ItemsSource="{Binding Countries}"
          SelectedItem="{Binding SelectedCountry}"
          PlaceholderText="Select a country" />
```

```csharp
public partial class MainViewModel : ObservableObject
{
    public ObservableCollection<string> Countries { get; } = new()
    {
        "United States", "United Kingdom", "Germany", "Japan", "Australia"
    };

    [ObservableProperty]
    private string? _selectedCountry;
}
```

:::tip
如果你希望在运行时添加或删除项目时 `ComboBox` 能自动更新，请使用 `ObservableCollection<T>`，而不是 `List<T>`。
:::

## 自定义项模板

当列表项是复杂对象时，可以使用 `ComboBox.ItemTemplate` 控制它们在下拉列表中的显示方式。这样你就可以展示多个属性、图标或任意自定义布局：

```xml
<ComboBox ItemsSource="{Binding Users}"
          SelectedItem="{Binding SelectedUser}"
          PlaceholderText="Select a user">
    <ComboBox.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal" Spacing="8">
                <Border Width="24" Height="24" CornerRadius="12"
                        Background="#6366F1">
                    <TextBlock Text="{Binding Initials}" Foreground="White"
                               HorizontalAlignment="Center"
                               VerticalAlignment="Center" FontSize="10" />
                </Border>
                <StackPanel>
                    <TextBlock Text="{Binding Name}" />
                    <TextBlock Text="{Binding Role}" FontSize="11" Foreground="Gray" />
                </StackPanel>
            </StackPanel>
        </DataTemplate>
    </ComboBox.ItemTemplate>
</ComboBox>
```

当你为复杂对象使用自定义项模板时，`ComboBox` 也会用同一模板来显示当前选中项。如果你希望“当前选中项”和“下拉项”采用不同布局，可以使用 `DataTemplateSelector`，或者对 popup 内部的项单独应用样式。

## 绑定到枚举

你可以通过调用 `Enum.GetValues<T>()` 并将结果公开为数组，把某个枚举的所有值填充到 `ComboBox` 中：

```csharp
public enum Priority { Low, Normal, High, Critical }

public partial class TaskViewModel : ObservableObject
{
    public Priority[] PriorityOptions { get; } = Enum.GetValues<Priority>();

    [ObservableProperty]
    private Priority _selectedPriority = Priority.Normal;
}
```

```xml
<ComboBox ItemsSource="{Binding PriorityOptions}"
          SelectedItem="{Binding SelectedPriority}" />
```

### 使用显示名称

默认情况下，`ComboBox` 会直接显示原始枚举成员名（例如 `"High"`，而不是 `"High Priority"`）。如果你希望显示更友好的文字，可以把每个枚举值包装成一个 record，并提供 `ItemTemplate`：

```csharp
public record PriorityOption(Priority Value, string Label);

public PriorityOption[] PriorityOptions { get; } = new[]
{
    new PriorityOption(Priority.Low, "Low Priority"),
    new PriorityOption(Priority.Normal, "Normal"),
    new PriorityOption(Priority.High, "High Priority"),
    new PriorityOption(Priority.Critical, "Critical!"),
};

[ObservableProperty]
private PriorityOption _selectedPriority;
```

```xml
<ComboBox ItemsSource="{Binding PriorityOptions}"
          SelectedItem="{Binding SelectedPriority}">
    <ComboBox.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Label}" />
        </DataTemplate>
    </ComboBox.ItemTemplate>
</ComboBox>
```

## 绑定到 `SelectedValue`

如果你只需要所选项中的某个属性值，而不是整个对象，可以使用 `SelectedValueBinding` 指定要提取的属性，再把结果绑定到 `SelectedValue`。当列表项是复杂对象、但你只想存储一个 ID 或代码时，这种方式尤其有用：

```xml
<ComboBox ItemsSource="{Binding Countries}"
          SelectedValueBinding="{Binding Code}"
          SelectedValue="{Binding SelectedCountryCode}">
    <ComboBox.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Name}" />
        </DataTemplate>
    </ComboBox.ItemTemplate>
</ComboBox>
```

## 在 XAML 中定义静态项

对于运行时不会变化的小型固定选项集，你可以直接在 XAML 中使用 `ComboBoxItem` 定义项目。通过 `SelectedIndex` 可以按位置预先选中某一项：

```xml
<ComboBox SelectedIndex="0">
    <ComboBoxItem Content="Small" />
    <ComboBoxItem Content="Medium" />
    <ComboBoxItem Content="Large" />
</ComboBox>
```

:::tip
静态项很适合那些在设计阶段就已确定选项的设置页或表单。对于动态或数据驱动的选项，建议改用 `ItemsSource` 绑定。
:::

## 可编辑选择框（`AutoCompleteBox`）

Avalonia 的 `ComboBox` 本身不带内置可编辑模式。如果你需要“边输入边筛选”的能力，也就是用户可以通过输入文本来过滤选项，请改用 `AutoCompleteBox`：

```xml
<AutoCompleteBox ItemsSource="{Binding AllCities}"
                 Text="{Binding SearchText}"
                 PlaceholderText="Search for a city..."
                 FilterMode="Contains"
                 MinimumPrefixLength="1" />
```

`AutoCompleteBox` 会在用户输入时自动过滤列表。你可以使用多种内置筛选模式（例如 `StartsWith`、`Contains`、`ContainsCaseSensitive` 等），也可以提供自定义筛选器：

```xml
<AutoCompleteBox ItemsSource="{Binding Users}"
                 FilterMode="Custom"
                 TextFilter="{Binding UserFilter}"
                 PlaceholderText="Search users...">
    <AutoCompleteBox.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Name}" />
        </DataTemplate>
    </AutoCompleteBox.ItemTemplate>
</AutoCompleteBox>
```

## 样式设置

### 自定义下拉宽度

如果下拉面板对你的内容来说太窄，可以在 `ComboBox` 模板内部的 `Popup` 上设置最小宽度：

```xml
<Style Selector="ComboBox /template/ Popup">
    <Setter Property="MinWidth" Value="300" />
</Style>
```

### 自定义占位文本样式

你可以修改未选中任何项时显示的占位文本样式：

```xml
<Style Selector="ComboBox:not(:selected) /template/ ContentControl#PlaceholderTextBlock">
    <Setter Property="Foreground" Value="Gray" />
</Style>
```

## 另请参阅

- [How to bind to a collection](/docs/data-binding/how-to-bind-to-a-collection): Collection binding basics.
- [Introduction to data templates](/docs/data-templates/introduction-to-data-templates): Customizing how items are displayed.
- [Collection views](/docs/data-binding/collection-views): Sorting, filtering, and grouping bound collections.
