---
id: centralizing-multiple-xpf-projects
title: 集中管理多个 XPF 项目
description: 了解如何在单个仓库中集中管理 XPF SDK 版本和许可证密钥配置。
doc-type: guide
---

当你在单个仓库中管理多个 XPF 项目时，在每个 `.csproj` 文件中保持 SDK 版本和许可证密钥同步可能会变得繁琐且容易出错。通过集中这些设置，你可以确保一致性，并简化未来的升级。

## 集中管理 XPF SDK 版本

你可以在仓库根目录使用一个 `global.json` 文件，来一次性为所有项目固定 XPF SDK 版本。当 `Xpf.Sdk` 存在 `global.json` 条目时，MSBuild 会自动解析该版本，因此你只需要在一个地方更新版本号。

在仓库根目录创建（或更新）一个 `global.json` 文件：

```json title="global.json"
{
  "msbuild-sdks": {
    "Xpf.Sdk": "1.6.0"
  }
}
```

然后，在每个 `.csproj` 文件中引用 `Xpf.Sdk`，**不要**带版本号：

```xml title="MyApp.csproj"
<Project Sdk="Xpf.Sdk">
```

当你需要升级时，只需更改 `global.json` 中的版本，仓库中的每个项目都会在下一次构建时采用新版本。

## 许可证密钥

将许可证密钥直接存放在受源代码管理的文件中存在安全风险。相反，你可以在 `NuGet.config` 和 `.csproj` 文件中都引用一个环境变量，这样实际的密钥值就不会出现在你的仓库中。

:::tip
你可以将环境变量命名为任何你喜欢的名称。下面的示例使用 `XpfLicenseKey`。
:::

### 设置环境变量

添加一个名为 `XpfLicenseKey` 的环境变量，其值为你的许可证密钥：

- **Windows**：在开始菜单中搜索“环境变量”，然后通过系统图形界面添加该变量。
- **macOS**：运行 `launchctl setenv XpfLicenseKey [LICENSE_KEY]`。你需要在每次重启后重新运行此命令。
- **Linux**：环境变量通常设置在 `.bash_profile`、`.bashrc` 或 `/etc/environment` 中。

在你创建或更改该变量后，请重启任何已打开的终端会话和 IDE，以便它们获取新值。

### 更新 `nuget.config`

编辑 `nuget.config` 文件中的凭据部分，以引用该环境变量：

```xml title="nuget.config"
<packageSourceCredentials>
  <xpf>
    <add key="Username" value="license" />
    <add key="ClearTextPassword" value="%XpfLicenseKey%" />
  </xpf>
</packageSourceCredentials>
```

### 更新 `.csproj` 文件

编辑每个 `.csproj` 文件中的 `RuntimeHostConfigurationOption` 条目，使其从环境变量读取密钥：

```xml title="MyApp.csproj"
<ItemGroup>
  <RuntimeHostConfigurationOption Include="AvaloniaUI.Xpf.LicenseKey"
                                  Value="$(XpfLicenseKey)" />
</ItemGroup>
```

## 另请参阅

- [XPF 入门](/xpf/getting-started)
- [自定义初始化](/xpf/configuration/customizing-initialization)
- [性能配置](/xpf/configuration/performance)
- [版本控制](/xpf/version-info/versioning)