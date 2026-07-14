---
id: resource-dictionary
title: 创建资源字典
description: 创建、引入并合并资源字典文件，以组织可复用的 XAML 资源。
doc-type: how-to
---

import AddNewItemDialog from '/img/gitbook-import/assets/image (8) (1) (2).png';
import ResourceDictionaryInSolution from '/img/gitbook-import/assets/image (1) (4).png';
import MergedResourceDictionaryStructure from '/img/gitbook-import/assets/image (1) (3).png';

在应用中，你经常需要统一一些图形基础元素，例如画刷和颜色等。当然，资源并不限于这些。你可以在 Avalonia 应用的不同层级将它们定义为资源，也可以将它们放入独立文件中按需引入。

资源始终定义在资源字典中。这意味着每个资源都必须有一个键属性。

资源字典所在的层级决定了其中资源的作用域：资源在定义它们的文件中以及更下层的位置都可用。因此，你可以通过决定资源字典放置的位置，来精细控制资源的可见范围。

## 声明资源

例如，你可能希望整个应用中的画刷颜色保持统一。此时可以在应用的 XAML 文件 **App.axaml** 中声明资源字典，如下所示：

```xml title="App.axaml"
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyApp.App">
    // highlight-start
  <Application.Resources>
    <SolidColorBrush x:Key="Warning">Yellow</SolidColorBrush>
  </Application.Resources>
    // highlight-end
</Application>
```

另一种情况是，你只希望一组资源作用于某个特定窗口或用户控件。此时就应在对应的窗口或用户控件文件中定义资源字典。例如：

```xml title="MyUserControl.axaml"
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyApp.MyUserControl">
    // highlight-start
  <UserControl.Resources>
    <SolidColorBrush x:Key="Warning">LightYellow</SolidColorBrush>
  </UserControl.Resources>
    // highlight-end
</UserControl>
```

实际上，如果有需要，你还可以把资源定义在控件级别：

```xml title="MainWindow.axaml"
<Window xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyApp.MainWindow">
  <StackPanel>
    // highlight-start
    <StackPanel.Resources>
      <SolidColorBrush x:Key="Warning">PaleGoldenRod</SolidColorBrush>
    </StackPanel.Resources>
    // highlight-end
  </StackPanel>
</Window>
```

你还可以声明仅供某个样式专用的资源。

```xml title="MyStyle.axaml"
<Style Selector="TextBlock.warning">
  <Style.Resources>
    <SolidColorBrush x:Key="Warning">Yellow</SolidColorBrush>
  </Style.Resources>
  <Setter ... />
</Style>
```

:::note
请注意，这个资源在该样式块之外是不可见的，也就是说，它不会让所有带有 `warning` 类的 `TextBlock` 在 `Style` 块之外都能访问这个资源。
:::

你也可以为特定主题变体定义资源，例如 Dark、Light 或自定义主题。下面的示例中，`BackgroundBrush` 和 `ForegroundBrush` 会根据系统或应用当前设置的主题变体而取不同值。更多信息请参阅 [Theme Variants](/docs/styling/theme-variants)。

```xml
<ResourceDictionary>
  <ResourceDictionary.ThemeDictionaries>
    <ResourceDictionary x:Key='Light'>
      <SolidColorBrush x:Key='BackgroundBrush'>White</SolidColorBrush>
      <SolidColorBrush x:Key='ForegroundBrush'>Black</SolidColorBrush>
    </ResourceDictionary>
    <ResourceDictionary x:Key='Dark'>
      <SolidColorBrush x:Key='BackgroundBrush'>Black</SolidColorBrush>
      <SolidColorBrush x:Key='ForegroundBrush'>White</SolidColorBrush>
    </ResourceDictionary>
  </ResourceDictionary.ThemeDictionaries>
</ResourceDictionary>
```

## 资源字典文件

将资源字典定义到独立文件中，可以让你的 Avalonia 应用项目结构更加清晰，也更方便定位和维护资源定义。

位于资源字典文件中的资源可供整个应用访问。

要添加资源字典文件，请按以下步骤操作：

- 在希望创建新文件的位置右键单击项目。
- 点击 **Add**，然后点击 **New Item**。
- 在左侧列表中点击 **Avalonia**：

<Image light={AddNewItemDialog} alt="Add New Item dialog showing Avalonia resource dictionary templates" position="center" maxWidth={400} cornerRadius="true" />

- 点击 **Resource Dictionary (Avalonia)**。
- 输入你想使用的文件名。
- 点击 **Add**。

:::note
创建好资源文件后，你还需要正确地把它引入应用。详见 [引入并合并资源](#引入并合并资源) 一节。
:::

现在你就可以在指定位置添加需要定义的资源了，结构如下：

```xml
<ResourceDictionary xmlns="https://github.com/avaloniaui"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <!-- 在这里添加资源 -->
</ResourceDictionary>
```

## 使用资源

你可以通过 `{DynamicResource}` 标记扩展来使用当前作用域内资源字典中的资源。

例如，如果要把某个资源直接用于边框元素的背景属性，可以使用下面的 XAML：

```xml
<Border Background="{DynamicResource Warning}">
  Look out!
</Border>
```

### 静态资源

你也可以改用 `StaticResource` 标记扩展。例如：

```xml
<Border Background="{StaticResource Warning}">
  Look out!
</Border>
```

静态资源的不同之处在于：它不会响应代码中对资源所做的运行时修改。一旦加载完成，静态引用的值就不会再改变。

使用静态资源的好处是运行期开销更小，因此加载会略快一些，占用的内存也会稍少一些。

## 资源优先级

Avalonia 在解析资源时，会从 `DynamicResource` 或 `StaticResource` 所在的位置开始，沿着 **逻辑控件树** 向上查找对应的资源键。

这意味着：当存在同名资源键时，离资源引用位置越近的定义优先级越高。也就是说，逻辑控件树上更高层定义的资源，会被更靠近引用点的资源“覆盖”。例如下面这段 XAML：

```xml
<UserControl ... >
  <UserControl.Resources>
    <SolidColorBrush x:Key="Warning">Yellow</SolidColorBrush>
  </UserControl.Resources>

  <StackPanel>
    <StackPanel.Resources>
      <SolidColorBrush x:Key="Warning">Orange</SolidColorBrush>
    </StackPanel.Resources>

    <Border Background="{DynamicResource Warning}">
      Look out!
    </Border>
  </StackPanel>
</UserControl>
```

这里的边框控件使用了键为 `Warning` 的资源。这个资源被定义了两次：一次在外层 `StackPanel` 上，一次在 `UserControl` 层。Avalonia 会判断该边框的背景应该是橙色，因为从边框自身开始沿逻辑控件树向上搜索时，会先找到父级 `StackPanel` 上的定义。

## 引入并合并资源

资源可以从资源字典文件中引入，并与其他文件中定义的资源合并使用（即使另一个文件本身没有资源也可以）。

<Image light={ResourceDictionaryInSolution} alt="Solution Explorer showing resource dictionary file included in a project" position="center" maxWidth={400} cornerRadius="true" />

如果你想在整个应用级别合并资源字典，就需要在应用 XAML 文件 **App.axaml** 的 **Application.Resources** 节中声明一个资源字典，如下所示：

```xml
<Application.Resources>
  <ResourceDictionary>
    <ResourceDictionary.MergedDictionaries>
      <MergeResourceInclude Source="/Assets/AppResources.axaml" />
    </ResourceDictionary.MergedDictionaries>
  </ResourceDictionary>
</Application.Resources>
```

你也可以通过合并资源字典，让这些合并后的资源仅服务于某个特定样式。

<Image light={MergedResourceDictionaryStructure} alt="Merged resource dictionary structure in a styles file" position="center" maxWidth={400} cornerRadius="true" />

这意味着你可以在一个文件中定义样式，在另一个文件中定义资源并供其使用。这样既能保持样式一致性，也能让应用结构更清晰、更易维护。

要在样式文件中引入另一个文件里的资源字典，请添加如下 XAML：

```xml
<Styles.Resources>
  <ResourceDictionary>
    <ResourceDictionary.MergedDictionaries>
      <ResourceInclude Source="/Assets/AppResources.axaml"/>
    </ResourceDictionary.MergedDictionaries>
  </ResourceDictionary>
</Styles.Resources>
```

在上面的示例中，资源文件 `AppResources.axaml` 位于项目的 `/Assets` 文件夹中。接下来你就可以在样式中使用这些资源，例如：

```xml
<Style Selector="Button.btn-info">
  <Setter Property="Background" Value="{StaticResource InfoColor}"/>
</Style>
```

其中资源 `InfoColor` 是在引入文件中定义的一个 `SolidColorBrush`。

:::info
请注意，这里使用 `StaticResource` 来引用资源，因为该资源不应发生变化——这里的目标是保持样式一致性。
:::

## 合并资源的优先级

正如前文所述，资源的解析过程会从标记扩展出现的位置开始，沿逻辑控件树向上搜索，直到找到所请求的资源键。

不过，当应用的不同层级中同时存在样式和合并字典时，还会引入额外的优先级规则：

* 控件资源 -> 合并字典
* 样式资源 -> 合并字典
* 应用资源 -> 合并字典

例如，在下面这个理论上的应用结构中，搜索底部边框控件所使用资源的顺序，会按照方括号 `[]` 中标注的顺序进行：

```text
Application
 |- Resources [11]
     |- Merged dictionary [12]
     |- Merged dictionary [13]
 |- Styles
     |- Resources [14]
         |- Merged dictionary [15]
         |- Merged dictionary [16]

Window
 |- Resources [6]
     |- Merged dictionary [7]
 |- Styles
     |- Resources [8]
         |- Merged dictionary [9]
         |- Merged dictionary [10]
 |- StackPanel
     |- Resources [1]
         |- Merged dictionary [2]
         |- Merged dictionary [3]
     |- Styles
         |- Resources [4]
             |- Merged dictionary [5]
     |- Border
```

从边框开始，首先搜索的是父级控件（`StackPanel`）上直接定义的资源。之后才会按它们在 XAML 中出现的顺序，检查同层级的合并字典。

接着，搜索会继续检查父级控件（`StackPanel`）中定义的样式资源，再检查该层级下对应的合并字典。

之后，搜索会继续沿逻辑控件树向上进行，并在每一层重复类似的流程，最终到达应用级别的资源与样式。

## 在代码中使用资源

Avalonia 提供了多种在代码中访问资源的方式。

:::note

下面示例中的 `ResourceNode` 可以是任何支持 `Resource` 的节点，例如 `Application.Current`、`Window`、`UserControl` 等。

:::

- **ResourceNode.Resources["TheKey"]**: <br/>
  直接访问底层 `Dictionary`。请注意：它不会扫描合并字典，也不会向父级查找。
- **ResourceNode.TryGetResource**: <br/>
  尝试获取指定资源，成功时返回 `true`，否则返回 `false`。它会扫描合并字典，但不会沿逻辑树继续查找。
- **ResourceNode.TryFindResource**:  <br/>
  这是一个扩展方法，会尝试获取指定资源，成功时返回 `true`，否则返回 `false`。它既会扫描合并字典，也会沿逻辑树继续查找。
- **ResourceNode.GetResourceObservable**: <br/>
  返回一个 [`IObservable`](https://learn.microsoft.com/en-us/dotnet/api/System.IObservable-1)，可以用来观察资源变化。例如，你可以把它直接用于绑定。

```csharp
// 在这个示例中，我们在 App.axaml 中定义了资源，
// 并希望在 MainWindow 构造函数中查找它的值。
//
//    <Application.Resources>
//         <x:String x:Key="TheKey">HelloWorld</x:String>
//    </Application.Resources>

public MainWindow()
{
    InitializeComponent();

    // found1 = false | result1 = null
    var found1 = this.TryGetResource("TheKey", this.ActualThemeVariant, out var result1);

    // found2 = true | result2 = "Hello World" 
    var found2 = this.TryFindResource("TheKey", this.ActualThemeVariant, out var result2);

    // 在代码后置中把资源绑定到 TextBlock
    myTextBlock.Bind(TextBlock.TextProperty, Resources.GetResourceObservable("TheKey"));

    // 这会通过已绑定的 observable 更新 myTextBlock.Text
    this.Resources["TheKey"] = "Hello from code behind";
}
```

## 另请参阅

- [Resources Overview](/docs/app-development/resources)：了解资源类型与查找行为。
- [Theme Variants](/docs/styling/theme-variants)：使用具备主题感知能力的资源。
