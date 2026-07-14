---
id: webassembly
title: WebAssembly 问题
sidebar_label: WebAssembly
description: 排查通过 WebAssembly 在浏览器中运行 Avalonia 应用时的常见问题，包括原生库错误、构建失败和性能方面的考虑。
doc-type: troubleshooting
---

## `System.DllNotFoundException: libSkiaSharp`

此错误通常意味着未安装 `wasm-tools` 工作负载。请安装它并重新构建：

```bash
dotnet workload install wasm-tools
```

如果在安装工作负载后错误仍然存在，请在项目文件中添加 `WasmBuildNative` 属性：

```xml
<PropertyGroup>
    <WasmBuildNative>true</WasmBuildNative>
</PropertyGroup>
```

然后执行一次清理后重建：

```bash
dotnet clean
dotnet build
```

:::tip
如果你使用的是 CI/CD 流水线，请确保在构建步骤运行之前，构建环境中已安装 `wasm-tools` 工作负载。
:::

## 使用 `wasm-tools` 版本不匹配导致构建失败

当你的 .NET SDK 版本与已安装的 `wasm-tools` 工作负载版本不同步时，构建可能会因难以理解的错误而失败。要修复此问题，请将两者都更新为兼容版本：

```bash
dotnet workload update
```

如果你在 `global.json` 文件中固定了特定的 .NET SDK 版本，请验证固定版本与你已安装的工作负载匹配。

## 应用已加载但显示空白屏幕

如果你的应用已编译成功，浏览器也无错误地加载了页面，但屏幕上没有任何渲染内容，请检查以下内容：

1. **浏览器控制台错误。** 打开浏览器开发者工具（F12），查看是否有 JavaScript 异常或网络请求失败。
2. **缺少静态资源。** 确保发布输出中包含所有必需的静态 Web 资源（字体、图片、样式表）。检查 `wwwroot` 文件夹内容。
3. **入口点不正确。** 确认你的 `index.html` 引用了构建生成的正确 `.js` 启动文件。
4. **内容安全策略（CSP）限制。** 如果你通过反向代理或 CDN 托管，请确认你的 CSP 标头允许 .NET WebAssembly 运行时所需的 `wasm-eval` 或 `unsafe-eval`。

## 调试 WebAssembly 应用

基于浏览器的 Avalonia WebAssembly 应用调试需要使用基于 Chromium 的浏览器（Chrome 或 Edge）。要启用调试：

1. 使用启用调试代理的方式启动应用：

    ```bash
    dotnet run --configuration Debug
    ```

2. 在 Chrome 或 Edge 中打开控制台输出中显示的 URL。
3. 按 **Shift+Alt+D** 打开调试面板。

:::note
断点支持、变量检查和单步调试都可用，但可能比原生平台更慢。支持热重载，但某些类型的更改可能需要完整刷新页面。
:::

## 调用外部 API 时出现 CORS 错误

由于你的应用运行在浏览器中，所有 HTTP 请求都受跨域资源共享（CORS）策略限制。如果你的 API 调用因 CORS 错误而失败：

- 确保目标 API 服务器返回适当的 `Access-Control-Allow-Origin` 标头。
- 如果你可以控制该 API，请为你的应用来源添加 CORS 中间件或标头。
- 如果无法修改目标 API 的 CORS 配置，可以考虑使用后端代理转发请求。

## 下载体积过大

WebAssembly 应用的初始下载体积可能会比较大，因为 .NET 运行时及所有引用的程序集都会传输到浏览器。要减小负载：

- 在项目文件中启用裁剪：

    ```xml
    <PropertyGroup>
        <PublishTrimmed>true</PublishTrimmed>
    </PropertyGroup>
    ```

- 在你的 Web 服务器上为 `.wasm` 和 `.dll` 文件启用 Brotli 或 Gzip 压缩。
- 移除未使用的 NuGet 包引用，以尽量减少程序集数量。
- 使用 AOT 编译来提升启动性能，但代价是二进制文件更大（但加载更快）：

    ```xml
    <PropertyGroup>
        <RunAOTCompilation>true</RunAOTCompilation>
    </PropertyGroup>
    ```

:::caution
裁剪可能会移除应用通过反射使用的代码。如果在启用裁剪后运行时看到 `MissingMethodException` 或 `TypeLoadException` 错误，可能需要添加 trim-root 程序集或 `DynamicDependency` 属性来保留所需类型。
:::

## 一般限制

WebAssembly 是一种沙箱化环境，存在一些固有限制：

- 你的应用运行在浏览器沙箱中。文件系统访问、网络访问以及其他操作系统级 API 都会受到浏览器允许范围的限制。
- 浏览器中的 .NET 多线程从 .NET 8 开始才可用，并且需要特定的浏览器支持。并非所有浏览器都支持线程所需的 `SharedArrayBuffer`。
- 与原生平台相比，GUI 性能可能更低，尤其是在较旧的浏览器或低性能设备上。
- 大多数浏览器中，剪贴板访问仅限于用户触发的事件。
- 不支持原生文件对话框。请使用浏览器的文件输入元素或 JavaScript 互操作方式来处理文件选择。

## 另见

- [Avalonia WebAssembly 概览](/docs/platform-specific-guides/webassembly)
- [应用性能问题](/docs/app-development/performance)