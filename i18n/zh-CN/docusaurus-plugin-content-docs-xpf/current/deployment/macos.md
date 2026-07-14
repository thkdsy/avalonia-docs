---
id: macos
title: macOS 部署
description: 如何在 macOS 上发布和部署 XPF 应用，包括 app bundle 结构和代码签名。
---

## 发布

从命令行发布您的 macOS XPF 应用：

```bash
dotnet publish -r osx-arm64 -c Release --self-contained
```

对于 Intel Mac：

```bash
dotnet publish -r osx-x64 -c Release --self-contained
```

:::caution
始终从命令行进行发布。Visual Studio 的发布可能会生成不完整的输出，缺少本机库。
:::

## App bundle 结构

macOS 应用必须打包为 `.app` bundles 才能分发。`.app` bundle 是一个具有以下结构的目录：

```text
MyApp.app/
  Contents/
    Info.plist
    MacOS/
      MyApp          (可执行文件或启动脚本)
    Resources/
      MyApp.icns     (应用图标)
```

### Info.plist

使用您的应用元数据创建一个 `Info.plist`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleName</key>
    <string>MyApp</string>
    <key>CFBundleDisplayName</key>
    <string>My Application</string>
    <key>CFBundleIdentifier</key>
    <string>com.yourcompany.myapp</string>
    <key>CFBundleVersion</key>
    <string>1.0.0</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0.0</string>
    <key>CFBundleExecutable</key>
    <string>MyApp</string>
    <key>CFBundleIconFile</key>
    <string>MyApp.icns</string>
    <key>NSHighResolutionCapable</key>
    <true/>
</dict>
</plist>
```

## 项目设置

以下 `.csproj` 设置对 macOS 部署很重要：

```xml
<PropertyGroup>
    <SelfContained>true</SelfContained>
    <RuntimeIdentifier>osx-arm64</RuntimeIdentifier>
</PropertyGroup>
```

:::danger
请 **不要** 将 `IncludeNativeLibrariesForSelfExtract` 设置为 `true`。这与 macOS 不兼容，并且会导致您的应用在运行时因“Failed to create CoreCLR”而失败。
:::

## 代码签名

所有 macOS 应用都必须进行代码签名后才能分发。在签名 XPF 应用时：

- **分别签名单个文件**，而不是对整个 bundle 签名。不要在 `codesign` 中使用 `--deep` 标志，因为它可能会遗漏文件或应用错误的权限。
- 在签名 `.app` bundle 之前，先签名所有 `.dylib` 文件和主可执行文件。

```bash
# 先签名单个二进制文件
find MyApp.app -name "*.dylib" -exec codesign --force --sign "Developer ID Application: Your Name" {} \;
codesign --force --sign "Developer ID Application: Your Name" MyApp.app/Contents/MacOS/MyApp

# 然后签名 bundle
codesign --force --sign "Developer ID Application: Your Name" MyApp.app
```

## 公证

对于在 Mac App Store 之外分发，Apple 要求应用必须经过公证。请使用 `notarytool`：

```bash
# 创建用于公证的 ZIP
ditto -c -k --keepParent MyApp.app MyApp.zip

# 提交进行公证
xcrun notarytool submit MyApp.zip --apple-id "you@example.com" \
    --team-id "YOUR_TEAM_ID" --password "app-specific-password" --wait

# 贴附公证票据
xcrun stapler staple MyApp.app
```

## 创建 DMG

要以 `.dmg` 磁盘镜像形式分发：

```bash
hdiutil create -volname "MyApp" -srcfolder MyApp.app -ov MyApp.dmg
```

## Parcel（预览版）

Avalonia **Parcel** 工具可以自动化整个 macOS 打包工作流，包括为 XPF 应用创建 `.app` bundle、代码签名、公证以及生成 `.dmg`。请联系 Avalonia 团队以获取预览版访问权限。

## Dock 可见性

要控制您的应用是否出现在 macOS Dock 中，请参阅 [macOS: Dock Visibility](/xpf/platforms/macos#dock-visibility)。

## 应用名称

要设置在 macOS 菜单栏中显示的名称（而不是 "Avalonia Application"），请参阅 [macOS: Application Name](/xpf/platforms/macos#application-name)。

## ReadyToRun

启用 ReadyToRun 以加快启动速度：

```xml
<PropertyGroup>
    <PublishReadyToRun>true</PublishReadyToRun>
</PropertyGroup>
```

详情请参阅 [Performance Optimization](/xpf/configuration/performance#reducing-startup-time-with-readytorun)。