---
id: android
title: Android
description: 将 Avalonia 应用构建、签名并发布为适用于 Android 设备的 APK 或 AAB。
doc-type: how-to
---

将 Avalonia 应用发布到 Android 平台时，会生成 Android Package（APK）或 Android App Bundle（AAB）文件。APK 用于将应用直接安装到 Android 设备上，而 AAB 则用于将应用发布到 Google Play。

## 在模拟器上运行

在 Android 项目目录中，使用以下命令进行构建和运行：

```bash
dotnet build
dotnet run
```

这会将应用部署到默认的 Android 模拟器中。如果当前没有模拟器在运行，系统会自动使用 Android SDK 中已配置的第一个可用 AVD（Android Virtual Device）来启动模拟器。

## 在真机上运行

如果要部署到真实 Android 设备上：

1. 确保设备上的 Android 版本与 `AndroidManifest.xml` 中声明的受支持版本或目标版本兼容。
2. 通过 USB 将设备连接到开发机器。
3. 在设备的开发者选项中启用 **USB Debugging**。
4. 如果默认 USB 连接模式是“仅充电”，请切换为 MTP 或其他能让 ADB 检测到设备的模式。

然后执行：

```bash
dotnet run
```

应用将被部署并在已连接的设备上启动。

## 发布

### 创建 keystore 文件

在开发阶段，.NET for Android 会使用一个调试 keystore 为应用签名，这样应用就可以直接部署到模拟器，或者部署到允许运行可调试应用的设备上。但这个 keystore 不能用于正式分发应用。要签署发布版本，必须创建并使用私有 keystore。

:::tip
这一步通常只需要执行一次。同一个 keystore 可用于后续版本更新发布，也可以用于签署其他应用。请务必备份你的 keystore 和密码；如果丢失，你将无法再用相同身份继续签署应用。
:::

使用 JDK 中的 `keytool`，并传入以下参数：

```bash
keytool -genkeypair -v -keystore myapp.keystore -alias myapp -keyalg RSA -keysize 2048 -validity 10000
```

系统会提示你输入并确认密码，然后继续填写姓名和组织信息。这些信息会写入证书中，但不会显示在应用里。

### 构建并签名应用

进入 Android 项目文件夹，并带上签名参数运行 `dotnet publish`：

```bash
dotnet publish -f net9.0-android -c Release \
  -p:AndroidKeyStore=true \
  -p:AndroidSigningKeyStore=myapp.keystore \
  -p:AndroidSigningKeyAlias=myapp \
  -p:AndroidSigningKeyPass=mypassword \
  -p:AndroidSigningStorePass=mypassword
```

这会完成应用的构建和签名，并在 `bin/Release/net9.0-android/publish` 文件夹中生成 AAB 和 APK 文件。已签名的文件名中会包含 **-signed**。

:::caution
在共享环境中，请避免直接在命令行中传递密码。建议改用 `env:` 或 `file:` 前缀（见下文）。
:::

#### 安全地处理密码

`AndroidSigningKeyPass` 和 `AndroidSigningStorePass` 都支持 `env:` 与 `file:` 前缀，以避免在构建日志中暴露密码。

使用环境变量：
```bash
dotnet publish -f net9.0-android -c Release \
  -p:AndroidKeyStore=true \
  -p:AndroidSigningKeyStore=myapp.keystore \
  -p:AndroidSigningKeyAlias=myapp \
  -p:AndroidSigningKeyPass=env:ANDROID_SIGNING_PASSWORD \
  -p:AndroidSigningStorePass=env:ANDROID_SIGNING_PASSWORD
```

使用文件：
```bash
dotnet publish -f net9.0-android -c Release \
  -p:AndroidKeyStore=true \
  -p:AndroidSigningKeyStore=myapp.keystore \
  -p:AndroidSigningKeyAlias=myapp \
  -p:AndroidSigningKeyPass=file:/path/to/password.txt \
  -p:AndroidSigningStorePass=file:/path/to/password.txt
```

:::note
当 `AndroidPackageFormat` 设置为 `aab` 时，不支持 `env:` 前缀。
:::

### 构建属性参考

以下属性既可以通过命令行中的 `-p:` 传入，也可以直接写在项目文件中的 `<PropertyGroup>` 内：

| 属性 | 说明 |
|---|---|
| `AndroidKeyStore` | 设为 `true` 表示对应用进行签名。默认值：`false`。 |
| `AndroidPackageFormats` | 以分号分隔，可设置为 `aab`、`apk` 或 `aab;apk`。发布版默认值为 `aab;apk`。 |
| `AndroidSigningKeyAlias` | keystore 中所使用密钥的别名。 |
| `AndroidSigningKeyPass` | 密钥密码。支持 `env:` 和 `file:` 前缀。 |
| `AndroidSigningKeyStore` | keystore 文件名。 |
| `AndroidSigningStorePass` | keystore 密码。支持 `env:` 和 `file:` 前缀。 |
| `ApplicationTitle` | 用户可见的应用名称。 |
| `ApplicationId` | 唯一标识符，例如 `com.companyname.myapp`。 |
| `ApplicationVersion` | 构建版本号。 |
| `ApplicationDisplayVersion` | 用于显示的版本字符串。 |
| `PublishTrimmed` | 是否裁剪未使用代码。发布构建默认值为 `true`。 |

#### 在项目文件中定义属性

你也可以不把所有参数都写在命令行里，而是直接在 `.csproj` 中进行设置：

```xml
<PropertyGroup Condition="$(TargetFramework.Contains('-android')) and '$(Configuration)' == 'Release'">
    <AndroidKeyStore>true</AndroidKeyStore>
    <AndroidSigningKeyStore>myapp.keystore</AndroidSigningKeyStore>
    <AndroidSigningKeyAlias>myapp</AndroidSigningKeyAlias>
    <AndroidSigningKeyPass>env:ANDROID_SIGNING_PASSWORD</AndroidSigningKeyPass>
    <AndroidSigningStorePass>env:ANDROID_SIGNING_PASSWORD</AndroidSigningStorePass>
</PropertyGroup>
```

之后只需执行：
```bash
dotnet publish -f net9.0-android -c Release
```

### 分发应用

- **Google Play**：通过 [Google Play Console](https://play.google.com/console) 提交你的 AAB 文件。详情请参阅 [Upload your app to the Play Console](https://developer.android.com/studio/publish/upload-bundle)。
- **直接下载**：将 APK 放到网站或文件共享中供用户下载。用户需要在设备上启用“允许安装未知来源应用”。详情请参阅 [User opt-in for unknown apps](https://developer.android.com/studio/publish#publishing-unknown)。

## 另请参阅

- [Android platform setup](/docs/platform-specific-guides/android)
