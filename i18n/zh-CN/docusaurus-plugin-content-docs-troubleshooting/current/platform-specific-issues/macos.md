---
id: macos
title: macOS 问题
sidebar_label: macOS
---

## 应用菜单

#### 应用菜单显示 _About Avalonia_ 菜单项

这意味着您的应用程序很可能没有指定菜单。在启动时，Avalonia 会为应用程序创建默认菜单项，并在未配置菜单时自动添加 _About Avalonia_ 项。可以通过在您的 `App.xaml` 中添加一个菜单来解决：

```xml
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="using:RoadCaptain.App.RouteBuilder"
             x:Class="RoadCaptain.App.RouteBuilder.App">
	<NativeMenu.Menu>
		<NativeMenu>
			<NativeMenuItem Header="About MyApp" Click="AboutMenuItem_OnClick" />
		</NativeMenu>
	</NativeMenu.Menu>
</Application>
```

其余 macOS 默认菜单项仍将由 Avalonia 生成。

#### 菜单栏中的应用程序名称不匹配

当您从 bundle 运行应用时，菜单栏中显示的应用名称取自 bundle 中的 `Info.plist`，而不是 `App.xaml` 中的 `Name` 属性。

如果名称不匹配，请确认 `CFBundleName`、`CFBundleDisplayName` 和 `Name` 属性的值相同。

请注意，`CFBundleName` 最多只能有 15 个字符，如果您的应用程序名称更长，**必须**设置 `CFBundleDisplayName`。

有关 macOS 从何处读取应用名称的完整细节，请参阅[应用程序名称和标识](/docs/platform-specific-guides/macos#application-name-and-identity)。

## 打包

1. 查看 Parcel 的构建日志以获取错误信息
2. 检查 [Apple Bundle Programming Guide](https://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFBundles/Introduction/Introduction.html) 以了解 bundle 要求

## 代码签名

### 常见问题

#### 创建证书时没有可用的 "Developer ID Application"

此选项需要加入 Apple Developer Account 团队成员资格。请联系您团队的账户持有人以获取访问权限。

#### 应用签名成功，但无法在其他机器上执行

请确认您使用的是 "Developer ID Application" 证书。 "Apple Development" 证书仅适用于开发构建。

### 其他代码签名问题

对于此处未涵盖的问题：

1. 查看 Parcel 的签名日志以获取错误信息
2. 在 Apple Developer 门户中验证证书状态

## 公证

### 常见问题

#### 公证耗时过长

公证通常需要几分钟，但在高峰期可能长达一小时。在最坏情况下，这可能会根据应用程序大小持续数小时。

#### "Invalid credentials" 错误

- 验证您的 Apple ID 和应用专用密码是否正确
- 确保您的 Team ID 准确无误
- 检查您的 Apple Developer 账户状态

#### "License agreement must be accepted" 错误

- 在网页浏览器中登录 [Apple Developer Account](https://developer.apple.com/account/)
- 检查是否有待处理的协议或通知
- 接受任何新的许可协议或服务条款
- 接受后等待几分钟再重试
- 常见于 Apple 开发者计划更新或政策变更之后

#### 上传过程中公证失败

- 检查您的互联网连接

### 其他公证问题

对于此处未涵盖的问题：

1. 查看 Apple 的 [在分发前对 macOS 软件进行公证](https://developer.apple.com/documentation/security/notarizing_macos_software_before_distribution) 指南
2. 查看 Parcel 输出中的公证日志