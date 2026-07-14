---
id: custom-fonts
title: 自定义字体
---

Avalonia 支持自定义 TrueType（`.ttf`）和 OpenType（`.otf`）字体。你可以将字体文件嵌入应用，并在 XAML 中引用它们，从而摆脱对宿主系统已安装字体的依赖。

使用自定义字体主要有三种方式，分别适用于不同场景：

| 方式 | 最适合 |
|----------|----------|
| [静态资源](#static-resource-fonts) | 通过命名资源键在特定位置使用字体 |
| [嵌入式字体集合](#embedded-font-collections) | 无需资源键，直接按字体家族名称引用字体 |
| [预构建字体包](#pre-built-font-packages) | 在多个项目之间共享同一个字体包 |

## 字体 URI 格式

这三种方式都使用同一种 URI 格式来定位嵌入的字体文件：

```text
avares://AssemblyName/Path/To/Fonts#Font Family Name
```

- `avares://AssemblyName/Path/To/Fonts` 是使用 Avalonia 资源协议指向字体目录或字体文件的路径。
- `#Font Family Name` 是字体的内部家族名称，而不是文件名。

:::caution
`#FontFamilyName` 后缀是必需的。没有它，Avalonia 无法识别应该加载哪一种字形，字体会静默回退到默认字体。这里的家族名称必须与字体内部元数据中的名称一致，而不是磁盘上的文件名。
:::

为了让字体文件在运行时可用，你的项目文件必须将它们包含为 `AvaloniaResource`：

```xml title="MyApp.csproj"
<AvaloniaResource Include="Assets\Fonts\*" />
```

## 静态资源字体

你可以将字体声明为 XAML 资源，并通过 `StaticResource` 标记扩展按键引用它。这种方式可以让你明确控制字体在哪些位置使用。

在应用或窗口资源中定义字体：

```xml title="App.axaml"
<Application.Resources>
    <FontFamily x:Key="NunitoFont">avares://MyApp/Assets/Fonts#Nunito</FontFamily>
</Application.Resources>
```

然后在任何带有 `FontFamily` 属性的控件上引用它：

```xml
<TextBlock FontFamily="{StaticResource NunitoFont}"
           FontSize="24"
           Text="Hello in Nunito" />
```

:::caution
应用级资源会涉及资源字典合并，这可能导致字体定义静默失败。如果你把字体定义在 `<Application.Resources>` 中但没有生效，请将资源包裹在 `ResourceDictionary.MergedDictionaries` 结构中：

```xml title="App.axaml"
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <ResourceDictionary>
                <FontFamily x:Key="NunitoFont">avares://MyApp/Assets/Fonts#Nunito</FontFamily>
            </ResourceDictionary>
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

另一种方法是将字体资源定义在独立的 AXAML 文件中，并通过 `ResourceInclude` 引入。定义在 `<Window.Resources>` 作用域中的字体资源则不受这个问题影响。
:::

## 嵌入式字体集合

[`EmbeddedFontCollection`](/api/avalonia/media/fonts/embeddedfontcollection) 可以在自定义 URI 方案下注册一个字体文件目录。这样你就能在 XAML 中直接按字体家族名称引用字体，而不必为每个字体单独声明资源键。

该集合会将一个自定义 URI（例如 `fonts:MyFonts`）映射到某个资源目录（例如 `avares://MyApp/Assets/Fonts`）。你可以通过继承 `EmbeddedFontCollection` 来创建它：

```csharp
using Avalonia.Media.Fonts;

public sealed class MyFontCollection : EmbeddedFontCollection
{
    public MyFontCollection() : base(
        new Uri("fonts:MyFonts", UriKind.Absolute),
        new Uri("avares://MyApp/Assets/Fonts", UriKind.Absolute))
    {
    }
}
```

然后在应用配置中注册这个集合：

```csharp title="Program.cs"
public static AppBuilder BuildAvaloniaApp() =>
    AppBuilder.Configure<App>()
        .UsePlatformDetect()
        .ConfigureFonts(fontManager =>
        {
            fontManager.AddFontCollection(new MyFontCollection());
        })
        .LogToTrace();
```

之后，就可以通过 `{scheme}:{collection-key}#{font-family-name}` 格式引用集合中的任意字体：

```xml
<TextBlock FontFamily="fonts:MyFonts#Nunito"
           FontSize="24"
           Text="Hello in Nunito" />
```

`EmbeddedFontCollection` 会搜索指定目录中的所有文件，并加载与请求字体家族名称匹配的字体。一个集合中可以包含多个字体家族和多个字体文件。如果要使用同一集合中的其他字体，只需修改 `#` 后面的名称。

## 预构建字体包

字体包会将 `EmbeddedFontCollection` 封装进一个 NuGet 包中，方便你在多个项目之间共享使用。[`Avalonia.Fonts.Inter`](https://github.com/AvaloniaUI/Avalonia/tree/master/src/Avalonia.Fonts.Inter) 就是这种模式的一个示例。

要使用 Inter 字体，请安装 `Avalonia.Fonts.Inter` NuGet 包，并在应用配置中加入 `.WithInterFont()`：

```csharp title="Program.cs"
public static AppBuilder BuildAvaloniaApp() =>
    AppBuilder.Configure<App>()
        .UsePlatformDetect()
        .WithInterFont()
        .LogToTrace();
```

然后在 XAML 中引用该字体：

```xml
<TextBlock FontFamily="fonts:Inter#Inter"
           FontSize="24"
           Text="Hello in Inter" />
```

## OpenType 字体特性

`FontFeatures` 属性用于控制 OpenType 特性，例如连字、等宽数字和替代字形。特性通过逗号分隔的标签来指定，语法遵循 [HarfBuzz syntax](https://harfbuzz.github.io/harfbuzz-hb-common.html#hb-feature-from-string)：

```xml
<!-- Disable contextual alternates and enable tabular numbers -->
<TextBlock Text="111111 x64 ->" FontFeatures="-calt,+tnum" />
```

你也可以在 `TextBlock` 中的单个 `Run` 元素上应用字体特性：

```xml
<TextBlock>
    <Run Text="Regular: 12345" />
    <Run Text="Tabular: 12345" FontFeatures="+tnum" />
</TextBlock>
```

或者使用附加属性将它们设置到某个容器上：

```xml
<StackPanel TextElement.FontFeatures="+tnum">
    <TextBlock Text="12345" />
    <TextBlock Text="67890" />
</StackPanel>
```

常见的 OpenType 特性标签：

| 标签 | 特性 |
|---|---|
| `+tnum` / `-tnum` | 启用/禁用等宽（固定宽度）数字。 |
| `+liga` / `-liga` | 启用/禁用标准连字。 |
| `+calt` / `-calt` | 启用/禁用上下文替代字形。 |
| `+smcp` | 启用小型大写字母。 |
| `+onum` | 启用旧式（比例）数字。 |

:::info
可用特性取决于具体字体。并非所有字体都支持全部 OpenType 特性。
:::

## 自定义字体匹配

对于更高级的字体处理场景，你可以重写字体集合上的相关方法，以控制 Avalonia 如何选择回退字体，以及如何创建合成字形。

### 自定义字符匹配

你可以重写 `TryMatchCharacter`，在请求的字体家族中找不到某个字符时，控制应改用哪种字体。这对于实现应用专属的字体回退链非常有用：

```csharp
public sealed class MyFontCollection : EmbeddedFontCollection
{
    public MyFontCollection() : base(
        new Uri("fonts:MyFonts", UriKind.Absolute),
        new Uri("avares://MyApp/Assets/Fonts", UriKind.Absolute))
    {
    }

    public override bool TryMatchCharacter(
        int codepoint,
        FontStyle fontStyle,
        FontWeight fontWeight,
        FontStretch fontStretch,
        CultureInfo? culture,
        out Typeface typeface)
    {
        // Custom logic to match characters to specific fonts
        // Return true if a match was found, false to fall through
        // to the default matching behavior
        return base.TryMatchCharacter(
            codepoint, fontStyle, fontWeight, fontStretch,
            culture, out typeface);
    }
}
```

### 控制合成字形

当 Avalonia 无法为请求的字重或字形找到精确匹配时，它会创建一个合成（通过算法模拟样式）的字形。你可以在字体集合上实现 `IFontCollection2`，并重写合成字形的创建行为，以阻止某些特定字体家族使用这种机制。

## 支持的字体格式

大多数 TrueType（`.ttf`）和 OpenType（`.otf`、`.ttf`）字体都受支持。当前尚不支持可变字体，详见 [Issue #11092](https://github.com/AvaloniaUI/Avalonia/issues/11092)。

## 故障排查

### 字体未显示（回退到默认字体）

如果你的自定义字体静默回退到了默认字体，请检查以下常见原因：

**1. Font files not included as AvaloniaResource**

你的项目文件必须显式将字体文件包含为 `AvaloniaResource`。否则字体文件不会被嵌入到构建输出中，Avalonia 也就无法找到它们。

```xml title="MyApp.csproj"
<AvaloniaResource Include="Assets\Fonts\*" />
```

添加后请重新生成项目。此时程序集文件大小通常会增加，这说明字体已经被嵌入进去了。

**2. Missing font family name in the URI**

字体 URI 必须在 `#` 分隔符后面包含字体家族名称，仅提供字体集合路径是不够的。

```xml
<!-- Wrong: missing #FontFamilyName -->
<FontFamily x:Key="MyFont">avares://MyApp/Assets/Fonts</FontFamily>

<!-- Correct: includes the font family name -->
<FontFamily x:Key="MyFont">avares://MyApp/Assets/Fonts#Roboto Mono</FontFamily>
```

同样的规则也适用于 `EmbeddedFontCollection` 的 URI：

```xml
<!-- Wrong -->
<TextBlock FontFamily="fonts:MyFonts" />

<!-- Correct -->
<TextBlock FontFamily="fonts:MyFonts#Source Code Pro" />
```

`#` 后面的字体家族名称必须与字体的内部名称一致，而不是文件名。你可以通过字体查看器打开该字体文件，或查看字体元数据来找到内部名称。

**3. 字体在 WebAssembly（WASM）中不起作用**

浏览器环境无法访问系统字体。如果你的字体在桌面端可用、但在 WASM 中不可用，那么浏览器很可能回退到了一个并不存在的系统字体。

要解决这个问题，请使用完整的字体集合 URI 语法，而不要依赖系统字体回退。例如，在使用 `Avalonia.Fonts.Inter` 包时：

```xml
<!-- May fail in WASM -->
<TextBlock FontFamily="Inter" />

<!-- Works in all environments -->
<TextBlock FontFamily="fonts:Inter#Inter" />
```

这样可以确保 Avalonia 从嵌入式字体集合中解析字体，而不是尝试去查找系统字体。

## 另请参阅

- [Typography](/docs/styling/typography): Font size, weight, style, letter spacing, line height, and text decorations.
- [How to add a custom font](/docs/how-to/custom-font-how-to)
- [Assets](/docs/fundamentals/including-assets)
- [Styles](/docs/styling/styles)
