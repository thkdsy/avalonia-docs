---
id: packaging-for-windows
title: 为 Windows 打包应用
sidebar_label: Windows
doc-type: reference
tags:
  - avalonia plus
  - avalonia pro
  - avalonia enterprise
---

## 打包

Parcel 可创建 Windows 安装程序（NSIS 可执行文件）和便携式可执行文件，并通过安装包或 ZIP 压缩包进行分发。该工具会处理可执行文件配置、注册表项，并可在所有受支持的平台上创建 Windows 包。

### 包配置

#### 通用属性

**Application Name**:

用于应用安装目录、开始菜单项和快捷方式文件名的显示名称。

:::note
目前无法本地化。
:::

**Package Name**:

输出安装程序的文件名（不含扩展名）。

### NSIS 安装程序属性

Parcel 使用 NSIS（Nullsoft Scriptable Install System）创建轻量级、自解压并具有灵活安装选项的安装程序。

**Company**:

显示在 Windows 属性、安装器和系统对话框中的发布者名称。启用 **Create Company Folder** 后，该名称还会用于组织 Program Files 中的应用目录以及相关注册表项。

**Create Company Folder**:

启用后，应用程序会安装到公司专属子目录中：
- Admin install: `Program Files\[Company]\[Application Name]\`
- User install: `%LocalAppData%\[Company]\[Application Name]\`

这也会影响开始菜单快捷方式的位置，使其按 `Start Menu\Programs\[Company]\[Application Name]` 的层级进行组织。

默认值：false。

**Installer Icon**:

安装程序图标，支持 **ICO** 或 **SVG** 格式。ICO 文件应包含从 16x16 到 256x256 像素的多个分辨率。该图标会出现在 Windows 资源管理器中的安装程序可执行文件、安装过程界面以及 Windows 卸载列表中。

:::note
这与应用图标是分开的。

应用图标由 .csproj 文件中的标准 .NET `<ApplicationIcon>file.ico</ApplicationIcon>` 属性定义。
:::

**Requires Admin**:

控制安装程序是否需要管理员权限才能安装。

启用后（默认），应用程序会安装到 `Program Files`，并要求提升 User Account Control（UAC）权限。禁用后，应用程序会安装到当前用户的 `%LocalAppData%` 目录，无需提权。

默认值：true。

**Include Uninstaller**:

启用后，Parcel 会随应用一起包含卸载程序，并在 Windows 的 Settings > Apps & Features（旧版 Windows 中为 Control Panel > Programs and Features）中创建对应条目。

默认值：true。

**License File**:

安装过程中显示的可选许可证文件。支持格式：
- 纯文本（.txt）
- 富文本格式（.rtf）

安装过程中会在单独页面显示许可证内容，用户必须接受后才能继续。

<!--- NOT YET AVAILABLE IN STABLE PARCEL

**File Type Associations**:

通过指定文件扩展名（例如 `.myfile`），并可选地添加 MIME 类型和自定义图标，将应用与特定文件类型关联。这会创建相应注册表项，以便正确集成到 Windows 资源管理器中。

如需在 Avalonia 应用中处理这些文件，请参阅 [Activatable Lifetime](https://docs.avaloniaui.net/docs/concepts/services/activatable-lifetime#handling-uri-activation) 文档。

**URL Scheme Handlers**:

通过定义自定义协议（例如 `myapp://`、`myprotocol://`）来注册用于深度链接的自定义 URL scheme。这会创建相应注册表项，使其他应用程序和 Web 浏览器能够携带特定参数启动你的应用。

如需在 Avalonia 应用中处理 URL scheme，请参阅 [Activatable Lifetime](https://docs.avaloniaui.net/docs/concepts/services/activatable-lifetime#handling-uri-activation) 文档。

--->

## 代码签名

Parcel 使用 Authenticode 证书为 Windows 可执行文件和安装程序签名，并支持在 Windows、Linux 和 macOS 平台上进行跨平台签名。

:::note
本文说明了如何将不同签名方式集成到 Parcel 中。它不包含获取证书或配置云签名服务的详细步骤；完整设置说明请参考各方法所链接的文档。
:::

### 前提条件

在为 Windows 应用签名之前，请确保你具备以下条件：

- **Code Signing Certificate**：由受信任证书颁发机构签发的有效 Authenticode 证书
- **Windows SDK**（仅 Windows）：可通过 [Visual Studio Downloads](https://visualstudio.microsoft.com/downloads/) 安装 Visual Studio Build Tools（CI 场景）或 Visual Studio（桌面场景）获取
- **Java Runtime**：跨平台签名操作所必需

### 签名方式

Parcel 会根据开发环境和工作流支持多种证书格式。

#### 本地证书

使用本地证书文件（PFX/P12 格式）进行签名。该方法不推荐用于生产应用，并且通常不会被 Windows 默认信任。

**所需配置：**
- **Local Signing Certificate File**：PFX 或 P12 证书文件路径
- **Local Signing Certificate Password**：保护证书文件的密码
- **Timestamp Server URL**（可选）：时间戳服务器地址（例如 `http://timestamp.digicert.com`）

**平台支持：**
- **Windows**：优先使用原生 SignTool，不可用时改用 JSign
- **Linux/macOS**：使用 [JSign](https://github.com/ebourg/jsign)（需要 Java Runtime）

**文档：**
- [New-SelfSignedCertificate](https://learn.microsoft.com/en-us/powershell/module/pki/new-selfsignedcertificate?view=windowsserver2025-ps)

#### Windows 证书存储

使用已安装在 Windows Certificate Store 中的证书，包括硬件安全模块（HSM）和 USB 令牌中的证书。

**所需配置：**
- **Store Certificate Name**：证书的主题名称或指纹
- **Use Local Machine Certificate Store**（可选）：在 Local Machine 存储而不是 Current User 中查找
- **Auto-Detect Matching Certificate**（可选）：自动选择最匹配的证书

**平台支持：**
- **仅 Windows**：需要 Windows SDK（SignTool）

**文档：**
- [Windows Certificate Store Overview](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/certificate-stores)

#### Microsoft Trusted Signing（跨平台）

一种基于云的签名服务，无需管理本地证书即可提供最高级别的信任。它可立即获得 SmartScreen 信任，并通过硬件安全模块（HSM）提升安全性。

**所需配置：**
- **Trusted Signing Endpoint**：Azure Trusted Signing 服务端点 URL（格式：`https://[region].codesigning.azure.net/`）
- **Certificate Profile Name**：Azure Trusted Signing 中的证书配置名称
- **Code Signing Account Name**：Azure Trusted Signing 账户名称

**身份验证：**
通过 Azure CLI 或环境变量进行身份验证：
- `AZURE_TENANT_ID`: Azure Active Directory tenant ID
- `AZURE_CLIENT_ID`: Service principal client ID
- `AZURE_CLIENT_SECRET`: Service principal client secret

**所需 Azure 角色**："Code Signing Certificate Profile Signer"

**平台支持：**
- **Windows**：优先使用原生 SignTool，不可用时改用 JSign
- **Linux/macOS**：使用 [JSign](https://github.com/ebourg/jsign)（需要 Java Runtime）

:::tip
对于希望无需长期积累信誉就获得立即信任的企业而言，Microsoft Trusted Signing 是推荐方案。
:::

**文档：**
- [Microsoft Trusted Signing Documentation](https://learn.microsoft.com/en-us/azure/trusted-signing/)
- [Trusted Signing Quickstart](https://learn.microsoft.com/en-us/azure/trusted-signing/quickstart)

#### Azure Key Vault

将证书和私钥安全地存储在 Azure Key Vault 中，以实现集中式证书管理和访问控制。

**所需配置：**
- **Azure Key Vault Name**：Azure Key Vault 实例名称
- **Azure Key Vault Certificate Name**：存储在 vault 中的证书名称

**身份验证：**
通过 Azure CLI 或环境变量进行身份验证：
- `AZURE_TENANT_ID`: Azure Active Directory tenant ID
- `AZURE_CLIENT_ID`: Service principal client ID
- `AZURE_CLIENT_SECRET`: Service principal client secret

**所需 Azure 角色：**
- "Key Vault Crypto User" - 用于签名操作
- "Key Vault Certificate User" - 用于证书访问

**文档：**
- [Azure Key Vault Overview](https://learn.microsoft.com/en-us/azure/key-vault/general/overview)

:::note
由 [JSign](https://github.com/ebourg/jsign) 提供支持，并需要 Java Runtime。
:::

#### AWS KMS

使用 AWS Key Management Service 安全存储私钥，证书则单独管理。

**所需配置：**
- **AWS Region Code**：密钥所在的 AWS 区域（例如 `us-east-1`、`eu-west-1`、`ap-southeast-2`）
- **AWS Signing Certificate File**：证书文件路径（AWS KMS 仅存储私钥）
- **AWS Signing Key ID or Alias**：AWS KMS 密钥标识符或别名

**身份验证：**
AWS 凭据可来自以下任一来源：
- 环境变量：`AWS_ACCESS_KEY_ID`、`AWS_SECRET_ACCESS_KEY`、`AWS_SESSION_TOKEN`
- ECS 容器凭据
- EC2 IMDSv2 服务

**所需 IAM 权限：**
- `kms:ListKeys` - 用于发现密钥
- `kms:DescribeKey` - 用于读取密钥元数据
- `kms:Sign` - 用于执行签名操作

**文档：**
- [AWS KMS Overview](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)

:::note
由 [JSign](https://github.com/ebourg/jsign) 提供支持，并需要 Java Runtime。
:::

#### DigiCert

使用存储在 DigiCert ONE Secure Software Manager（原 DigiCert KeyLocker）中的证书和密钥，无需安装 DigiCert 客户端工具。

**所需配置：**
- **DigiCert API Key**：用于 DigiCert ONE 身份验证的 API 密钥（从 DigiCert ONE 门户获取）
- **DigiCert Keystore**：包含客户端身份验证证书的 PKCS#12 密钥库文件
- **DigiCert Storepass**：密钥库文件密码
- **DigiCert Certificate Name or ID**：DigiCert ONE 中证书的标识符
- **DigiCert Host**（可选）：自定义主机地址（美国默认值为 `https://clientauth.one.digicert.com`）

**文档：**
- [DigiCert Software Trust Manager](https://docs.digicert.com/en/software-trust-manager.html)

:::note
由 [JSign](https://github.com/ebourg/jsign) 提供支持，并需要 Java Runtime。
:::

#### Google Cloud KMS

使用 Google Cloud Key Management Service 安全存储私钥。由于 Google Cloud KMS 仅存储私钥，因此证书需要单独提供。

**所需配置：**
- **Google Access Token**：用于身份验证的 OAuth 2.0 访问令牌
- **Google Signing Keyring**：密钥环完整路径，格式为：`projects/[PROJECT]/locations/[LOCATION]/keyRings/[KEYRING]/cryptoKeys/[KEY]`
- **Google Signing Certificate File**：证书文件路径
- **Google Signing Certificate Version**（可选）：指定密钥版本（省略时使用最新版本）

**所需 IAM 权限：**
- `cloudkms.cryptoKeyVersions.useToSign` - 用于签名操作
- `cloudkms.cryptoKeyVersions.list` - 未指定版本时需要该权限
- `cloudkms.cryptoKeys.list` - 用于发现密钥

**文档：**
- [Google Cloud KMS Overview](https://cloud.google.com/kms/docs)

:::note
由 [JSign](https://github.com/ebourg/jsign) 提供支持，并需要 Java Runtime。
:::

#### SSL.com eSigner

SSL.com 提供的云签名服务，使用由硬件安全模块（HSM）支持的证书，并提供可选的沙盒环境用于测试。

**所需配置：**
- **eSigner User Name**：SSL.com 账户用户名
- **eSigner Password**：SSL.com 账户密码
- **eSigner Key Password**：用于双因素认证的 Base64 编码 TOTP（基于时间的一次性密码）密钥
- **eSigner Credential ID**：证书对应的凭据标识符（可在 SSL.com 控制台中找到）
- **eSigner Sandbox**（可选）：启用沙盒环境（`https://cs-try.ssl.com`）以便在正式使用前进行测试

**文档：**
- [SSL.com eSigner](https://www.ssl.com/esigner/)

:::note
由 [JSign](https://github.com/ebourg/jsign) 提供支持，并需要 Java Runtime。
:::

## 另请参阅

- [Parcel 安装](/tools/parcel/setup)
- [Parcel 命令行参考](/tools/parcel/command-line-reference)
