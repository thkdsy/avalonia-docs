---
id: content-templates
title: 内容模板
description: 使用 ContentTemplate 属性定义控件如何显示数据对象。
doc-type: explanation
---

import ContentTemplateStudentScreenshot from '/img/concepts/data-concepts/data-templates/content-templates/contenttemplate-student.png';

数据模板的作用，是定义 _Avalonia UI_ 应如何显示一个由你自定义类创建的对象，前提是这个对象不是控件，也不是简单字符串。

使用数据模板通常分为两个阶段：

1. 定义数据模板
2. 为内容选择数据模板

使用数据模板的一种方式，是直接设置控件的 `ContentTemplate` 属性。这种方式同样适用于窗口，因为窗口和其他相关控件一样继承自 `ContentControl`。

你可以使用 `DataTemplate` 标记、内置控件组合以及若干绑定来定义一个数据模板（不局限于某个特定类）。例如：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:local="using:MySample"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
        x:Class="MySample.MainWindow"
        Title="MySample">
  <Window.ContentTemplate>
    <DataTemplate DataType="{x:Type local:Student}">
      <StackPanel>
        <Grid ColumnDefinitions="Auto,Auto" RowDefinitions="Auto,Auto">
          <TextBlock Grid.Row="0" Grid.Column="0">First Name:</TextBlock>
          <TextBlock Grid.Row="0" Grid.Column="1" Text="{Binding FirstName}"/>
          <TextBlock Grid.Row="1" Grid.Column="0">Last Name:</TextBlock>
          <TextBlock Grid.Row="1" Grid.Column="1" Text="{Binding LastName}"/>
        </Grid>
      </StackPanel>
    </DataTemplate>
  </Window.ContentTemplate>
  
  <local:Student FirstName="Jane" LastName="Deer"/>
</Window>
```

在上面的示例中，绑定指向窗口内容区域内对象的属性。这里窗口内容仍然是前面使用过的学生对象；但当你运行这段代码时，_Avalonia UI_ 现在会显示：

<Image light={ContentTemplateStudentScreenshot} alt="Window displaying student first and last name using a content template" position="center" maxWidth={400} cornerRadius="true"/>

通过这种方式使用数据模板时，你在同一个位置同时完成了“定义模板”和“为内容选择模板”这两件事，也就是直接设置窗口的 `ContentTemplate` 属性。

这段代码能够正常工作，是因为窗口内容区域中的对象正好具有绑定中指定的那些属性。你可以自行尝试：添加一个绑定到学生类中不存在的属性。（应用依然可以运行，只是会忽略找不到的属性。）

在下一页中，你将看到如何定义多个数据模板，并根据窗口内容区域中对象的类型选择正确的模板。

## 另请参阅

- [控件内容](/docs/data-templates/control-content)：控件如何显示非控件内容。
- [数据模板集合](/docs/data-templates/data-template-collection)：按类型定义多个模板。
- [在代码中创建数据模板](/docs/data-templates/creating-data-templates-in-code)：实现 `IDataTemplate` 并使用 `FuncDataTemplate<T>`。
