---
id: reusing-data-templates
title: 复用数据模板
description: 通过在 Application.DataTemplates 集合中定义模板，在多个窗口之间共享数据模板。
doc-type: explanation
---

import DataTemplatesScopeScreenshot from '/img/guides/data/data-templates/datatemplates-scope.png';

如果你像上一页那样在 `Window.DataTemplates` 集合中定义了数据模板，那么它可以在当前窗口中的任意位置复用。不过，你也可以进一步将数据模板的复用范围扩展到应用中的任意窗口。

之所以可以这样做，是因为 _Avalonia UI_ 会沿逻辑树执行分层搜索来选择数据模板。最完整的搜索过程会从某个控件开始，递归地向上查找父控件，然后查看窗口级别的模板集合（如上一页所示），最后再查看应用本身的数据模板集合。

:::info
有关 _Avalonia UI_ 中逻辑树概念的更多信息，请参阅 [UI Composition](/docs/fundamentals/ui-composition)。
:::

因此，如果你希望在应用的任意窗口中复用某个模板，请将模板定义在 `app.axaml` 文件中的 `Application.DataTemplates` 集合里。

为了了解它的工作方式，先按下面这样添加另一个视图模型：

```csharp
namespace MySample
{
    public class Teacher
    {
        public string Name { get; set; } = String.Empty;
        public string Subject { get; set; } = String.Empty;
    }
}
```

然后在 `app.axaml` 文件中，为 `Teacher` 类型添加一个数据模板：

```xml
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:vm="using:MySample"
             x:Class="MySample.App"
             RequestedThemeVariant="Light">
    <Application.Styles>
        <FluentTheme />
    </Application.Styles>

  <Application.DataTemplates>
    <DataTemplate DataType="{x:Type vm:Teacher}">
      <Grid ColumnDefinitions="Auto,Auto" RowDefinitions="Auto,Auto">
        <TextBlock Grid.Row="0" Grid.Column="0">Name:</TextBlock>
        <TextBlock Grid.Row="0" Grid.Column="1" Text="{Binding Name}"/>
        <TextBlock Grid.Row="1" Grid.Column="0">Subject:</TextBlock>
        <TextBlock Grid.Row="1" Grid.Column="1" Text="{Binding Subject}"/>
      </Grid>
    </DataTemplate>
  </Application.DataTemplates>
</Application>
```

接着在窗口内容区域中使用一个本地定义的 teacher 对象：

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
  
  <local:Teacher Name="Dr Jones" Subject="Maths"/>
</Window>
```

尽管窗口中并没有为 teacher 定义数据模板，Avalonia UI 仍会找到你在应用级别定义的模板，并按预期显示：

<Image light={DataTemplatesScopeScreenshot} alt="窗口使用应用级数据模板显示教师姓名和学科" position="center" maxWidth={400} cornerRadius="true"/>

:::caution
请记得在每一个数据模板中都指定 `DataType`，无论它定义在哪里。因为如果 _Avalonia UI_ 无法为你的数据找到匹配的数据模板，就不会显示任何内容！
:::

## 另请参阅

- [数据模板简介](/docs/data-templates/introduction-to-data-templates)：Avalonia 中数据模板的概览。
- [数据模板集合](/docs/data-templates/data-template-collection)：按类型定义多个模板。
- [在代码中创建数据模板](/docs/data-templates/creating-data-templates-in-code)：实现 `IDataTemplate` 并使用 `FuncDataTemplate<T>`。
