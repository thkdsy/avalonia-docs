---
id: native-aot
title: 原生 AOT
description: 使用提前编译技术将 XPF 应用程序发布为原生可执行文件，包括裁剪要求和 XAML 类型保留。
doc-type: how-to
---

XPF 支持原生 AOT（提前编译）。与 WPF 不同，XPF 不使用 COM 封送处理，因此兼容 AOT 编译。使用第三方控件库的大型应用程序可以成功通过原生 AOT 编译。

## 项目配置

在 `.csproj` 中添加 `PublishAot`。

```xml
<PropertyGroup>
    <PublishAot>true</PublishAot>
</PropertyGroup>
```

## 发布

要发布你的应用程序，请在命令行中运行 `dotnet publish`：

```
dotnet publish -r <runtime> -c Release
```

例如，`dotnet publish -r osx-arm64 -c Release` 将为 Apple Silicon 设备发布应用程序。

更多信息，请参阅 .NET 文档站点上的[原生 AOT 部署](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/?tabs=windows%2Cnet8#publish-native-aot-using-the-cli)和 [dotnet publish](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-publish)。

## 原生 AOT 链接器的裁剪

要在 XPF 中使用 AOT，裁剪和链接必须保守。

默认情况下，XPF SDK 附带一个 `rd.xml` 根描述符，该描述符强制包含 XPF 运行时库，包括所有内置 WPF 程序集以及任何设置了 `Xpf.Sdk` 的用户可执行程序集。

但是，如果你的应用程序引用了第三方 WPF 库，则它们必须具有等效的裁剪配置。否则，原生 AOT 链接器可能会删除那些仅在 XAML 中被引用、但无法检测到正在使用的类型。

如果你因缺少类型而遇到运行时错误，请检查你是否使用了任何未附带根描述符的第三方库。为这些库添加一个根描述符到你的 `.csproj` 文件中。

```xml title=".csproj"
<ItemGroup>
    <ProjectReference Include="..\YourAssembly\YourAssembly.csproj" />
    <TrimmerRootAssembly Include="YourAssembly" />
</ItemGroup>
```

更多信息，请参阅 .NET 文档站点上的[裁剪](https://learn.microsoft.com/en-us/dotnet/core/deploying/trimming/prepare-libraries-for-trimming#csproj-file)。

## 另请参阅

- [原生 AOT (Avalonia)](/docs/deployment/native-aot)：标准 Avalonia 应用程序的 AOT 设置
- [Windows 部署](/xpf/deployment/windows)
- [macOS 部署](/xpf/deployment/macos)
- [Linux 部署](/xpf/deployment/linux)
- [性能优化](/xpf/configuration/performance)
