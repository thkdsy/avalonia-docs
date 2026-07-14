---
id: ios
title: iOS
description: 为 iPhone、iPad 和 Mac Catalyst 构建、签名并分发 Avalonia 应用。
doc-type: how-to
---

## 在模拟器上运行

在 iOS 项目目录中，使用以下命令进行构建和运行：

```bash
dotnet build
dotnet run
```

这会将应用部署到默认的 iOS 模拟器中。如果你使用的是 JetBrains Rider 或 Visual Studio for Mac，也可以直接在 IDE 中运行、构建和调试。

:::info
根据 .NET 版本和 iOS 模拟器版本的不同，在 Apple Silicon Mac 上可能需要安装 Rosetta 2。安装命令如下：

```bash
/usr/sbin/softwareupdate --install-rosetta
```
:::

## 在真机上运行

如果要部署到真实的 iPhone 或 iPad，首先必须使用 Xcode 对设备进行配置。这包括创建签名证书，并将设备关联到一个开发用 provisioning profile。

有关如何使用 Xcode 配置设备的详细步骤，包括创建免费的 Apple 开发者签名证书，请参阅 [iOS platform setup guide](/docs/platform-specific-guides/ios)。

设备配置完成后，请编辑 `.iOS.csproj`，设置运行时标识符和代码签名密钥：

```xml
<RuntimeIdentifier>ios-arm64</RuntimeIdentifier>
<CodesignKey>Apple Development: yourname@example.com (XXXXXXXXXX)</CodesignKey>
```

之后按正常方式构建并运行即可。应用会被部署到你连接的设备上。

## 发布

将 Avalonia 应用发布到 iOS 平台时，会生成一个可用于分发的 `.ipa` 文件。分发 iOS 应用要求应用必须使用 provisioning profile 进行签名，该 profile 中包含代码签名信息以及目标分发方式。

### 前提条件

- 一台已安装 Xcode 的 Mac（iOS 应用必须在 macOS 上构建）
- 一个 [Apple Developer Program](https://developer.apple.com/programs/) 会员账号（分发应用时必需）
- 已在 Xcode 中配置好的 provisioning profile 和签名证书

### 分发方式

Apple 提供三种 iOS 应用分发方式：

- **App Store**：通过 [App Store Connect](https://appstoreconnect.apple.com) 提交。应用需要通过 Apple 审核。这是面向最终用户最常见的方式。
- **Ad-hoc**：最多可分发到 100 台已注册设备用于测试。适用于 Apple Developer Program 成员。
- **In-house（企业内部分发）**：在组织内部进行分发。需要加入 [Apple Developer Enterprise Program](https://developer.apple.com/programs/enterprise/)。

以上所有方式都要求应用使用合适的 provisioning profile 完成签名。

### 构建并签名应用

#### 在 macOS 上发布

进入 iOS 项目目录并运行 `dotnet publish`：

```bash
dotnet publish -f net9.0-ios -c Release \
  -p:ArchiveOnBuild=true \
  -p:RuntimeIdentifier=ios-arm64 \
  -p:CodesignKey="Apple Distribution: John Smith (AY2GDE9QM7)" \
  -p:CodesignProvision="MyAvaloniaApp"
```

这会完成构建和签名，并在 `bin/Release/net9.0-ios/ios-arm64/publish/` 中生成 `.ipa` 文件。

具体的分发渠道由 provisioning profile 中使用的分发证书决定（例如 App Store、Ad Hoc 或 Enterprise）。

#### 在 Windows 上发布

在 Windows 上构建 iOS 应用时，需要一个可通过网络访问的 Mac 构建主机。你需要通过额外参数提供连接信息：

```bash
dotnet publish -f net9.0-ios -c Release \
  -p:ArchiveOnBuild=true \
  -p:RuntimeIdentifier=ios-arm64 \
  -p:CodesignKey="Apple Distribution: John Smith (AY2GDE9QM7)" \
  -p:CodesignProvision="MyAvaloniaApp" \
  -p:ServerAddress=192.168.1.100 \
  -p:ServerUser=macuser \
  -p:ServerPassword=mypassword \
  -p:TcpPort=58181 \
  -p:_DotNetRootRemoteDirectory=/Users/macuser/Library/Caches/Xamarin/XMA/SDKs/dotnet/
```

### 构建属性参考

以下属性既可以通过命令行中的 `-p:` 传入，也可以写入项目文件中的 `<PropertyGroup>`：

| 属性 | 说明 |
|---|---|
| `ArchiveOnBuild` | 设为 `true` 时会生成 `.ipa`。 |
| `RuntimeIdentifier` | 目标运行时。应使用 `ios-arm64`。 |
| `CodesignKey` | 代码签名密钥名称（例如 `Apple Distribution: Name (ID)`）。 |
| `CodesignProvision` | 签名时使用的 provisioning profile 名称。 |
| `CodesignEntitlements` | entitlements 文件路径（如果需要）。 |
| `ApplicationTitle` | 用户可见的应用名称。 |
| `ApplicationId` | 唯一标识符，例如 `com.companyname.myapp`。 |
| `ApplicationVersion` | 构建版本号。 |
| `ApplicationDisplayVersion` | 显示给用户的版本字符串。 |

#### 在项目文件中定义属性

你也可以不把全部参数写在命令行中，而是直接在 `.csproj` 中设置：

```xml
<PropertyGroup Condition="$(TargetFramework.Contains('-ios')) and '$(Configuration)' == 'Release'">
    <ArchiveOnBuild>true</ArchiveOnBuild>
    <CodesignKey>Apple Distribution: John Smith (AY2GDE9QM7)</CodesignKey>
    <CodesignProvision>MyAvaloniaApp</CodesignProvision>
</PropertyGroup>
```

之后只需执行：
```bash
dotnet publish -f net9.0-ios -c Release
```

### 分发应用

- **App Store**：使用 [Transporter](https://apps.apple.com/us/app/transporter/id1450874784?mt=12) 或 Xcode 上传 `.ipa`。在此之前，你必须先在 [App Store Connect](https://appstoreconnect.apple.com) 中创建应用记录，并生成一个 [app-specific password](https://support.apple.com/HT204397)。
- **Ad-hoc**：使用 [Apple Configurator](https://apps.apple.com/app/id1037126344) 进行分发。
- **In-house**：通过安全网站或移动设备管理（MDM）进行分发。详情请参阅 [Distribute proprietary in-house apps](https://support.apple.com/guide/deployment/depce7cefc4d/web)。

## Mac Catalyst 部署

如果你的 iOS 项目面向的是 Mac Catalyst（`net10.0-maccatalyst`），那么你可以使用同一个项目为 macOS 构建和发布应用。配置说明请参阅 iOS 平台指南中的 [Mac Catalyst](/docs/platform-specific-guides/ios#mac-catalyst)。

发布 Mac Catalyst 应用时：

```bash
dotnet publish -f net10.0-maccatalyst -c Release
```

Mac Catalyst 应用可以通过 Mac App Store 分发，也可以作为已签名的 `.app` 包分发。签名与分发流程与 iOS 基本一致，同样依赖 Apple Developer 证书和 provisioning profile。

## 另请参阅

- [iOS platform setup](/docs/platform-specific-guides/ios)
