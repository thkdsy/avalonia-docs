---
id: container-queries
title: 容器查询
---

容器查询允许你根据某个祖先元素（作为容器）的尺寸，为控件激活对应的样式。

:::tip
Avalonia 的容器查询与 CSS 容器查询类似，但功能更精简，以适配 Avalonia 所支持的平台与设备形态。如果将 `TopLevel` 设置为容器，它们也可以表现得像媒体查询。
:::

## 工作原理

容器查询依赖于某个祖先控件被设置为容器。容器尺寸发生变化时，会根据查询条件激活相应样式。查询可以检查容器的宽度、高度，或同时检查两者。任何控件都可以作为容器，但被设置为容器的控件本身，不能受到与其关联的容器查询中所承载样式的影响。当某个查询被激活时，其中包含的所有样式也会根据各自的选择器一并激活。

## 如何使用查询

### 声明容器查询
容器查询可以在 XAML 中定义为某个控件 `Styles` 属性的直接子元素，如下所示：

```xml
<StackPanel Orientation="Horizontal">
  <StackPanel.Styles>
    <ContainerQuery Name="container"
                    Query="max-width:400">
      <Style Selector="Button">
        <Setter Property="Background"
                Value="Red"/>
      </Style>
    </ContainerQuery>
  </StackPanel.Styles>
</StackPanel>
```

它们也可以作为 `ControlTheme` 样式的一部分：

```xml
<ControlTheme x:Key="{x:Type ListBox}" TargetType="ListBox">
    ...
  <Setter Property="Template">
    <ControlTemplate>
      <Border Name="border"
              Container.Name="Test"
              Container.Sizing="WidthAndHeight"
              >
        <ScrollViewer Name="PART_ScrollViewer">
            ...
        </ScrollViewer>
      </Border>
    </ControlTemplate>
  </Setter>


  <ContainerQuery Name="Test"
                  Query="max-height:400">
    <Style Selector="ScrollViewer#PART_ScrollViewer">
      <Setter Property="Background"
              Value="Red"/>
    </Style>
  </ContainerQuery>
</ControlTheme>
```
`Name` 属性定义了它要附加到的容器名称。它不是唯一标识符，多个容器查询可以使用同一个名称。
`Query` 定义了触发该容器尺寸条件的规则。详见下方的 [查询](#查询)。

这使得容器查询非常适合用于面向不同屏幕尺寸的主题，或根据父级可用空间切换不同布局形式的主题。不过它也有一些限制。

1. 容器查询不能放在 `Style` 元素中。
   下列写法是无效的。

```xml
<StackPanel Orientation="Horizontal">
  <StackPanel.Styles>
    <Style Selector="...">
      <ContainerQuery Name="container"
                      Query="max-width:400">
        <Style Selector="Button">
          <Setter Property="Background"
                  Value="Red"/>
        </Style>
      </ContainerQuery>
    </Style>
  </StackPanel.Styles>
</StackPanel>
```

2. 在 `ContainerQuery` 中声明的样式不能影响该容器本身或它的祖先。这与普通 `Styles` 可以影响其父控件不同。因为容器查询依赖于容器的实际尺寸，如果容器又被它自己的查询激活的样式所影响，就可能产生循环行为，导致两个或更多查询不断更新容器尺寸。

### 声明容器
只有当某个控件既是 `ContainerQuery` 所在宿主的后代，又被声明为容器时，容器查询才会生效。为任意控件设置附加属性 `Container.Name` 和 `Container.Sizing`，就可以将该控件声明为容器，例如：

```xml
<Button
  Container.Name="container-name"
  Container.Sizing="container-sizing"
/>
```

`Container.Name` 定义容器的名称。这个名称对容器来说不是唯一的，同一作用域中的多个控件可以拥有相同的容器名称，并且它们都会受到同一组容器查询的影响。

`Container.Sizing` 定义容器用于查询时的尺寸策略。容器的最终尺寸取决于这个值。它是一个枚举，包含以下取值：

* `Normal`：不查询容器尺寸。这是默认值。控件按照正常的测量与排列流程工作。
* `Width`：查询容器宽度。容器会使用其父级允许的最大宽度，并将该值用于所有相关的容器查询。在大多数情况下，最终宽度就是允许的最大宽度。
* `Height`：与 `Width` 类似，但只查询容器高度。
* `WidthAndHeight`：同时查询容器的宽度和高度。

根据尺寸策略的不同，容器会将最大可用尺寸作为自己的期望尺寸。

### 查询
可用的查询如下。

* `min-width`：等价于 `x >= width`
* `min-height`：等价于 `x >= height`
* `max-width`：等价于 `x <= width`
* `max-height`：等价于 `x <= height`
* `height`：等价于 `x == height`
* `width`：等价于 `x == width`

下面是一个使用多个不同查询条件的容器查询示例：

```xml
<ContainerQuery Name="uniformGrid"
                Query="max-width:400">
  <Style Selector="UniformGrid#ContentGrid">
    <Setter Property="Columns"
            Value="1"/>
  </Style>
</ContainerQuery>
<ContainerQuery Name="uniformGrid"
                Query="min-width:400">
  <Style Selector="UniformGrid#ContentGrid">
    <Setter Property="Columns"
            Value="2"/>
  </Style>
</ContainerQuery>
<ContainerQuery Name="uniformGrid"
                Query="min-width:800">
  <Style Selector="UniformGrid#ContentGrid">
    <Setter Property="Columns"
            Value="3"/>
  </Style>
</ContainerQuery>
```
多个查询可以使用 `,` 进行 OR 组合，或使用 `and` 进行 AND 组合。

```xml
<ContainerQuery Name="uniformGrid"
                Query="max-width:400,min-width:300">
  <Style Selector="UniformGrid#ContentGrid">
    <Setter Property="Columns"
            Value="1"/>
  </Style>
</ContainerQuery>
<ContainerQuery Name="uniformGrid"
                Query="min-width:400 and min-width:300">
  <Style Selector="UniformGrid#ContentGrid">
    <Setter Property="Columns"
            Value="2"/>
  </Style>
</ContainerQuery>
```

这样你就可以针对尺寸区间编写查询。

## 另请参阅

- [Responsive layouts](/docs/layout/responsive-layouts)：使用容器查询构建自适应布局。
- [How to: Build responsive layouts](/docs/how-to/responsive-layout-how-to)：常见响应式模式的分步示例。
- [Styles](/docs/styling/styles)
- [Control themes](/docs/styling/control-themes)
