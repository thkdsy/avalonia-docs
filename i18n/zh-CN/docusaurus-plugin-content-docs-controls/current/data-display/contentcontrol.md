---
id: contentcontrol
title: ContentControl
description: 一个用于显示单个内容项的基础控件，该内容可以是字符串、控件，或通过数据模板渲染的数据绑定对象。
doc-type: reference
---

import ControlContentStudentScreenshot from '/img/controls/contentcontrol/contentcontrol-student.png';

[`ContentControl`](/api/avalonia/controls/contentcontrol) 是一个用于显示单个内容项的控件。其内容可以是字符串、控件，或通过 `DataTemplate` 渲染的数据绑定对象。许多常见的 Avalonia 控件——包括 `Button`、`Window` 和 `UserControl`——都继承自 `ContentControl`，因此理解它的工作方式对于构建 Avalonia 应用至关重要。

## 常用属性

你最常使用的通常是这些属性：

| 属性 | 说明 |
|---|---|
| `Content` | 要在控件中显示的内容。 |
| `ContentTemplate` | 用于渲染 `Content` 对象的 `DataTemplate`。 |
| `HorizontalContentAlignment` | 控制内容在控件中的水平对齐方式。 |
| `VerticalContentAlignment` | 控制内容在控件中的垂直对齐方式。 |

## 显示内容

最简单的情况下，`ContentControl` 会直接显示你赋值给其 [`Content`](/api/avalonia/controls/contentcontrol#content-property) 属性的数据。

例如：

```xml
<ContentControl Content="Hello World!"/>
```

这会显示字符串 “Hello World!”。由于 `Content` 是该控件的默认内容属性，因此你也可以这样写：

```xml
<ContentControl>Hello World!</ContentControl>
```

### 承载子控件

如果你将一个控件赋值给 `ContentControl`，它会直接渲染该控件：

```xml
<ContentControl>
  <Button>Click Me!</Button>
</ContentControl>
```

`ContentControl` 只能容纳一个直接子元素。如果你需要显示多个元素，请将它们包装在一个布局面板中，例如 `StackPanel` 或 `Grid`：

```xml
<ContentControl>
  <StackPanel>
    <TextBlock Text="Line one" />
    <TextBlock Text="Line two" />
  </StackPanel>
</ContentControl>
```

### 使用模板显示内容

当你将 `ContentControl` 与数据绑定和数据模板结合使用时，它会变得特别有用。通过设置 `ContentTemplate` 属性，你可以控制绑定数据对象的可视化渲染方式。例如，假设有如下视图模型：

```csharp
namespace Example
{
    public class MainWindowViewModel : ViewModelBase
    {
        object content = new Student("Jane", "Deer");

        public object Content
        {
            get => content;
            set => this.RaiseAndSetIfChanged(ref content, value);
        }
    }

    public class Student
    {
        public Student(string firstName, string lastName)
        {
            FirstName = firstName;
            LastName = lastName;
        }

        public string FirstName { get; }
        public string LastName { get; }
    }
}
```

> 注意：下面的示例假定窗口的 `DataContext` 已被设置为 `MainWindowViewModel` 的一个实例。更多信息请参阅 [DataContext 相关章节](/docs/data-binding/data-context)。

你可以通过 `ContentTemplate` 属性，在 `ContentControl` 中显示学生的名和姓：

```xml
<Window xmlns="https://github.com/avaloniaui">
  <ContentControl Content="{Binding Content}">
    <ContentControl.ContentTemplate>
      <DataTemplate>
        <Grid ColumnDefinitions="Auto,Auto" RowDefinitions="Auto,Auto">
          <TextBlock Grid.Row="0" Grid.Column="0">First Name:</TextBlock>
          <TextBlock Grid.Row="0" Grid.Column="1" Text="{Binding FirstName}"/>
          <TextBlock Grid.Row="1" Grid.Column="0">Last Name:</TextBlock>
          <TextBlock Grid.Row="1" Grid.Column="1" Text="{Binding LastName}"/>
        </Grid>
      </DataTemplate>
    </ContentControl.ContentTemplate>
  </ContentControl>
</Window>
```

<Image light={ControlContentStudentScreenshot} alt="Student first and last name" position="center" maxWidth={400} cornerRadius="true" />

更多信息请参阅 [data templates](/docs/data-templates/introduction-to-data-templates) 页面。

### 动态切换内容

由于 `Content` 是一个可绑定属性，因此你可以在运行时切换 `ContentControl` 显示的内容。这种模式常用于基于视图的导航场景：你可以将某个视图模型绑定到 `Content`，再使用数据模板（或 `ViewLocator`）解析出对应的视图：

```xml
<ContentControl Content="{Binding CurrentPage}">
  <ContentControl.DataTemplates>
    <DataTemplate DataType="vm:HomeViewModel">
      <views:HomeView />
    </DataTemplate>
    <DataTemplate DataType="vm:SettingsViewModel">
      <views:SettingsView />
    </DataTemplate>
  </ContentControl.DataTemplates>
</ContentControl>
```

当你的视图模型将 `CurrentPage` 从 `HomeViewModel` 改为 `SettingsViewModel` 时，`ContentControl` 会自动渲染匹配的视图。

如果你希望内容切换时带有动画过渡效果，可以考虑改用 [`TransitioningContentControl`](/controls/data-display/transitioningcontentcontrol)。

## 另请参阅

- [ContentControl API reference](/api/avalonia/controls/contentcontrol)
- [`ContentControl.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/ContentControl.cs)
- [Data templates](/docs/data-templates/introduction-to-data-templates)
- [`TransitioningContentControl`](/controls/data-display/transitioningcontentcontrol)
- [Data binding](/docs/data-binding/data-context)
