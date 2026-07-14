---
id: versioning
title: XPF 版本管理
description: 了解如何选择和配置 XPF 包版本，包括稳定版发布和夜间构建。
doc-type: guide
---

## 选择版本

对于生产环境发布，你应当使用最新的稳定版本。你可以在[发行说明](/xpf/version-info/release-notes)中找到每个稳定版发布的详细信息。对于日常开发，你可能更倾向于更新到最新的夜间构建，以便更快获得新功能和错误修复。

你可以在[XPF NuGet 服务器](https://xpf-nuget-feed.avaloniaui.net/packages/xpf.sdk)上浏览所有可用版本。

## 访问 NuGet 源

XPF NuGet 源需要身份验证。要登录 Web 门户或配置你的 NuGet 客户端，请使用以下凭据：

- **用户名：** `license`
- **密码：** 你的 XPF 许可证密钥

当你将该源添加到 `NuGet.config` 文件时，请提供相同的凭据，以便 `dotnet restore` 和 Visual Studio 可以自动拉取包。

## 为夜间构建配置源

XPF 的夜间构建可能依赖于 Avalonia 的预发布版本。为确保所有依赖项都能正确解析，请将以下包源添加到你的 `NuGet.config` 文件中（完整设置请参见[快速开始，第 2 步](/xpf/getting-started#step-2-add-a-nugetconfig)）：

```xml
<add key="api.nuget.org" value="https://api.nuget.org/v3/index.json" />
<add key="xpf" value="https://xpf-nuget-feed.avaloniaui.net/v3/index.json" />
<add key="avalonia-nightly" value="https://nuget-feed-all.avaloniaui.net/v3/index.json" />
```

`avalonia-nightly` 源仅在你使用 XPF 夜间构建时需要。如果你使用的是稳定版 XPF 发布，则可以省略它。

## 锁定特定版本

要将你的项目锁定到特定的 XPF 版本，请在项目文件或 `Directory.Build.props` 中设置 `XpfVersion` 属性：

```xml
<PropertyGroup>
  <XpfVersion>1.6.0</XpfVersion>
</PropertyGroup>
```

锁定版本可以避免意外升级，并确保团队中的每位开发者都使用相同的包进行构建。

## 另请参阅

- [发行说明](/xpf/version-info/release-notes)
- [缺失功能](/xpf/version-info/missing-features)
- [快速开始](/xpf/getting-started)