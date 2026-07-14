---
id: localizing
title: 使用 ResX 进行本地化
description: 使用 ResX 资源文件、运行时语言切换和 RTL 支持来本地化 Avalonia 应用。
doc-type: tutorial
---

本地化是为全球用户提供良好使用体验的重要一步。在 .NET 中，`ResXResourceReader` 和 `ResXResourceWriter` 类用于读取和写入基于 XML 格式的资源文件（`.resx`）。本指南将带你一步步了解如何使用 ResX 文件对 Avalonia 应用进行本地化。


<GitHubSampleLink title="Localization" link="https://github.com/AvaloniaUI/AvaloniaUI.QuickGuides/tree/main/Localization/"/>


## 将 ResX 文件添加到项目中

在开始本地化之前，你需要为每种要支持的语言准备对应的 ResX 文件。本指南使用了三个 ResX 文件，分别对应以下区域性：

* `Resources.fil-PH.resx`（菲律宾语）
* `Resources.ja-JP.resx`（日语）
* `Resources.resx`（默认语言，通常是英语）

每个 ResX 文件都包含与应用中所用键相对应的翻译文本。

在本示例中，我们把新文件添加到了名为 `Lang` 的新文件夹中。由于 .NET 生成器会根据文件夹结构生成命名空间，因此你的项目结构可能会略有不同。

:::caution
如果你把这些文件放进 `Assets` 文件夹，请务必将 `Build Action` 改为 `Embedded resource`，否则代码生成可能会失败。
:::

## 设置区域性

要让应用使用某种特定语言，你需要设置当前区域性。这可以在 `App.axaml.cs` 文件中完成。下面的示例把区域性设置为菲律宾语（`fil-PH`）：

```cs title="App.xaml.cs"
public partial class App : Application
{
    public override void Initialize()
    {
        AvaloniaXamlLoader.Load(this);
    }

    public override void OnFrameworkInitializationCompleted()
    {
        // highlight-start
        Lang.Resources.Culture = new CultureInfo("fil-PH");
        // highlight-end
        if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktop)
        {
            desktop.MainWindow = new MainWindow
            {
                DataContext = new MainWindowViewModel(),
            };
        }

        base.OnFrameworkInitializationCompleted();
    }
}
```
请根据需要将 `fil-PH` 替换为对应的区域性代码。

## 在视图中使用本地化文本

要在视图中使用本地化文本，你可以在 XAML 中通过静态方式引用资源：

```xml
<TextBlock Text="{x:Static lang:Resources.GreetingText}"/>
```

在上面的示例中，`GreetingText` 是与 ResX 文件中某个字符串对应的键。`{x:Static}` 标记扩展用于引用 .NET 类中定义的静态属性；在这里，它引用的是资源文件中的属性（`lang:Resources.GreetingText`）。

这样就完成了。现在你已经成功使用 ResX 对 Avalonia 应用进行了本地化。通过将区域性设置为不同的区域代码，你就可以让用户界面显示为对应语言，从而构建一个支持多语言、面向全球用户的应用。

:::caution
如果希望这些本地化属性能在 XAML 中被访问，那么由资源文件生成的代码必须是公开可见的。默认情况下，`Resources` 类由 `ResXFileCodeGenerator` 生成，并且可见性是 `internal`。请确保将自定义工具改为 `PublicResXFileCodeGenerator`。对应的 `csproj` 片段应如下所示：

```xml
<ItemGroup>
  <EmbeddedResource Update="Resources.resx">
    <Generator>PublicResXFileCodeGenerator</Generator>
    <LastGenOutput>Resources.Designer.cs</LastGenOutput>
  </EmbeddedResource>
</ItemGroup>

<ItemGroup>
  <Compile Update="Resources.Designer.cs">
    <DesignTime>True</DesignTime>
    <AutoGen>True</AutoGen>
    <DependentUpon>Resources.resx</DependentUpon>
  </Compile>
</ItemGroup>
```

另外请注意，只有默认资源文件（`Resources.resx`）才应该生成代码。

:::

## 运行时语言切换

你可以在运行时修改区域性，从而允许用户在不重启应用的情况下切换语言：

```csharp
public void SwitchLanguage(string cultureCode)
{
    Lang.Resources.Culture = new CultureInfo(cultureCode);
    // 为所有本地化属性触发 PropertyChanged
    // 或重新加载视图以获取新的字符串
}
```

请注意，`x:Static` 绑定在区域性变化时不会自动更新。因为 `x:Static` 只会在加载时解析一次值，所以在刷新视图之前，UI 不会反映出新的语言。要解决这个问题，可以考虑以下方案之一：

- **重新加载视图或窗口。** 关闭并重新创建窗口（或用户控件），这样所有 `x:Static` 引用都会基于新区域性重新计算。
- **使用带 `INotifyPropertyChanged` 的本地化服务。** 创建一个服务类，将本地化字符串作为属性暴露出来，并在区域性变化时触发 `PropertyChanged`。然后绑定到这些属性，而不是使用 `x:Static`。

## 从右到左（RTL）支持

Avalonia 通过 [`FlowDirection`](/api/avalonia/media/flowdirection) 属性支持从右到左布局。将 `FlowDirection` 设置为 `RightToLeft` 后，子控件的布局会被镜像翻转，这对于阿拉伯语、希伯来语和波斯语等语言非常重要。

```xml
<Window FlowDirection="RightToLeft">
    <!-- 所有子控件都会镜像其布局 -->
</Window>
```

你也可以根据当前区域性动态设置 `FlowDirection`：

```csharp
var culture = new CultureInfo("ar-SA");
if (culture.TextInfo.IsRightToLeft)
{
    mainWindow.FlowDirection = FlowDirection.RightToLeft;
}
```

以下控件会遵循 `FlowDirection`，并据此调整布局：

- `StackPanel`（反转水平项的顺序）
- `Grid`（镜像列顺序）
- `DockPanel`（交换左右停靠方向）
- `TextBlock`（调整文本对齐方式）

## 文化感知格式化

当你在数据绑定中使用 `StringFormat` 时，格式化结果会遵循当前线程的区域性。例如，货币格式会自动适配当前激活的文化环境：

```xml
<TextBlock Text="{Binding Price, StringFormat='{}{0:C}'}" />
```

这会使用当前区域性的货币符号和格式来显示 `Price` 的值。如果你想控制格式化所使用的区域性，请在应用启动早期设置 `Thread.CurrentThread.CurrentCulture`：

```csharp
Thread.CurrentThread.CurrentCulture = new CultureInfo("de-DE");
```

当使用 `de-DE` 时，`1234.56` 会显示为 `1.234,56 €`，而不是 `$1,234.56`。

## 平台注意事项

Avalonia 的本地化功能在所有受支持平台上的行为基本一致：

| 功能 | Windows | macOS | Linux | Mobile |
|---|---|---|---|---|
| ResX 本地化 | 完整支持 | 完整支持 | 完整支持 | 完整支持 |
| FlowDirection | 完整支持 | 完整支持 | 完整支持 | 完整支持 |
| 系统区域性检测 | `CultureInfo.CurrentCulture` | 相同 | 相同 | 相同 |

## 另请参阅

- [Resources](/docs/app-development/resources)：应用资源。
- [Custom Fonts](/docs/styling/custom-fonts)：为不同文字系统加载字体。
