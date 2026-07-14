---
id: control-content
title: 控件内容
description: 了解控件如何显示非控件内容，以及为什么需要数据模板。
doc-type: explanation
---

import ControlContentButtonScreenshot from '/img/concepts/data-concepts/data-templates/control-content/content-button.png';
import ControlContentStringScreenshot from '/img/concepts/data-concepts/data-templates/control-content/content-string.png';
import ControlContentTypeScreenshot from '/img/concepts/data-concepts/data-templates/control-content/content-type.png';

你大概已经见过把一个按钮控件放进 _Avalonia UI_ 窗口内容区域后会发生什么。

:::info
有关 _Avalonia UI_ 控件中不同区域的更多说明，请参阅[布局](/docs/layout/)。
:::

例如：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
        x:Class="MySample.MainWindow"
        Title="MySample">
  <Button HorizontalAlignment="Center" >Hello World!</Button>
</Window>
```

窗口会显示这个按钮——在这里它会水平居中（显式指定）并垂直居中（默认行为）。效果如下：

<Image light={ControlContentButtonScreenshot} alt="Window displaying a centered Hello World button" position="center" maxWidth={400} cornerRadius="true"/>

如果你把一个字符串放进窗口内容区域，例如：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
        x:Class="MySample.MainWindow"
        Title="MySample">
  Hello World!
</Window>
```

窗口会显示这个字符串：

<Image light={ControlContentStringScreenshot} alt="Window displaying a Hello World string" position="center" maxWidth={400} cornerRadius="true"/>

但如果你尝试在窗口中显示一个由你自己定义的类实例，会发生什么呢？

例如，使用 `Student` 这个类定义：

```csharp
namespace MySample
{
    public class Student
    {
        public string FirstName { get; set;} = String.Empty;
        public string LastName { get; set;} = String.Empty;
    }
}
```

并将 XML 命名空间 `local` 定义为前面提到的 `MySample` 命名空间后，你可以像下面这样在窗口内容区域中定义一个学生对象：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="using:MySample"
        x:Class="MySample.Views.MainWindow">
  <local:Student FirstName="Jane" LastName="Deer"/>
</Window>
```

但你最终只会看到该学生对象的完整限定类名：

<Image light={ControlContentTypeScreenshot} alt="窗口显示 Student 对象的完整限定类名" position="center" maxWidth={400} cornerRadius="true"/>

这显然没什么帮助！之所以会这样，是因为 _Avalonia UI_ 并不知道该如何显示 `Student` 类的对象——它本身也不是控件——因此会退回到调用 `.ToString()` 方法，最终你看到的就只是完整限定类名。

## 另请参阅

- [内容模板](/docs/data-templates/content-templates)：使用 `ContentTemplate` 定义数据如何显示。
- [数据模板集合](/docs/data-templates/data-template-collection)：按类型定义多个模板。
- [数据模板简介](/docs/data-templates/introduction-to-data-templates)：Avalonia 中数据模板的概览。
