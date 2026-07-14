---
id: including-assets
title: 资源文件
description: 在 Avalonia 应用程序中包含并引用位图、样式和资源字典等资源文件。
doc-type: reference
---

import AssetFileDiagram from '/img/concepts/ui-concepts/assets/asset-file.png';
import AssetLibraryDiagram from '/img/concepts/ui-concepts/assets/asset-library.png';

许多应用程序都需要包含位图、样式和资源字典等资源文件。资源字典中可以包含在 XAML 中声明的图形基础资源。样式也可以写成 XAML，而位图资源则通常是二进制文件，例如 PNG 和 JPEG 格式。

## 包含资源文件

<Image light={AssetFileDiagram} alt="Diagram showing how asset files are included in an Avalonia project" position="center" maxWidth={400} cornerRadius="true"/>

你可以通过在项目文件中使用 `<AvaloniaResource>` 元素来把资源文件包含进应用程序中。

例如，Avalonia .NET Core MVVM App 解决方案模板会创建一个名为 `Assets` 的文件夹（其中包含 `avalonia-logo.ico` 文件），并在项目文件中添加一个元素，把该目录下的所有文件都包含进去，如下所示：

```xml
<ItemGroup>
  <AvaloniaResource Include="Assets\**"/>
</ItemGroup>
```

你也可以在这个项组中继续添加额外的 `<AvaloniaResource>` 元素，以包含任意你需要的文件。

:::tip
这里的元素名 `AvaloniaResource` 仅表示这些资源会在构建时以内嵌 .NET 资源的形式存储。不过从 Avalonia 的术语上讲，它们被称为 “Assets”，用来与 “XAML resources” 区分。
:::


### 引用已包含的资源

资源文件被包含之后，就可以在定义 UI 的 XAML 中按需引用它们。例如，可以通过相对路径来引用这些资源：

```xml
<Image Source="icon.png"/>
<Image Source="images/icon.png"/>
<Image Source="../icon.png"/>
```

此外，你也可以使用根路径：

```xml
<Image Source="/Assets/icon.png"/>
```

## 类库中的资源

<Image light={AssetLibraryDiagram} alt="Diagram showing how to reference assets from a library assembly" position="center" maxWidth={400} cornerRadius="true"/>

如果资源文件位于与当前 XAML 文件不同的程序集里，那么你需要使用 `avares:` URI 方案。例如，如果资源位于名为 `MyAssembly.dll` 的程序集中的 `Assets` 文件夹里，那么可以这样写：

```xml
<Image Source="avares://MyAssembly/Assets/icon.png"/>
```

### 资源类型转换

Avalonia 内置了一些转换器，可以开箱即用地加载位图、图标和字体等资源。因此，一个资源 URI 可以自动转换为以下任意类型：

* Image - `Image` 类型
* Bitmap - `Bitmap` 类型
* Window Icon - `WindowIcon` 类型
* Font - `FontFamily` 类型

### 在代码中加载资源

你也可以使用 `AssetLoader` 静态类，通过代码来加载资源。例如：

```csharp title='C#'
var bitmap = new Bitmap(AssetLoader.Open(new Uri(uri)));
```

上面代码中的 `uri` 变量可以包含任意有效的 `avares:` URI（如前文所述）。

Avalonia 本身并不支持 `file://`、`http://` 或 `https://` 这类 URI 方案。如果你想从磁盘或网络加载文件，就需要自己实现相关功能，或者使用社区提供的方案。

:::info
社区中有一个可用的图片加载实现：[AsyncImageLoader.Avalonia](https://github.com/AvaloniaUtils/AsyncImageLoader.Avalonia)。
:::

## 另请参阅

- [Avalonia XAML](/docs/fundamentals/avalonia-xaml)
- [UI composition](/docs/fundamentals/ui-composition)
