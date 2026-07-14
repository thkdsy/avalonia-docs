---
id: binding-classes
title: 如何绑定样式类
description: 将样式类绑定到布尔属性上，以便按条件为 Avalonia 控件应用样式。
doc-type: how-to
---

import BindStyleClassSampleScreenshot from '/img/guides/data/bind-style-class.png';

本指南演示如何根据数据绑定的布尔值，为控件应用不同的样式类。

要做到这一点，首先需要在 `<Styles>` 集合中定义样式类，并让它们针对你正在使用的控件类型。

之后，你就可以通过特殊的 `Classes.` 语法配合数据绑定，有条件地把这些样式类应用到控件上。语法如下：

```xml
<SomeControl Classes.myClass="{Binding IsMyClassActive}">
```

### 绑定多个样式类

你可以把多个样式类绑定到同一个控件上。每个样式类绑定彼此独立，因此可以自由组合：

```xml
<TextBlock Classes.error="{Binding HasError}"
           Classes.highlight="{Binding IsHighlighted}"
           Classes.large="{Binding IsLarge}" />
```

### 取反运算符

你可以在绑定表达式中使用取反运算符（`!`），从而在某个布尔属性为 `false` 时应用某个样式类。当你想在两个互斥样式类之间切换、又不想额外新增一个视图模型属性时，这会非常方便：

```xml
<TextBlock Classes.classA="{Binding IsOptionA}"
           Classes.classB="{Binding !IsOptionA}" />
```

在这个例子中，当 `IsOptionA` 为 `true` 时，会应用 `classA`；当 `IsOptionA` 为 `false` 时，会应用 `classB`。

## 示例

在这个示例中，定义了两个带类选择器的样式，它们会让 `TextBlock` 呈现红色或绿色背景。`Classes.class1` 会在某个项的 `IsClass1` 属性为 `true` 时应用 `class1`；借助取反运算符，当 `IsClass1` 为 `false` 时，则会应用 `class2`。

```xml
<StackPanel Margin="20">
  <ListBox ItemsSource="{Binding ItemList}">
    <ListBox.Styles>
      <Style Selector="TextBlock.class1">
        <Setter Property="Background" Value="OrangeRed" />
      </Style>
      <Style Selector="TextBlock.class2">
        <Setter Property="Background" Value="PaleGreen" />
      </Style>
    </ListBox.Styles>
    <ListBox.ItemTemplate>
      <DataTemplate>
        <StackPanel>
          <TextBlock
              Classes.class1="{Binding IsClass1}"
              Classes.class2="{Binding !IsClass1}"
              Text="{Binding Title}"/>
        </StackPanel>
      </DataTemplate>
    </ListBox.ItemTemplate>
  </ListBox>
</StackPanel>
```

```csharp title='MainWindowViewModel.cs'
public class MainWindowViewModel : ViewModelBase
{
    public ObservableCollection<ItemClass> ItemList { get; set; }

    public MainWindowViewModel()
    {
        ItemList = new ObservableCollection<ItemClass>(new List<ItemClass>
        {
            new ItemClass("Item 1", false),
            new ItemClass("Item Two", false),
            new ItemClass("Third Item", true),
            new ItemClass("Item #4", false),
        });
    }
}
```

```csharp title='ItemClass.cs'
public class ItemClass
{
    public string Title { get; set; }
    public bool IsClass1 { get; set; }

    public ItemClass(string title, bool isClass1)
    {
        Title = title;
        IsClass1 = isClass1;
    }
}
```

<Image light={BindStyleClassSampleScreenshot} alt="Sample app showing style classes toggled by data binding" position="center" maxWidth={400} cornerRadius="true"/>

## 另请参阅

- [Styles](/docs/styling/styles)
- [Data binding syntax](/docs/data-binding/data-binding-syntax)
