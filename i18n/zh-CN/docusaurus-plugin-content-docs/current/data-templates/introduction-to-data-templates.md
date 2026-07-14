---
id: introduction-to-data-templates
title: 数据模板简介
description: 使用数据模板及其类型匹配与复用机制，定义 Avalonia 如何显示数据对象。
doc-type: overview
---

Avalonia 中的数据模板定义了数据的视觉呈现方式。它们决定数据对象在 UI 中如何被展示和格式化。本页将介绍数据模板的基本概念，并说明如何在应用中使用它们。

## 什么是数据模板？

从本质上说，数据模板是一种可复用定义，用于指定某种特定类型的数据应如何被呈现。它定义了数据在用户界面中显示时所采用的视觉结构和外观。在 Avalonia 中，数据模板通常与列表控件相关联，例如 [`ListBox`](/api/avalonia/controls/listbox) 或 `ItemsControl`，并负责渲染该控件中的各个数据项。

## 将数据模板应用到 ListBox

要将数据模板应用到 `ListBox`，通常需要使用控件的 `ItemTemplate` 属性。

例如，如果你有一个 `ListBox`，希望它使用已定义的数据模板来显示一组 `Item` 对象，那么可以像下面这样设置 `ItemTemplate` 属性：

```xml
<ListBox ItemsSource="{Binding Items}">
  <ListBox.ItemTemplate>
    <DataTemplate>
        <StackPanel Orientation="Horizontal">
            <TextBlock Text="{Binding Name}" />
            <Image Source="{Binding ImageSource}" />
        </StackPanel>
    </DataTemplate>
  </ListBox.ItemTemplate>
</ListBox>
```

在这个示例中，数据模板使用 `StackPanel` 容器定义了一个视觉布局。在 `StackPanel` 内部，`TextBlock` 绑定到数据项的 `Name` 属性，而 `Image` 控件则绑定到 `ImageSource` 属性。

## 类型专属数据模板

你可以使用 `DataType`，根据正在显示对象的类型自动选择对应模板：

```xml
<Window.DataTemplates>
    <DataTemplate DataType="{x:Type local:Customer}">
        <StackPanel Orientation="Horizontal" Spacing="8">
            <TextBlock Text="{Binding Name}" FontWeight="Bold" />
            <TextBlock Text="{Binding Email}" Foreground="Gray" />
        </StackPanel>
    </DataTemplate>

    <DataTemplate DataType="{x:Type local:Product}">
        <StackPanel Orientation="Horizontal" Spacing="8">
            <TextBlock Text="{Binding ProductName}" />
            <TextBlock Text="{Binding Price, StringFormat='${0:F2}'}" />
        </StackPanel>
    </DataTemplate>
</Window.DataTemplates>
```

当 Avalonia 在某个内容区域中遇到一个对象时，它会根据类型去搜索匹配的 `DataTemplate`。搜索会从当前控件开始，并沿树向上查找，直到找到匹配项为止。

## 数据模板可以定义在哪里

| 位置 | 作用范围 |
|---|---|
| `Control.DataTemplates` | 对该控件及其子元素可用。 |
| `Window.DataTemplates` | 对整个窗口可用。 |
| `Application.DataTemplates` | 对整个应用可用。 |
| `ContentTemplate` 属性 | 直接应用到某个特定的 `ContentControl`。 |
| `ItemTemplate` 属性 | 应用于列表或集合控件中的每一项。 |

## 资源中的数据模板

你可以把可复用模板定义为资源：

```xml
<Application.Resources>
    <DataTemplate x:Key="CustomerTemplate" DataType="{x:Type local:Customer}">
        <TextBlock Text="{Binding Name}" />
    </DataTemplate>
</Application.Resources>
```

然后再引用它：

```xml
<ContentControl Content="{Binding SelectedCustomer}"
                ContentTemplate="{StaticResource CustomerTemplate}" />
```

## 另请参阅

- [Control Content](/docs/data-templates/control-content): How controls display non-control content.
- [Content Templates](/docs/data-templates/content-templates): Using `ContentTemplate` directly.
- [Data Template Collection](/docs/data-templates/data-template-collection): Defining multiple templates by type.
- [Reusing Data Templates](/docs/data-templates/reusing-data-templates): Sharing templates across your application.
