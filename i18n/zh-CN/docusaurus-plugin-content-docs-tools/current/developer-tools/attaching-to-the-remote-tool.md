---
id: attaching-to-the-remote-tool
title: 将 DevTools 附加到远程工具
sidebar_label: 附加到远程工具
doc-type: how-to
tags:
  - avalonia plus
  - avalonia pro
  - avalonia enterprise
---

`Developer Tools` 可以连接到运行在不同机器上的应用程序。本指南涵盖两种场景：
1. 局域网访问（例如虚拟机或连接到同一 Wi-Fi 的设备）
2. 通过 VPN 进行互联网访问（出于安全考虑，推荐）

`Developer Tools` 会在端口 `29414` 上运行一个 HTTP 服务器。关键在于确保连接方机器可以访问该服务器。

## 局域网访问

1. 获取运行 `Developer Tools` 的机器的局域网 IP 地址：
- Windows：打开命令提示符并运行 `ipconfig`
- macOS/Linux：打开终端并运行 `ip addr` 或 `ifconfig`
  查找以以下内容开头的 IPv4 地址：
- `192.168.`（大多数家庭网络）

2. 配置你的应用使用该 IP：

```csharp
this.AttachDeveloperTools(o =>
{
    o.Protocol = DeveloperToolsProtocol.CreateHttp(IPAddress.Parse("YOUR_LOCAL_NETWORK_HOST_IP"));
});
```
3. 启动 `Developer Tools`（通过 avdt 命令）
4. 在设置中确认已启用 `Allow Any IP`（如果之前未启用，则需要重启应用）
5. 在第二台机器上启动你的应用，并按下 <kbd>F12</kbd> 进行连接。

:::note

请确保两台机器上的防火墙都允许端口 29414 通信。

:::

## 通过 VPN 进行互联网访问

虽然也可以不使用 VPN，而是在公网环境中设置端口转发，但并不推荐这样做。长期开放端口通常被认为是不良实践。

本教程改用 VPN 在机器之间建立受限访问，这里将使用 `Tailscale`，它是最简单的选择之一。
同时请确保安装了 developer tools 的机器能够使用 `Tailscale CLI`。可参阅 [Tailscale CLI](https://tailscale.com/kb/1080/cli)。

1. 请按照 `Tailscale` 的 [Quick Start guide](https://tailscale.com/kb/1017/install) 在两台机器上完成安装，并配置彼此访问。本教程特别关注 `MagicDNS` 功能。
2. 当 `Tailscale` 在两台设备上都已安装并连接后，需要在安装了 `Developer Tools` 的机器上对外提供 `29414` 端口。运行
   `tailscale serve 29414`
   或者在 macOS 上运行
   `/Applications/Tailscale.app/Contents/MacOS/Tailscale serve 29414`
   CLI 会输出类似如下内容：

```bash
Available within your tailnet:

https://machinename.tail.ts.net/
|-- proxy http://127.0.0.1:29414

Press Ctrl+C to exit.
```

从输出中复制 `https://machinename.tail.ts.net/` 这个 URL。下一步会用到它。

3. 在 `AttachDeveloperTools` 选项中使用上一步得到的 URL：

```csharp
this.AttachDeveloperTools(o =>
{
    o.Protocol = DeveloperToolsProtocol.CreateHttp(new Uri("https://machinename.tail.ts.net"));
});
```

4. 启动 `Developer Tools`（通过 avdt 命令）
5. 在第二台机器上启动你的应用，并按下 <kbd>F12</kbd> 进行连接。

![Connected via VPN](/img/tools/dev-tools/remote-connect-via-vpn.png)

## 更改默认端口

在某些情况下，默认端口 `29414` 可能不可用。

如果要更改端口，需要同时调整 `Developer Tools` 和 `AttachDeveloperTools` 两侧的设置。

在 `Developer Tools` 中，可在设置页修改 `HTTP port` 参数，然后重启应用。更多细节请参阅 [Settings](/tools/developer-tools/settings)。

在 `AttachDeveloperTools` 侧，可以在 `DeveloperToolsProtocol.CreateHttp` 方法中将新端口作为可选参数传入。

## 另请参阅

- [附加应用程序](/tools/developer-tools/attaching-applications)
- [Developer tools 设置](/tools/developer-tools/settings)
