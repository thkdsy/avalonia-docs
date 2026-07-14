---
id: data-template-collection
title: 数据模板集合
description: 在控件的 DataTemplates 集合中定义多个数据模板，并按类型进行匹配。
doc-type: explanation
---

import DataTemplatesCollectionStudentScreenshot from '/img/concepts/data-concepts/data-templates/data-template-collection/datatemplates-collection-student.png';

_Avalonia UI_ 中的每个控件都有一个 [`DataTemplates`](/api/avalonia/controls/templates/datatemplates) 集合，你可以在其中放置任意数量的数据模板定义。随后便可根据类类型选择用于显示的模板。

当控件没有像前一页那样直接在 `ContentTemplate` 属性中设置数据模板时，它会从自身的 `DataTemplates` 集合中选择一个与当前显示对象类型匹配的模板。窗口同样适用这一规则。

数据模板按类型匹配：当被显示对象的类，与模板 `DataType` 属性中指定的完整限定类名一致时，就会发生匹配。

因此，你可以将前面的示例修改为使用 `DataTemplates` 集合，如下所示：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:local="using:MySample"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
        x:Class="MySample.MainWindow"
        Title="MySample">
  <Window.DataTemplates>
    <DataTemplate DataType="{x:Type local:Student}">
      <Grid ColumnDefinitions="Auto,Auto" RowDefinitions="Auto,Auto">
        <TextBlock Grid.Row="0" Grid.Column="0">First Name:</TextBlock>
        <TextBlock Grid.Row="0" Grid.Column="1" Text="{Binding FirstName}"/>
        <TextBlock Grid.Row="1" Grid.Column="0">Last Name:</TextBlock>
        <TextBlock Grid.Row="1" Grid.Column="1" Text="{Binding LastName}"/>
      </Grid>
    </DataTemplate>
  </Window.DataTemplates>
  
  <local:Student FirstName="Jane" LastName="Deer"/>
</Window>
```

这样得到的显示结果与上一页完全相同：

<Image light={DataTemplatesCollectionStudentScreenshot} alt="Window displaying student first and last name using a data template from the DataTemplates collection" position="center" maxWidth={400} cornerRadius="true"/>

## 按类型定义多个数据模板

`DataTemplates` 集合可以针对不同类型选择不同模板。当 Avalonia 遇到一个对象时，它会在 `DataTemplates` 集合中查找 `DataType` 与该对象类型匹配的模板：

```xml
<Window.DataTemplates>
    <DataTemplate DataType="{x:Type local:Student}">
        <StackPanel Orientation="Horizontal" Spacing="8">
            <TextBlock Text="🎓" />
            <TextBlock Text="{Binding FirstName}" />
            <TextBlock Text="{Binding LastName}" />
        </StackPanel>
    </DataTemplate>

    <DataTemplate DataType="{x:Type local:Teacher}">
        <StackPanel Orientation="Horizontal" Spacing="8">
            <TextBlock Text="📚" />
            <TextBlock Text="{Binding Name}" FontWeight="Bold" />
            <TextBlock Text="{Binding Subject}" Foreground="Gray" />
        </StackPanel>
    </DataTemplate>
</Window.DataTemplates>
```

定义完这些模板后，显示 `Student` 对象的 `ListBox` 或 `ContentControl` 会使用第一个模板，而显示 `Teacher` 对象时则会使用第二个模板：

```xml
<ListBox ItemsSource="{Binding People}" />
```

## 模板搜索顺序

当 Avalonia 需要为某个对象查找数据模板时，会按以下顺序进行搜索：

1. 控件自身的 `DataTemplates` 集合。
2. 沿树向上遍历每个父控件的 `DataTemplates` 集合。
3. `Window.DataTemplates` 集合。
4. `Application.DataTemplates` 集合。

第一个匹配成功的模板会被使用。这使你能够在树中的任意层级覆盖应用级别的模板。

## 另请参阅

- [数据模板简介](/docs/data-templates/introduction-to-data-templates)：Avalonia 中数据模板的概览。
- [内容模板](/docs/data-templates/content-templates)：直接使用 `ContentTemplate`。
- [复用数据模板](/docs/data-templates/reusing-data-templates)：在整个应用中共享模板。
