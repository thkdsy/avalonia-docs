---
id: data-templates
title: 数据模板
description: WPF 与 Avalonia 在数据模板、类型匹配和模板存储方式上的差异。
doc-type: migration
---

Avalonia 中的数据模板与 WPF 的工作方式相似，都允许你为数据对象定义可视化表示。核心概念基本一致，但在模板存储位置、类型匹配方式以及可用扩展能力方面，仍然存在一些关键差异。

## 模板存储方式

在 WPF 中，数据模板通常存储在 `ResourceDictionary` 中，可以放在控件、窗口或 `App.xaml` 上：

```xml
<!-- WPF -->
<Window.Resources>
    <DataTemplate DataType="{x:Type viewmodels:FooViewModel}">
        <TextBlock Text="{Binding Name}" />
    </DataTemplate>
</Window.Resources>
```

在 Avalonia 中，数据模板并不存储在资源中。相反，它们被放在 [`DataTemplates`](/api/avalonia/controls/templates/datatemplates) 集合中，而这个集合存在于每个 `Control` 和 `Application` 上：

```xml
<!-- Avalonia -->
<Window xmlns:viewmodels="using:MyApp.ViewModels">
    <Window.DataTemplates>
        <DataTemplate DataType="viewmodels:FooViewModel">
            <TextBlock Text="{Binding Name}" />
        </DataTemplate>
    </Window.DataTemplates>
</Window>
```

模板解析会沿着可视树向上查找，依次检查每个控件的 `DataTemplates` 集合，最后再回退到 `Application.DataTemplates`。这与 WPF 的资源查找机制相似，但使用的是专用集合，而不是通用资源字典。

## DataType 匹配

两个框架都支持通过 `DataType` 匹配模板。不过，Avalonia 还提供了一些 WPF 不具备的能力：

- **接口匹配：** Avalonia 可以让 `DataType` 与某个接口匹配；WPF 只能匹配具体类型。
- **派生类匹配：** Avalonia 会把模板匹配到指定 `DataType` 的派生类；而 WPF 通常要求精确类型匹配。

正因为这些匹配规则更宽松，模板在集合中的顺序就变得很重要。模板会按声明顺序依次评估，因此应将更具体的模板放在更宽泛的模板之前：

```xml
<Window.DataTemplates>
    <!-- 最具体的类型放在前面 -->
    <DataTemplate DataType="viewmodels:SpecialItemViewModel">
        <Border Background="Gold">
            <TextBlock Text="{Binding Name}" />
        </Border>
    </DataTemplate>
    <!-- 基类或接口模板放在后面 -->
    <DataTemplate DataType="viewmodels:ItemViewModel">
        <TextBlock Text="{Binding Name}" />
    </DataTemplate>
</Window.DataTemplates>
```

请注意，在 WPF 中，`DataType` 通常使用 `{x:Type}` 标记扩展；而在 Avalonia 中，你会直接以字符串形式指定类型。

## DataTemplateSelector 的替代方式

在 WPF 中，你可以创建一个 `DataTemplateSelector` 子类，根据自定义逻辑选择模板：

```csharp
// WPF
public class MyTemplateSelector : DataTemplateSelector
{
    public DataTemplate TemplateA { get; set; }
    public DataTemplate TemplateB { get; set; }

    public override DataTemplate SelectTemplate(object item, DependencyObject container)
    {
        return item is SpecialItem ? TemplateA : TemplateB;
    }
}
```

Avalonia 没有 `DataTemplateSelector`。取而代之的是实现 `IDataTemplate` 接口，它承担相同职责：

```csharp
// Avalonia
public class MyDataTemplate : IDataTemplate
{
    public Control? Build(object? data)
    {
        if (data is SpecialItem)
            return new Border { Background = Brushes.Gold, Child = new TextBlock { Text = "Special" } };

        return new TextBlock { [!TextBlock.TextProperty] = new ReflectionBinding("Name") };
    }

    public bool Match(object? data)
    {
        return data is ItemViewModel;
    }
}
```

随后你就可以在 XAML 中直接使用这个自定义模板：

```xml
<Window.DataTemplates>
    <local:MyDataTemplate />
</Window.DataTemplates>
```

如需完整可运行示例，请参阅 [IDataTemplate 示例](https://github.com/AvaloniaUI/Avalonia.Samples/tree/main/src/Avalonia.Samples/DataTemplates/IDataTemplateSample)。

## TreeDataTemplate

WPF 中的 `HierarchicalDataTemplate` 在 Avalonia 中叫做 `TreeDataTemplate`。两者在功能上等价，差别主要只是名称不同。

**WPF：**

```xml
<!-- WPF -->
<TreeView ItemsSource="{Binding RootNodes}">
    <TreeView.Resources>
        <HierarchicalDataTemplate DataType="{x:Type viewmodels:NodeViewModel}"
                                  ItemsSource="{Binding Children}">
            <TextBlock Text="{Binding Title}" />
        </HierarchicalDataTemplate>
    </TreeView.Resources>
</TreeView>
```

**Avalonia：**

```xml
<!-- Avalonia -->
<TreeView ItemsSource="{Binding RootNodes}">
    <TreeView.DataTemplates>
        <TreeDataTemplate DataType="viewmodels:NodeViewModel"
                          ItemsSource="{Binding Children}">
            <TextBlock Text="{Binding Title}" />
        </TreeDataTemplate>
    </TreeView.DataTemplates>
</TreeView>
```

请注意，Avalonia 将模板放在 `DataTemplates` 中，而不是 `Resources` 中。

## ItemTemplate and ContentTemplate

`ItemsControl`、`ListBox` 及类似控件上的 `ItemTemplate` 属性，在两个框架中的工作方式基本相同。你需要为其指定一个 `DataTemplate`，以控制每个项目如何渲染：

```xml
<ListBox ItemsSource="{Binding Items}">
    <ListBox.ItemTemplate>
        <DataTemplate>
            <StackPanel Orientation="Horizontal" Spacing="8">
                <Image Source="{Binding Icon}" Width="16" Height="16" />
                <TextBlock Text="{Binding DisplayName}" />
            </StackPanel>
        </DataTemplate>
    </ListBox.ItemTemplate>
</ListBox>
```

同样，`ContentControl` 和 `ContentPresenter` 上的 `ContentTemplate` 也按预期工作。如果你没有显式设置 `ItemTemplate` 或 `ContentTemplate`，Avalonia 会沿着树向上查找 `DataTemplates` 集合中的匹配模板，这与 WPF 通过资源进行查找的思路类似。

## 用于编译绑定的 x:DataType

Avalonia 支持编译绑定，它能在编译期校验绑定路径，并提升运行时性能。WPF 中没有完全对应的特性。

如果要在数据模板中启用编译绑定，请将 `x:DataType` 属性设置为该模板将接收的数据类型：

```xml
<DataTemplate DataType="viewmodels:FooViewModel"
              x:DataType="viewmodels:FooViewModel">
    <StackPanel>
        <!-- 这些绑定会在编译期校验 -->
        <TextBlock Text="{Binding Name}" />
        <TextBlock Text="{Binding Description}" />
    </StackPanel>
</DataTemplate>
```

设置 `x:DataType` 后，编译器会检查 `Name` 和 `Description` 是否真实存在于 `FooViewModel` 上。拼写错误或错误的属性名会直接产生构建错误，而不是在运行时悄悄失败。

你还可以通过在 `.csproj` 文件中添加 `<AvaloniaUseCompiledBindingsByDefault>true</AvaloniaUseCompiledBindingsByDefault>` 来为整个项目启用默认编译绑定，这会让 `x:DataType` 成为所有绑定的默认预期。

## 另请参阅

- [数据模板简介](/docs/data-templates/introduction-to-data-templates)
- [数据模板集合](/docs/data-templates/data-template-collection)
- [编译绑定](/docs/data-binding/compiled-bindings)
