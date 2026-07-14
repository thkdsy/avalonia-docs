---
id: creating-data-templates-in-code
title: 在代码中创建数据模板
description: 在 C# 中使用 FuncDataTemplate 或通过实现 IDataTemplate 来创建数据模板。
doc-type: how-to
---

## `FuncDataTemplate`

_Avalonia UI_ 支持在代码中创建数据模板。你可以使用支持 [`IDataTemplate`](/api/avalonia/controls/templates/idatatemplate) 接口的 `FuncDataTemplate<T>` 类来实现这一点。

最简单的方式，是向 `FuncDataTemplate<T>` 构造函数传入一个用于创建控件的 lambda 函数，例如：

```csharp
var template = new FuncDataTemplate<Student>((value, namescope) =>
    new TextBlock
    {
        [!TextBlock.TextProperty] = new ReflectionBinding("FirstName"),
    });
```

这等价于下面的 XAML：

```xml
<DataTemplate DataType="{x:Type local:Student}">
    <TextBlock Text="{Binding FirstName}"/>
</DataTemplate>
```

## 在代码中进行更精细的控制

如果你需要在代码中更细致地控制数据模板，可以自己编写一个实现 `IDataTemplate` 接口的类。这样你就能按照自己的需要展示绑定数据类型的属性。

要使用 `IDataTemplate` 接口，你必须在数据模板类中实现以下两个成员：

* `public bool Match(object data) { ... }` - 实现该成员以检查传入的绑定数据是否与当前 `IDataTemplate` 匹配。若绑定数据类型匹配则返回 true，否则返回 false。
* `public Control Build(object param) { ... }` - 实现该成员以构建并返回用于呈现数据的控件。

## 示例

下面是一个简单的 `IDataTemplate` 接口实现，它会将字符串数据显示在文本块中：

```csharp
using Avalonia.Controls.Templates;
...
public class MyDataTemplate : IDataTemplate
{
    public Control Build(object param)
    {
        return new TextBlock() { Text = (string)param };
    }

    public bool Match(object data)
    {
        return data is string;
    }
}
```

现在你可以在视图中像下面这样使用 `MyDataTemplate` 类：

```xml
<!-- xmlns:dataTemplates="using:MyApp.DataTemplates" -->

<ContentControl Content="{Binding MyContent}">
	<ContentControl.ContentTemplate>
		<dataTemplates:MyDataTemplate />
	</ContentControl.ContentTemplate>
</ContentControl>
```

## 更多示例

[`FuncDataTemplate<T>` 类的高级用法](https://github.com/AvaloniaUI/Avalonia.Samples/blob/main/src/Avalonia.Samples/DataTemplates/FuncDataTemplateSample)。

[`IDataTemplate` 接口的高级实现](https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/DataTemplates/IDataTemplateSample)。

## 另请参阅

- [数据模板简介](/docs/data-templates/introduction-to-data-templates)：Avalonia 中数据模板的概览。
- [数据模板集合](/docs/data-templates/data-template-collection)：按类型定义多个模板。
- [视图定位器](/docs/data-templates/view-locator)：为视图模型自动解析视图。
