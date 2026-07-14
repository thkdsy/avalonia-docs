---
id: xaml
title: 平台特定 XAML
---

## OnPlatform 标记扩展

### 概述
Avalonia 中的 OnPlatform 标记扩展允许开发者根据应用运行所在的操作系统，为属性指定不同的值。这对于需要按照平台调整 UI 或行为的跨平台应用尤其有用。

### 标记扩展语法中的基本用法

你可以为各个平台分别指定值，并设置一个在没有匹配到特定平台时使用的默认值：

```xml
<TextBlock Text="{OnPlatform Default='Unknown', Windows='Im Windows', macOS='Im macOS', Linux='Im Linux'}"/>
```

你也可以使用构造器语法直接定义默认值，而无需写 `Default` 关键字。不过平台特定属性仍然需要单独定义：

```xml
<TextBlock Text="{OnPlatform 'Hello World', Android='Im Android'}"/>
```

这个标记扩展并不只适用于字符串，也可以用于其他类型：

```xml
<Border Height="{OnPlatform 10, Windows=50.5}"/>
```

### 指定类型参数

你可以使用自定义 TypeArguments 来显式指定这些值的类型：

```xml
<TextBlock Tag="{OnPlatform '0, 0, 0, 0', Windows='10, 10, 10, 10', x:TypeArguments=Thickness}"/>
```

在上面的示例中，`Tag` 属性的类型是 `object`，因此编译器没有足够的信息来解析输入字符串。如果不指定 TypeArguments，该属性在所有平台上都会被当作 `string`。但由于这里提供了 `TypeArguments`，编译器就会把这些值解析为 `Thickness`。

### 嵌套标记扩展

OnPlatform 扩展支持在内部嵌套其他标记扩展：

```xml
<Border Background="{OnPlatform Default={StaticResource DefaultBrush}, Windows={StaticResource WindowsBrush}}"/>
```

### XML 语法

OnPlatform 也可以通过 XML 语法来定义属性值：

```xml
<StackPanel>
    <OnPlatform>
        <OnPlatform.Default>
            <ToggleButton Content="Hello World" />
        </OnPlatform.Default>
        <OnPlatform.iOS>
            <ToggleSwitch Content="Hello iOS" />
        </OnPlatform.iOS>
    </OnPlatform>
</StackPanel>
```

注意，在这个示例里，`OnPlatform` 是 `StackPanel` 的子元素。但在运行时，实际上只会创建一个控件（`ToggleButton` 或 `ToggleSwitch`）并把它加入到 StackPanel 中。

### 复杂属性设置器

和前面的示例类似，OnPlatform 也可以作为复杂属性设置器的一部分，出现在 `ResourceDictionary` 或其他字典、集合中：

```xml
<ResourceDictionary>
    <OnPlatform x:Key="MyBrush">
        <OnPlatform.Default>
            <SolidColorBrush Color="Blue" />
        </OnPlatform.Default>
        <OnPlatform.iOS>
            <SolidColorBrush Color="Yellow" />
        </OnPlatform.iOS>
    </OnPlatform>
</ResourceDictionary>
```

### XML 合并语法

为了避免重复编写分支，可以在单个分支中定义多个平台。另一个常见用法是按平台引入不同的样式：

```xml
<Application.Styles>
    <!-- 始终包含 -->
    <FluentTheme />

    <!-- 运行时只会执行其中一个分支 -->
    <OnPlatform>
        <!-- if (Android || iOS) -->
        <On Options="Android, iOS">
            <StyleInclude Source="/Styles/Mobile.axaml" />
        </On>
        <!-- else -->
        <On Options="Default">
            <StyleInclude Source="/Styles/Default.axaml" />
        </On>
    </OnPlatform>
</Application.Styles>
```

### 其他细节

`OnPlatform` 标记扩展的工作方式与 C# 中的 switch-case 类似。编译器会为所有可能的值生成分支，但运行时只会根据条件执行其中一个分支。

还需要注意的是，如果应用使用特定的 [Runtime Identifier](https://learn.microsoft.com/en-us/dotnet/core/rid-catalog) 构建，并启用了 [Trimming](https://learn.microsoft.com/en-us/dotnet/core/deploying/trimming/trimming-options)，`OnPlatform` 扩展中只有可能命中的分支会被保留下来。例如，如果 `OnPlatform` 包含 Windows 和 macOS 分支，但应用只为 Windows 构建，那么其他分支就会被移除，这也有助于减小应用体积。


## OnFormFactor 标记扩展

`OnFormFactor` 标记扩展与 `OnPlatform` 的工作方式类似，并且整体语法也相同。主要区别在于，它不是按平台定义值，而是按设备形态定义值，例如 Desktop 和 Mobile：

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <TextBlock Text="{OnFormFactor 'Default value', Mobile='Im Mobile', Desktop='Im Desktop'}"/>
</UserControl>
```

`OnFormFactor` 不具备编译期裁剪优化，因为设备形态无法在编译时确定。这些标记扩展都不是动态的；值一旦设置完成，运行时不会再自动变化。

## 另请参阅

- [平台特定 .NET](/docs/platform-specific-guides/dotnet)
