---
id: docker
title: Docker 容器
description: 如何在 Docker 容器中运行 Avalonia 应用，包括所需的 Linux 依赖和虚拟显示配置。
doc-type: how-to
---

要在 Docker 容器中运行 Avalonia 应用，需要安装 Avalonia 依赖的原生 Linux 库，并为渲染后端提供显示服务器（或虚拟帧缓冲）。

## 所需软件包

Linux 上的 Avalonia 会直接面向 X11，因此容器镜像中必须包含若干默认 .NET 运行时镜像中没有的原生库。

请在 Dockerfile 中安装以下软件包（适用于基于 Debian/Ubuntu 的镜像）：

```bash
apt-get update && apt-get install -y \
    libx11-6 \
    libice6 \
    libsm6 \
    libfontconfig1 \
    xvfb
```

| 软件包 | 用途 |
|---|---|
| `libx11-6` | X11 客户端库。Avalonia 通过它连接到 X 显示服务器。 |
| `libice6` | Inter-Client Exchange 协议库，X 会话所需。 |
| `libsm6` | X Session Management 协议库。 |
| `libfontconfig1` | 字体发现与配置库，文本渲染所必需。 |
| `xvfb` | X Virtual Framebuffer。在没有物理显示器时提供虚拟显示。 |

## 使用 Xvfb 设置虚拟显示

Docker 容器没有物理显示设备。Avalonia 渲染需要 X11 显示，因此你必须运行 Xvfb 来创建虚拟帧缓冲。

在 Dockerfile 中设置 `DISPLAY` 环境变量：

```dockerfile
ENV DISPLAY=:99
```

然后在应用启动前启动 Xvfb。最简单的方式是使用一个 entrypoint 脚本：

```bash title="entrypoint.sh"
#!/bin/bash
Xvfb :99 -screen 0 1920x1080x24 &

# 等待 Xvfb 就绪
sleep 1

exec "$@"
```

或者，也可以在初始化 Avalonia UI 之前，从应用代码中启动 Xvfb：

```csharp
var display = Environment.GetEnvironmentVariable("DISPLAY") ?? ":99";
var xvfb = Process.Start("Xvfb", new[] { display, "-screen", "0", "1920x1080x24" });

// 等待显示变得可用
var timeout = TimeSpan.FromSeconds(5);
var sw = Stopwatch.StartNew();
while (sw.Elapsed < timeout)
{
    // 尝试连接显示
    try
    {
        // 如果应用能无错误启动，则说明显示已就绪。
        break;
    }
    catch
    {
        Thread.Sleep(100);
    }
}
```

## 安装字体

精简版 Docker 镜像通常不包含字体。如果没有字体，文本会渲染成空白方块。请至少安装一个字体族：

```dockerfile
RUN apt-get install -y fonts-noto fonts-ubuntu \
    && fc-cache -fv
```

## 完整 Dockerfile 示例

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src
COPY ["MyApp/MyApp.csproj", "MyApp/"]
RUN dotnet restore "MyApp/MyApp.csproj"
COPY . .
WORKDIR /src/MyApp
RUN dotnet publish "MyApp.csproj" -c Release -o /app/publish \
    --runtime linux-x64 --self-contained true

FROM mcr.microsoft.com/dotnet/runtime:9.0 AS final

# 安装 Avalonia 依赖和 Xvfb
RUN apt-get update && apt-get install -y \
    libx11-6 \
    libice6 \
    libsm6 \
    libfontconfig1 \
    xvfb \
    fonts-noto \
    && fc-cache -fv \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV DISPLAY=:99

WORKDIR /app
COPY --from=build /app/publish .
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD ["./MyApp"]
```

在 Dockerfile 同级目录中创建 entrypoint 脚本：

```bash title="entrypoint.sh"
#!/bin/bash
Xvfb :99 -screen 0 1920x1080x24 &
sleep 1
exec "$@"
```

构建并运行：

```bash
docker build -t myapp .
docker run myapp
```

## 改用无头平台

如果你的容器不需要产生可见输出（例如运行自动化测试或生成图像），可以考虑改用[无头测试平台](/docs/testing/setting-up-the-headless-platform)。无头平台会使用基于内存的实现替代窗口系统和渲染后端，因此不需要 X11 或 Xvfb。

```csharp
public static AppBuilder BuildAvaloniaApp() => AppBuilder.Configure<App>()
    .UseSkia()
    .UseHeadless(new AvaloniaHeadlessPlatformOptions
    {
        UseHeadlessDrawing = false // 如果不需要像素输出，可设为 true
    });
```

由于不再需要 X11 软件包，这种方式可以生成更小的容器镜像。

## 故障排查

### `libX11.so.6: cannot open shared object file`

缺少 `libx11-6` 软件包。请将其添加到 Dockerfile 中：

```dockerfile
RUN apt-get update && apt-get install -y libx11-6
```

### 文本空白或缺失

容器中没有安装字体。请安装字体包并重建字体缓存：

```dockerfile
RUN apt-get install -y fonts-noto && fc-cache -fv
```

### `Cannot open display`

Xvfb 未运行，或者 `DISPLAY` 变量与 Xvfb 的显示编号不一致。请确认 Xvfb 在应用启动前已经运行，并且 `DISPLAY` 设置为相同的值（例如 `:99`）。

## 另请参阅

- [部署到桌面 Linux](/docs/deployment/linux)：了解 `.deb` 打包。
- [部署到嵌入式 Linux](/docs/deployment/embedded-linux)：适用于 DRM/KMS 场景。
- [桌面 Linux 平台集成](/docs/platform-specific-guides/linux)：了解 X11 依赖和 WSL 2。
- [无头测试平台](/docs/testing/setting-up-the-headless-platform)：在没有任何显示服务器的情况下运行。
