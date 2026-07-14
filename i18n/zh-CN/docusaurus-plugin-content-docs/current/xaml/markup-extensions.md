---
id: markup-extensions
title: 标记扩展
---

标记扩展是用花括号 `{}` 包裹的特殊表达式，用于在 XAML 中提供动态值。它扩展了简单字符串字面量之外的表达能力。

## Binding

在控件属性与源属性之间创建数据绑定：

```xml
<TextBlock Text="{Binding UserName}" />
```

### 常见绑定参数

| 参数 | 说明 |
|---|---|
| `Path` | 源对象上的属性路径。这是默认参数。 |
| `Mode` | 绑定方向：`OneWay`、`TwoWay`、`OneTime`、`OneWayToSource`、`Default`。 |
| `Converter` | 用于转换值的 `IValueConverter`。 |
| `ConverterParameter` | 传递给转换器的参数。 |
| `StringFormat` | 应用于绑定值的格式字符串。 |
| `FallbackValue` | 当绑定失败时使用的值。 |
| `TargetNullValue` | 当源值为 `null` 时使用的值。 |
| `Source` | 显式指定的源对象（会覆盖 `DataContext`）。 |
| `ElementName` | 绑定到同一 XAML 作用域中的某个已命名元素。 |
| `RelativeSource` | 绑定到相对元素（例如 `TemplatedParent`、`Self`）。 |

```xml
<TextBlock Text="{Binding Price, StringFormat='Price: {0:C}'}" />
<TextBox Text="{Binding Name, Mode=TwoWay}" />
<TextBlock Text="{Binding Amount, Converter={StaticResource CurrencyConverter}}" />
<Image Source="{Binding ImageUrl, FallbackValue={x:Null}}" />
```

:::info
如需完整了解绑定语法，请参阅 [数据绑定语法](/docs/data-binding/data-binding-syntax)。
:::

## CompiledBinding

一种会在编译时验证的绑定。当设置 `x:CompileBindings="True"` 时，它在功能上等价于 `{Binding}`，但也可以显式使用：

```xml
<TextBlock Text="{CompiledBinding UserName}" />
```

编译绑定要求当前作用域中设置 `x:DataType`。它可以在编译期检查绑定路径，并提供更好的运行时性能。

## ReflectionBinding

一种基于反射的绑定，会绕过编译期验证。在需要绑定到动态属性或延迟绑定属性时使用它：

```xml
<TextBlock Text="{ReflectionBinding DynamicProperty}" />
```

## StaticResource

按键从当前元素的资源链中查找资源，并沿着父元素一直向上查找到 `Application.Resources`。该查找只会在加载时执行一次。

```xml
<TextBlock Foreground="{StaticResource PrimaryBrush}" />
```

如果找不到资源，就会在运行时抛出异常。

### 资源查找顺序

1. 当前元素的 `Resources` 字典。
2. 父元素的 `Resources` 字典，沿逻辑树逐级向上查找。
3. `Application.Resources` 字典。
4. 主题资源。

## DynamicResource

与 `StaticResource` 类似，但如果资源在运行时发生变化，值也会自动更新（例如切换主题时）：

```xml
<TextBlock Foreground="{DynamicResource SystemAccentColor}" />
```

以下情况适合使用 `DynamicResource`：
- 依赖主题的值（颜色、画刷、尺寸）
- 会在运行时变化的资源（例如用户偏好设置）
- 定义在主题字典中的资源

以下情况适合使用 `StaticResource`：
- 运行时永远不会变化的值
- 对性能敏感的场景（静态查找会略快一些）

## TemplateBinding

一种轻量级绑定，用于 `ControlTemplate` 定义内部，绑定到模板父控件的某个属性：

```xml
<ControlTemplate TargetType="Button">
    <Border Background="{TemplateBinding Background}"
            Padding="{TemplateBinding Padding}"
            CornerRadius="{TemplateBinding CornerRadius}">
        <ContentPresenter Content="{TemplateBinding Content}" />
    </Border>
</ControlTemplate>
```

`TemplateBinding` 等价于 `{Binding RelativeSource={RelativeSource TemplatedParent}}`，但效率更高。

:::info
`TemplateBinding` 只支持 `OneWay` 模式。如果你需要在模板中使用 `TwoWay` 绑定，请改用 `{Binding RelativeSource={RelativeSource TemplatedParent}}`。
:::

## OnPlatform

根据当前操作系统返回不同的值：

```xml
<TextBlock FontSize="{OnPlatform Default=14, macOS=13, Android=16}" />

<Window Width="{OnPlatform 800, macOS=900}" />
```

支持的平台值包括：`Default`、`Windows`、`macOS`、`Linux`、`Android`、`iOS`、`Browser`。

对于复杂值，可以使用嵌套语法：

```xml
<TextBlock>
    <TextBlock.Margin>
        <OnPlatform>
            <On Options="Windows">8,4</On>
            <On Options="macOS">12,6</On>
            <On Options="Default">8,4</On>
        </OnPlatform>
    </TextBlock.Margin>
</TextBlock>
```

## OnFormFactor

根据设备形态返回不同的值：

```xml
<TextBlock FontSize="{OnFormFactor Default=14, Desktop=14, Mobile=18}" />
```

支持的值包括：`Default`、`Desktop`、`Mobile`。

## RelativeSource

指定一个相对于绑定目标在视觉树或逻辑树中位置的绑定源。它用于 `{Binding}` 内部：

```xml
<!-- 绑定到自身 -->
<Border Tag="{Binding Width, RelativeSource={RelativeSource Self}}" />

<!-- 绑定到模板父级 -->
<TextBlock Text="{Binding Header, RelativeSource={RelativeSource TemplatedParent}}" />

<!-- 按类型绑定到祖先元素 -->
<TextBlock Text="{Binding DataContext.Title,
    RelativeSource={RelativeSource FindAncestor, AncestorType=Window}}" />
```

## 标记扩展语法规则

### 基本语法

```xml
Property="{ExtensionName}"
Property="{ExtensionName Value}"
Property="{ExtensionName Param1=Value1, Param2=Value2}"
```

### 嵌套

标记扩展可以嵌套使用：

```xml
<TextBlock Text="{Binding Name, Converter={StaticResource UpperCaseConverter}}" />
```

### 转义

如果要把属性设置为以 `{` 开头的字面量字符串，请使用一对空花括号：

```xml
<TextBlock Text="{}{这是普通文本，不是标记扩展}" />
```

## 另请参阅

- [XAML 参考](/docs/xaml)：XAML 语法概览。
- [x: 指令](/docs/xaml/directives)：XAML 语言指令。
- [数据绑定语法](/docs/data-binding/data-binding-syntax)：完整的绑定语法参考。
- [编译绑定](/docs/data-binding/compiled-bindings)：编译期验证的绑定。
- [标记扩展（数据绑定）](/docs/data-binding/markup-extensions)：与绑定相关的标记扩展细节。
