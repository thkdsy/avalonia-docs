---
id: resources
title: 资源概览
description: 定义、引用和管理可复用的 XAML 资源、合并字典和主题变体。
doc-type: overview
---

Avalonia 中的资源是在 XAML 中定义、并可在整个应用中共享的可复用对象。画刷、颜色、厚度、字符串和样式都常常被定义为资源，以保持视觉一致性并简化维护工作。

## 定义资源

资源存储在 `ResourceDictionary` 集合中，你可以把它声明在任意元素的 `Resources` 属性上。每个资源都必须拥有一个 `x:Key`：

```xml
<Application.Resources>
    <SolidColorBrush x:Key="PrimaryBrush" Color="#6366F1" />
    <SolidColorBrush x:Key="DangerBrush" Color="#EF4444" />
    <x:Double x:Key="DefaultSpacing">8</x:Double>
    <Thickness x:Key="PagePadding">24,16</Thickness>
</Application.Resources>
```

资源可以定义在树中的任意层级：

| 层级 | 作用范围 |
|---|---|
| `Application.Resources` | 整个应用内都可用 |
| `Window.Resources` | 仅在该窗口内可用 |
| `UserControl.Resources` | 仅在该用户控件内可用 |
| 任意控件的 `.Resources` | 对该控件及其后代可用 |
| `Style.Resources` | 仅在该样式块内部可用 |

## 使用资源

### StaticResource

`StaticResource` 会在 XAML 加载时执行一次性查找：

```xml
<Button Background="{StaticResource PrimaryBrush}" />
<StackPanel Spacing="{StaticResource DefaultSpacing}" />
```

如果找不到资源，运行时会抛出异常。

### DynamicResource

`DynamicResource` 会监听变化，并在资源值于运行时发生变化时自动更新（例如主题切换时）：

```xml
<TextBlock Foreground="{DynamicResource SystemAccentColor}" />
<Border Background="{DynamicResource WindowBackgroundBrush}" />
```

### 何时使用它们

| 用法 | 适用场景 |
|---|---|
| `StaticResource` | 资源值在运行时不会变化，查找速度也略快。 |
| `DynamicResource` | 资源值可能发生变化（例如主题切换、用户偏好变化、运行时更新）。 |

:::tip
对于需要响应主题变化的颜色、画刷和尺寸，请使用 `DynamicResource`。对于数据模板、转换器以及其他不会变化的结构性资源，请使用 `StaticResource`。
:::

## 资源查找顺序

当你引用某个资源时，Avalonia 会从引用出现的位置开始，沿逻辑树向上查找：

1. 元素自身的 `Resources` 字典
2. 该层级的合并字典
3. 父元素的 `Resources`（以及它的合并字典）
4. 继续沿逻辑树向上查找
5. 各层级上的样式资源
6. `Application.Resources` 及其合并字典
7. 主题资源

第一个匹配到的资源会胜出。这意味着：离使用位置更近定义的资源，会覆盖更高层级定义的同名资源。

## 合并字典

你可以将资源拆分到独立文件中，再把它们合并到任意 `ResourceDictionary`：

```xml title="Resources/Colors.axaml"
<ResourceDictionary xmlns="https://github.com/avaloniaui"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <SolidColorBrush x:Key="PrimaryBrush" Color="#6366F1" />
    <SolidColorBrush x:Key="SecondaryBrush" Color="#8B5CF6" />
</ResourceDictionary>
```

```xml title="App.axaml"
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <ResourceInclude Source="/Resources/Colors.axaml" />
            <ResourceInclude Source="/Resources/Sizes.axaml" />
        </ResourceDictionary.MergedDictionaries>
        <!-- Additional inline resources -->
        <x:String x:Key="AppName">My Application</x:String>
    </ResourceDictionary>
</Application.Resources>
```

### MergeResourceInclude 与 ResourceInclude 的区别

| 类型 | 行为 |
|---|---|
| `ResourceInclude` | 创建一个独立的资源字典作用域，属于标准的资源文件引入方式。 |
| `MergeResourceInclude` | 直接将资源合并到父字典中，使它们像内联定义的一样可访问。 |

## 主题变体资源

你可以使用 `ThemeDictionaries` 为浅色和深色主题定义不同的资源值：

```xml
<ResourceDictionary>
    <ResourceDictionary.ThemeDictionaries>
        <ResourceDictionary x:Key="Light">
            <SolidColorBrush x:Key="CardBackground" Color="White" />
            <SolidColorBrush x:Key="CardForeground" Color="#1A1A1A" />
        </ResourceDictionary>
        <ResourceDictionary x:Key="Dark">
            <SolidColorBrush x:Key="CardBackground" Color="#2D2D2D" />
            <SolidColorBrush x:Key="CardForeground" Color="#FAFAFA" />
        </ResourceDictionary>
    </ResourceDictionary.ThemeDictionaries>
</ResourceDictionary>
```

请使用 `DynamicResource` 来引用主题变体资源，这样它们会在主题变化时自动更新：

```xml
<Border Background="{DynamicResource CardBackground}">
    <TextBlock Foreground="{DynamicResource CardForeground}" Text="Hello" />
</Border>
```

## 在代码中访问资源

Avalonia 提供了四种以编程方式访问资源的方法：

```csharp
// Direct dictionary access (does not search merged dictionaries or parent elements)
var brush = (SolidColorBrush)this.Resources["PrimaryBrush"];

// Search merged dictionaries at the current level only
if (this.TryGetResource("PrimaryBrush", this.ActualThemeVariant, out var result))
{
    // result contains the resource value
}

// Search the full logical tree (most common usage)
if (this.TryFindResource("PrimaryBrush", this.ActualThemeVariant, out var found))
{
    // found contains the resource value from anywhere in the tree
}

// Observable for runtime changes
myBorder.Bind(Border.BackgroundProperty,
    this.GetResourceObservable("PrimaryBrush"));
```

| 方法 | 查找合并字典 | 查找父元素 |
|---|---|---|
| `Resources["key"]` | 否 | 否 |
| `TryGetResource` | 是 | 否 |
| `TryFindResource` | 是 | 是 |
| `GetResourceObservable` | 是 | 是（并监听变化） |

## 在运行时更新资源

你可以在代码中修改资源，以动态改变应用外观：

```csharp
// Update a resource (DynamicResource references update automatically)
Application.Current!.Resources["PrimaryBrush"] =
    new SolidColorBrush(Colors.Red);
```

只有 `DynamicResource` 引用会响应运行时资源变化。`StaticResource` 引用会保留初始值不变。

## 另请参阅

- [Resource Dictionary](/docs/app-development/resource-dictionary)：创建和组织资源字典的分步指南。
- [Theme Variants](/docs/styling/theme-variants)：主题感知资源的工作方式。
- [Styles](/docs/styling/styles)：如何在样式定义中使用资源。
- [Sharing Styles](/docs/styling/sharing-styles)：组织和共享样式资源。
