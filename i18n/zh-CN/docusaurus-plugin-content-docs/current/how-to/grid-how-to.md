---
id: grid-how-to
title: "如何：使用 Grid 布局"
description: 了解 Grid 的行列定义、尺寸模式、跨行跨列、共享尺寸以及响应式布局模式。
doc-type: how-to
---

本指南介绍常见的 [`Grid`](/api/avalonia/controls/grid) 布局场景，包括行列定义、尺寸模式、跨行跨列、共享尺寸以及响应式模式。

## 行和列定义

你可以使用简写语法定义行和列：

```xml
<Grid ColumnDefinitions="200,*,Auto" RowDefinitions="Auto,*,Auto">
    <TextBlock Grid.Row="0" Grid.Column="0" Text="Sidebar Header" />
    <ListBox Grid.Row="1" Grid.Column="0" />
    <ContentControl Grid.Row="0" Grid.RowSpan="3" Grid.Column="1"
                    Content="{Binding MainContent}" />
    <TextBlock Grid.Row="2" Grid.Column="0" Grid.ColumnSpan="2"
               Text="Status Bar" />
</Grid>
```

如果你需要对单个定义做更细粒度控制（例如设置 `MinWidth` 或 `MaxWidth`），则可以使用完整写法：

```xml
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="200" />
        <ColumnDefinition Width="*" />
        <ColumnDefinition Width="Auto" />
    </Grid.ColumnDefinitions>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto" />
        <RowDefinition Height="*" />
    </Grid.RowDefinitions>
</Grid>
```

:::tip
简写语法更紧凑，而完整写法则允许你为每个定义设置更多属性，例如 `MinWidth`、`MaxWidth`、`MinHeight`、`MaxHeight` 和 `SharedSizeGroup`。
:::

## 尺寸模式

### 像素尺寸

使用设备无关像素设置固定尺寸：

```xml
<Grid ColumnDefinitions="200,300">
    <!-- 第 0 列固定为 200px，第 1 列固定为 300px -->
</Grid>
```

当你希望某一列或某一行无论内容如何都保持固定大小时，应使用像素尺寸。这种方式很适合侧边栏、工具栏和图标列。

### Auto 尺寸

根据内容自动决定行或列的大小：

```xml
<Grid ColumnDefinitions="Auto,*">
    <!-- 第 0 列收缩到刚好包住内容 -->
    <TextBlock Grid.Column="0" Text="Label:" />
    <!-- 第 1 列填充剩余空间 -->
    <TextBox Grid.Column="1" Text="{Binding Value}" />
</Grid>
```

:::note
`Auto` 行或列会测量其所有子元素，并扩展到足以容纳其中最大的一个。如果内容是动态增长的（例如运行时加载的长文本），该列也会随之变宽，从而可能把其他列挤出可视区域。如果你希望限制其上限，请结合完整语法设置 `MaxWidth` 或 `MaxHeight`。
:::

### 星号尺寸

在 `Auto` 和固定像素列完成测量之后，按比例分配剩余空间：

```xml
<Grid ColumnDefinitions="*,2*,*">
    <!-- 第 0 列：剩余空间的 25%（1/4） -->
    <!-- 第 1 列：剩余空间的 50%（2/4） -->
    <!-- 第 2 列：剩余空间的 25%（1/4） -->
</Grid>
```

你可以把星号尺寸与其他尺寸模式混用。星号比例只作用于在固定尺寸和 `Auto` 列分配完成后剩下的空间：

```xml
<Grid ColumnDefinitions="100,*,2*">
    <!-- 第 0 列：固定 100px -->
    <!-- 剩余空间按 1:2 分配给第 1、2 列 -->
</Grid>
```

### MinWidth 和 MaxWidth 限制

你可以通过完整写法约束行和列的尺寸：

```xml
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="*" MinWidth="200" MaxWidth="400" />
        <ColumnDefinition Width="2*" />
    </Grid.ColumnDefinitions>
</Grid>
```

这对星号尺寸列尤其有用：你既希望它具备弹性，又不想让它变得过窄或过宽。对行来说，也可以用同样方式设置 `MinHeight` 和 `MaxHeight`。

## 行间距和列间距

使用 `RowSpacing` 和 `ColumnSpacing` 为各行各列之间添加统一间距：

```xml
<Grid ColumnDefinitions="*,*,*" RowDefinitions="Auto,Auto,Auto"
      ColumnSpacing="8" RowSpacing="8">
    <!-- 所有行和列之间都保持 8px 间距 -->
</Grid>
```

:::note
`RowSpacing` 和 `ColumnSpacing` 只会在单元格之间增加间距，不会在 Grid 的外边缘增加留白。如果你还需要外围空白，请在 `Grid` 本身上设置 `Margin` 或 `Padding`。
:::

## 跨行与跨列

使用 `Grid.RowSpan` 和 `Grid.ColumnSpan` 让控件跨越多行或多列：

```xml
<Grid ColumnDefinitions="200,*" RowDefinitions="Auto,*,Auto">
    <!-- 标题跨越两列 -->
    <TextBlock Grid.Row="0" Grid.ColumnSpan="2" Text="Header"
               FontSize="20" FontWeight="Bold" />

    <!-- 侧边栏跨越第 1、2 行 -->
    <ListBox Grid.Row="1" Grid.Column="0" Grid.RowSpan="2" />

    <!-- 内容区域 -->
    <ContentControl Grid.Row="1" Grid.Column="1" />

    <!-- 页脚只位于第 1 列 -->
    <TextBlock Grid.Row="2" Grid.Column="1" Text="Footer" />
</Grid>
```

:::tip
`Grid.RowSpan` 和 `Grid.ColumnSpan` 的默认值都是 `1`。如果你设置的跨度超过了剩余可用行列数，控件会自动扩展到 Grid 边缘，而不会报错。
:::

## 默认行列值

如果你在子控件上省略了 `Grid.Row` 或 `Grid.Column`，它们都会默认取 `0`。这意味着，当 Grid 里只有一个子元素时，你可以不写任何附加属性：

```xml
<Grid>
    <!-- 这个控件默认放在第 0 行、第 0 列 -->
    <TextBlock Text="Hello" />
</Grid>
```

如果你完全没有定义 `RowDefinitions` 或 `ColumnDefinitions`，Grid 会自动创建一个填满可用空间的星号尺寸单行单列布局。

## 表单布局

标签-输入值对的常见布局模式，是使用一个 `Auto` 列放标签，再用一个星号列放输入控件：

```xml
<Grid ColumnDefinitions="Auto,*" RowDefinitions="Auto,Auto,Auto,Auto"
      RowSpacing="8" ColumnSpacing="12">
    <TextBlock Grid.Row="0" Grid.Column="0" Text="Name:"
               VerticalAlignment="Center" />
    <TextBox Grid.Row="0" Grid.Column="1" Text="{Binding Name}" />

    <TextBlock Grid.Row="1" Grid.Column="0" Text="Email:"
               VerticalAlignment="Center" />
    <TextBox Grid.Row="1" Grid.Column="1" Text="{Binding Email}" />

    <TextBlock Grid.Row="2" Grid.Column="0" Text="Department:"
               VerticalAlignment="Center" />
    <ComboBox Grid.Row="2" Grid.Column="1" ItemsSource="{Binding Departments}"
              SelectedItem="{Binding Department}" />

    <StackPanel Grid.Row="3" Grid.Column="1" Orientation="Horizontal"
                Spacing="8" HorizontalAlignment="Right">
        <Button Content="Cancel" Command="{Binding CancelCommand}" />
        <Button Content="Save" Command="{Binding SaveCommand}" />
    </StackPanel>
</Grid>
```

给标签设置 `VerticalAlignment="Center"`，可以确保它们与对应输入控件保持垂直居中对齐，即使输入控件高度比标签更大也是如此。

## SharedSizeGroup

使用 `SharedSizeGroup` 可以让多个 `Grid` 之间的列宽（或行高）保持一致。你需要在具体的 `ColumnDefinition` 或 `RowDefinition` 上设置该属性：

```xml
<StackPanel Grid.IsSharedSizeScope="True" Spacing="4">
    <Grid ShowGridLines="False">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto" SharedSizeGroup="Labels" />
            <ColumnDefinition Width="*" />
        </Grid.ColumnDefinitions>
        <TextBlock Grid.Column="0" Text="Name:" Margin="0,0,8,0" />
        <TextBox Grid.Column="1" Text="{Binding Name}" />
    </Grid>
    <Grid ShowGridLines="False">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto" SharedSizeGroup="Labels" />
            <ColumnDefinition Width="*" />
        </Grid.ColumnDefinitions>
        <TextBlock Grid.Column="0" Text="Email Address:" Margin="0,0,8,0" />
        <TextBox Grid.Column="1" Text="{Binding Email}" />
    </Grid>
</StackPanel>
```

即使这两个名为 `Labels` 的列分别属于不同的 `Grid`，它们也会共享相同宽度（取较宽标签那一列的宽度）。父级 `StackPanel` 通过设置 `Grid.IsSharedSizeScope="True"` 来定义共享范围。

:::note
`SharedSizeGroup` 是定义在 `ColumnDefinition` 和 `RowDefinition` 上的属性，而不是写在子控件上的。组名是一个字符串，在同一个共享范围内，只要定义使用相同组名，就会采用相同的测量尺寸。
:::

## 嵌套 Grid

对于复杂布局，你可以在一个 Grid 中再嵌套另一个 Grid。每个内部 `Grid` 都会独立管理自己的行列：

```xml
<Grid ColumnDefinitions="250,*">
    <!-- 侧边栏 -->
    <Grid Grid.Column="0" RowDefinitions="Auto,*,Auto">
        <TextBlock Grid.Row="0" Text="Navigation" FontWeight="Bold" />
        <ListBox Grid.Row="1" ItemsSource="{Binding MenuItems}" />
        <Button Grid.Row="2" Content="Settings" />
    </Grid>

    <!-- 主内容 -->
    <Grid Grid.Column="1" RowDefinitions="Auto,*">
        <TextBlock Grid.Row="0" Text="{Binding Title}" FontSize="24" />
        <ContentControl Grid.Row="1" Content="{Binding CurrentPage}" />
    </Grid>
</Grid>
```

:::tip
嵌套 `Grid` 很直接，但会增加布局复杂度。如果内部网格只是为了实现简单的垂直或水平堆叠，那么通常改用 `StackPanel` 或 `DockPanel` 会有更好的可读性和性能。
:::

## 使用 Grid 实现响应式布局

你可以把 `Grid` 与 `OnFormFactor` 结合起来，让列定义随着设备类型变化，从而实现响应式设计：

```xml
<Grid ColumnDefinitions="{OnFormFactor Desktop='250,*', Mobile='*'}">
    <!-- 桌面端：双列布局 -->
    <!-- 移动端：单列布局（侧边栏隐藏或放入抽屉） -->
</Grid>
```

## 内容重叠

如果你把多个子元素放在同一个单元格里，它们在视觉上就会发生重叠。XAML 中写在后面的那个子元素会显示在最上层：

```xml
<Grid>
    <!-- 背景图像 -->
    <Image Source="background.jpg" Stretch="UniformToFill" />

    <!-- 覆盖渐变层 -->
    <Border>
        <Border.Background>
            <LinearGradientBrush StartPoint="0%,0%" EndPoint="0%,100%">
                <GradientStop Color="Transparent" Offset="0.5" />
                <GradientStop Color="#CC000000" Offset="1.0" />
            </LinearGradientBrush>
        </Border.Background>
    </Border>

    <!-- 顶层文本 -->
    <TextBlock Text="Hello World"
               VerticalAlignment="Bottom"
               Margin="16"
               Foreground="White"
               FontSize="24" />
</Grid>
```

当你省略 `Grid.Row` 和 `Grid.Column` 时，子元素默认都会放在第 0 行、第 0 列。你也可以使用 `ZIndex` 来独立控制叠放顺序，而不依赖 XAML 中的书写先后：

```xml
<Grid>
    <Border ZIndex="1" Background="Red" Opacity="0.5" />
    <Border ZIndex="2" Background="Blue" Opacity="0.5" />
    <!-- 即使调整了书写顺序，蓝色边框仍会显示在上层 -->
</Grid>
```

### 使用负边距实现部分重叠

如果你希望两个元素在不共用同一单元格的情况下仍然产生一定重叠，可以给第二个元素设置负边距：

```xml
<StackPanel Orientation="Horizontal">
    <Border Background="LightBlue" Padding="12">
        <TextBlock Text="First" />
    </Border>
    <Border Background="LightCoral" Padding="12" Margin="-10,0,0,0">
        <TextBlock Text="Second (overlaps by 10px)" />
    </Border>
</StackPanel>
```

负的左边距会把第二个元素向左拉 10 像素，从而覆盖在第一个元素上。由于它在 XAML 中出现得更晚，因此也会显示在更上层。这种技巧不仅适用于 `Grid`，在其他面板中同样适用。

## 调试 Grid 布局

在开发阶段，可以为 `Grid` 设置 `ShowGridLines="True"`，以便直观看到行列边界：

```xml
<Grid ColumnDefinitions="Auto,*,200" RowDefinitions="Auto,*"
      ShowGridLines="True">
    <!-- 网格线会以虚线形式显示，方便你观察每个单元格 -->
</Grid>
```

请记得在正式发布前移除 `ShowGridLines`，因为它仅用于开发调试辅助。

## 另请参阅

- [Grid control reference](/controls/layout/panels/grid)
- [Layout overview](/docs/layout)
- [Positioning controls](/docs/layout/positioning-controls)
