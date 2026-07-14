---
id: native-aot
title: Native AOT
description: 使用预先编译将 Avalonia 应用发布为原生可执行文件。
doc-type: how-to
---

Native AOT（Ahead-of-Time，预先编译）可让你将 Avalonia 应用发布为自包含的可执行文件，并获得接近原生的性能表现。本指南介绍 Avalonia 在 Native AOT 部署中的专属注意事项与配置方式。

## Avalonia 应用的优势

Native AOT 编译可为 Avalonia 应用带来以下优势：

- 更快的应用启动速度，尤其适合桌面应用
- 更低的内存占用，适合资源受限的环境
- 自包含部署，无需额外安装 .NET 运行时
- 通过缩小攻击面提升安全性（无 JIT 编译）
- 与裁剪结合使用时可减小分发体积

## 为 Avalonia 配置 Native AOT

### 项目配置

将以下内容添加到你的 csproj 文件中：

```xml
<PropertyGroup>
    <PublishAot>true</PublishAot>
    <!-- Avalonia 12.0 之前需要，曾用于无障碍 API -->
    <BuiltInComInteropSupport>false</BuiltInComInteropSupport>
</PropertyGroup>
```

## Avalonia 专属注意事项

### XAML 加载
使用 Native AOT 时，XAML 会在构建阶段编译进应用程序。请确保：
- 在 XAML 文件中使用 `x:CompileBindings="True"`
- 避免在运行时动态加载 XAML
- 在可行时优先使用静态资源引用，而非动态资源

### 资源与资产
- 将所有资源作为嵌入资源打包
- 为资源文件使用 `AvaloniaResource` 构建操作
- 避免从外部来源动态加载资源

### 视图模型与依赖注入
- 在启动时注册视图模型
- 使用编译期依赖注入配置
- 避免基于反射的服务定位方式

## 发布 Avalonia Native AOT 应用

### Windows
```bash
dotnet publish -r win-x64 -c Release
```

### Linux
```bash
dotnet publish -r linux-x64 -c Release
```

### macOS
基于 Intel 的 macOS
```bash
dotnet publish -r osx-x64 -c Release
```

基于 Apple Silicon 的 macOS
```bash
dotnet publish -r osx-arm64 -c Release
```

:::tip
之后你可以使用 Apple 的 [lipo 工具](https://developer.apple.com/documentation/apple-silicon/building-a-universal-macos-binary) 合并 Intel 和 Apple Silicon 的二进制文件，以便发布通用二进制版本。
:::

## 常见问题排查

### 1. 与反射相关的错误
对于使用反射的视图模型或服务：
```xml
<ItemGroup>
    <TrimmerRootDescriptor Include="TrimmerRoots.xml" />
</ItemGroup>
```

创建一个 `TrimmerRoots.xml`：
```xml
<linker>
    <assembly fullname="YourApplication">
        <type fullname="YourApplication.ViewModels*" preserve="all"/>
    </assembly>
</linker>
```

## 已知限制

在 Avalonia 中使用 Native AOT 时，请注意以下限制：
- 动态创建控件必须在裁剪设置中显式配置
- 部分第三方 Avalonia 控件可能与 AOT 不兼容
- 平台特定功能需要额外显式配置
- 设计时工具中的实时预览能力可能受限

## 平台支持

有关平台支持，请参阅[平台/架构限制](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/#platformarchitecture-restrictions)。

## 另请参阅

- [Native AOT 部署](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/?tabs=windows%2Cnet9plus#platformarchitecture-restrictions)：Microsoft 提供的 Native AOT 文档。
- [使用 Native AOT 的 Avalonia 示例应用](https://github.com/AvaloniaUI/Avalonia.Samples)：示例项目。
