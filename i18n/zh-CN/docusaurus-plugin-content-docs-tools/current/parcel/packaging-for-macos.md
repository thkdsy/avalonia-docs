---
id: packaging-for-macos
title: 为 macOS 打包应用
sidebar_label: macOS
doc-type: reference
tags:
  - avalonia plus
  - avalonia pro
  - avalonia enterprise
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## 打包

Parcel 会创建 macOS 应用程序包（.app），可通过 DMG 镜像或 ZIP 压缩包分发。该工具会处理 bundle 结构、Info.plist 生成以及文件权限。Parcel 也可以在 Windows 和 Linux 平台上创建 macOS bundle。

### Bundle 配置

#### 通用属性

**Application Name**:

作为 `CFBundleDisplayName` 使用的应用显示名称。

:::note
目前无法本地化。
:::

**Package Name**:

作为 bundle 名称和输出 dmg 文件名使用的包名。

### Bundle 属性

这些是定义应用在 macOS 上如何展示和运行的核心 bundle 元数据。

**Bundle Identifier**:

应用程序的唯一反向 DNS 标识符（例如 `com.Company.AppName`）。它必须遵循 Apple 的反向 DNS 命名规则，避免使用点号和连字符之外的特殊字符，并确保标识符以字母开头。

**Team ID**:

Apple Developer 账户的唯一标识符。在签名和公证过程中会使用到；其他情况下为可选项。

**App Category**:

用于 macOS 和 App Store 分类的应用类别。它会映射到 Apple 的 `public.app-category.*` 标识符。

**App Icon**:

应用图标，支持 **ICNS** 或 **SVG** 格式。ICNS 文件应包含从 16x16 到 1024x1024 像素的多个分辨率。Parcel 会根据源文件自动生成 bundle 图标结构。

**Permissions**:

带自定义用途说明的系统权限。每项权限都需要提供用途描述，该描述会显示在 macOS 的权限对话框中。

:::note
用途说明是必填项；否则操作系统可能会拒绝访问相关系统资源。
:::

<!--- NOT YET AVAILABLE IN STABLE PARCEL
**File Type Associations**:

通过指定文件扩展名（例如 `.myfile`）并可选地添加 MIME 类型，将应用与特定文件类型关联。

如需在 Avalonia 应用中处理这些文件，请参阅 [Activatable Lifetime](https://docs.avaloniaui.net/docs/concepts/services/activatable-lifetime#handling-uri-activation) 文档。

**URL Scheme Handlers**:

通过定义自定义协议（例如 `myapp://`、`myprotocol://`）来注册用于深度链接的自定义 URL scheme。这使其他应用程序能够携带特定参数启动你的应用。

如需在 Avalonia 应用中处理 URL scheme，请参阅 [Activatable Lifetime](https://docs.avaloniaui.net/docs/concepts/services/activatable-lifetime#handling-uri-activation) 文档。
--->

#### 自定义 Info.plist 配置

Parcel 支持使用自定义 Info.plist 文件进行更高级的 bundle 配置。

1. 在项目根目录中创建 `Info.plist` 文件
2. 按照 Apple 文档添加自定义键和值
3. Parcel 会将自定义属性与自动生成的属性合并
4. 自定义文件中已存在的属性会优先生效
5. 缺失的属性会根据项目配置自动补齐

### 创建 DMG

Parcel 可创建带拖放界面、自定义背景和符号链接的 DMG 安装包。

:::caution
在 Windows 上创建 DMG 需要 [WSL2](https://learn.microsoft.com/en-us/windows/wsl/)。创建 ZIP 包则不需要 WSL2。
:::

**DMG Background**:

DMG 安装器的背景图片，使用 TIFF 格式。

Parcel 使用固定大小为 **660x422** 像素的 DMG 窗口，并采用如下布局：

- **应用 Bundle 图标**：位于坐标 (180, 170)，图标尺寸为 160px
- **Applications 文件夹**：位于坐标 (480, 170)，图标尺寸为 160px
- **文本大小**：图标标签为 12px

图标的位置坐标是从左上角计算到图标中心点。

设计背景图时，请考虑这些固定位置以及拖放操作流程。

:::note
目前 DMG 的自定义能力仅限于背景图片。未来版本计划提供更灵活的编辑器。
:::

### 创建 ZIP

Parcel 在创建 ZIP 时会保留可执行权限。解压到 macOS 后，bundle 结构会保持完整，应用也无需额外处理即可执行。

### 故障排查

参阅 [macOS 故障排查页面](/troubleshooting/platform-specific-issues/macos#packaging)。

## 代码签名

Parcel 使用 Apple Developer 证书为 macOS bundle 签名，并支持在 Windows、Linux 和 macOS 平台上进行跨平台签名。

### 前提条件

在为 macOS 应用签名之前，请确保你具备以下条件：

- **Apple Developer Account**：有效的 [Apple Developer Program](https://developer.apple.com/programs/) 会员资格（99 美元/年）
- **Xcode Command Line Tools**（仅 macOS）：可从 [Apple Developer Resources](https://developer.apple.com/xcode/resources/) 获取

### 签名方式

Parcel 会根据开发环境和工作流支持多种证书格式。

#### KeyChain Identity（仅 macOS）

使用通过证书请求安装到 macOS Keychain 中的证书。

如果要在 Mac App Store 之外分发应用，则需要使用与你的团队 ID 关联的 “Developer ID Application” 证书。

#### P12 证书（跨平台）

一种同时包含证书和私钥的可移植证书格式。
Apple 不直接提供 P12 证书，但你可以从 Keychain 导出它，或者使用 OpenSSL 生成它。

Parcel 在 Windows 和 Linux 机器上使用 [rcodesign](https://github.com/indygreg/apple-platform-rs/tree/main/apple-codesign) 对二进制文件和 bundle 进行签名。

### 创建开发者证书

<Tabs>
<TabItem value="keychain" label="Keychain (macOS Only)" default>

初次设置需要一台 macOS 机器。

**使用 Keychain 创建证书：**

1. 在 macOS 上打开 **Keychain Access**
2. 进入 **Keychain Access** > **Certificate Assistant** > **Request a Certificate From a Certificate Authority**
3. 在 Common Name 字段中输入名称，CA Email Address 保持为空
4. 选择 **Saved to disk**，然后点击 **Continue** 生成 `certificate.csr`
5. 打开 [Apple Developer Account](https://developer.apple.com/account/) > **Certificates, Identifiers & Profiles**
6. 进入 **Certificates** > **All Certificates**
7. 点击 ➕ 创建新证书
8. 如果应用将在 App Store 外分发，请选择 **Developer ID Application**
9. 按提示上传 `certificate.csr`
10. 下载生成的 `.cer` 文件
11. 将证书导入 Keychain

:::tip
将证书导出为 P12，这样在完成此步骤后即可进行跨平台签名，而无需继续依赖 macOS。
:::

</TabItem>

<TabItem value="openssl" label="OpenSSL (Cross-Platform)">

可在任意平台使用 OpenSSL 生成证书。

**前提条件：**

- 已安装 OpenSSL（Windows 推荐配合 WSL2 使用）

**使用 OpenSSL 创建证书：**

1. 创建私钥：

    ```bash
    openssl genrsa -out private.key 2048
    ```

2. 生成证书签名请求：

    ```bash
    openssl req -new -key private.key -out certificate.csr
    ```

3. 将 CSR 上传到 [Apple Developer Portal](https://developer.apple.com/account/)
    - Go to **Certificates, Identifiers & Profiles** > **Certificates**
    - Click ➕, choose **Developer ID Application**
    - Upload `certificate.csr`, then download the `.cer` file

4. 将证书转换为 PEM 格式：

    ```bash
    openssl x509 -in development.cer -inform DER -out certificate.pem -outform PEM
    ```

5. 创建 P12 文件（需要使用前面生成的 `private.key` 文件）：

    ```bash
    openssl pkcs12 -export -out certificate.p12 -inkey private.key -in certificate.pem
    ```

    系统提示时请设置一个安全密码。

:::tip
生成的 `certificate.p12` 和对应密码可在任意平台上配合 Parcel 使用。
:::

</TabItem>

</Tabs>

## 故障排查

参阅 [macOS 故障排查页面](/troubleshooting/platform-specific-issues/macos#code-signing)。

## 公证

Apple 公证会验证应用是否已由 Apple 检查过恶意软件。在 macOS 10.15（Catalina）及更高版本上，如果应用在 Mac App Store 之外分发，就需要进行公证。

该流程会将应用上传到 Apple 服务器进行扫描，并将 bundle 哈希与开发者账户关联起来。

### 前提条件

在为应用进行公证之前，请确保你具备以下条件：

- **Apple Developer Account**：付费 Apple Developer Program 会员资格（99 美元/年）
- **Valid Developer ID Certificate**：用于为在 Mac App Store 之外分发的应用进行代码签名
- （仅 macOS）**Xcode Command Line Tools**：可从 [Apple Developer Resources](https://developer.apple.com/xcode/resources/) 获取

### Apple 账户身份验证

Parcel 需要通过 Apple 的公证服务进行身份验证。目前有两种凭据提供方式可用。

#### App-Specific Password（推荐）

Apple 要求 Notary API 使用应用专用密码，而不是用户登录密码。请参考 Apple 指南：[How to generate an app-specific password](https://support.apple.com/en-us/102654)。

**在 Parcel 中配置凭据：**

1. 选择 “Apple Account” 作为公证凭据方式
2. 输入你的 Apple ID（邮箱地址）
3. 输入你的应用专用密码
4. 输入你的 Team ID（可在 [Apple Developer Membership](https://developer.apple.com/account/#/membership) 页面查看）

:::tip
建议使用环境变量存储凭据，而不是将其硬编码到配置文件中。
:::

#### Keychain Profile（仅 macOS）

将 Apple 账户凭据存储到 macOS Keychain 中，并通过配置文件名引用它们。凭据会被加密并保存在本地。

**设置 keychain 配置文件：**

1. 打开 Terminal
2. 运行以下命令：

    ```bash
    xcrun notarytool store-credentials "MyParcelProfile" --apple-id "your-email@example.com" --team-id "YOUR_TEAM_ID"
    ```

3. 在提示时输入应用专用密码：

    ```text
    App-specific password for your-email@example.com: [enter your app-specific password]
    Credentials saved to Keychain.
    To use them, specify `--keychain-profile "MyParcelProfile"`
    ```

**在 Parcel 中配置 keychain 配置文件：**

1. 选择 “Keychain Profile” 作为公证凭据方式
2. 输入配置文件名称（例如 “MyParcelProfile”）

:::caution
Apple Keychain 仅在 macOS 上可用。在 Windows 或 Linux 上请使用 App-Specific Password 方式。
:::
### 运行未公证应用（测试与个人使用）
对于测试、开发，或没有 Apple Developer Account 的个人使用场景，未公证应用仍可在用户手动干预下运行。
当 macOS 阻止未公证应用运行时，用户可以按以下方式绕过警告：

1. 打开 **System Preferences** → **Security & Privacy** → **General** 选项卡
2. 尝试运行应用，系统会阻止它
3. 几分钟内，Security & Privacy 中会出现有关被阻止应用的提示
4. 点击该提示旁边的 **"Open Anyway"**
5. 在弹窗中点击 **"Open"** 进行确认

:::note
即使暂时不做公证，也建议在条件允许时使用 Developer ID 证书对应用进行签名。
:::
### 公证问题排查

参阅 [macOS 故障排查页面](/troubleshooting/platform-specific-issues/macos#notarization)。

## 另请参阅

- [Parcel 安装](/tools/parcel/setup)
- [Parcel 命令行参考](/tools/parcel/command-line-reference)
