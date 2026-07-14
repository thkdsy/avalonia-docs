---
id: webassembly
title: WebAssembly
---

Avalonia 应用可以通过 WebAssembly（WASM）在浏览器中运行。本页介绍如何配置用于浏览器部署的项目，以及如何使用 JavaScript 互操作。

## 为 WebAssembly 配置 Avalonia 项目

1. 安装 `wasm-tools` workload，它提供将 .NET 编译为 WebAssembly 所需的构建工具链。

```bash
dotnet workload install wasm-tools
```

:::note
如果你是在 .NET 9 SDK 上运行 `net8.0-browser` 应用，则应改为安装 `wasm-tools-net8` workload。
如果你使用的是较旧版本的 .NET SDK，系统也可能提示你安装 `wasm-experimental` 等其他 workload。
:::

2. 安装或更新 dotnet 模板到最新版本。

```bash
dotnet new install avalonia.templates
```

3. 为项目创建一个新目录。

```bash
mkdir BrowserTest
cd BrowserTest
```

4. 生成一个支持在浏览器中运行的新项目。你可以执行 `dotnet new list` 查看所有可用的 Avalonia 模板。

```bash
dotnet new avalonia.xplat
```

5. 在控制台输出中，你会看到用于打开应用的 HTTP 和 HTTPS 链接。
运行应用：

```bash
cd BrowserTest.Browser
dotnet run

# Output:
# App url: http://127.0.0.1:53576/
# App url: https://127.0.0.1:53577/
# Debug at url: http://127.0.0.1:53576/_framework/debug
# Debug at url: https://127.0.0.1:53577/_framework/debug
```

## 部署

关于发布和部署 WebAssembly 应用的更多信息，请参阅 [部署 WebAssembly](/docs/deployment/webassembly)。

## JavaScript 互操作

Avalonia 浏览器应用可以从 C# 调用 JavaScript，也可以通过 .NET 标准的 `[JSImport]`/`[JSExport]` 互操作 API 将 C# 方法暴露给 JavaScript。该 API 属于 `System.Runtime.InteropServices.JavaScript` 命名空间，并适用于任何 .NET WebAssembly 应用。

### 配置

在 Browser 项目的项目文件中加入 `AllowUnsafeBlocks`。生成互操作绑定的 .NET 源生成器需要该设置：

```xml
<PropertyGroup>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
</PropertyGroup>
```

### 从 C# 调用 JavaScript

在 `partial` 方法上使用 `[JSImport]` 特性来导入 JavaScript 函数。第一个参数是 JS 函数名，第二个参数是加载时使用的模块名。

创建一个 JavaScript 模块（例如 `wwwroot/js/interop.js`）：

```javascript
export function showAlert(message) {
    globalThis.alert(message);
}

export function getCurrentUrl() {
    return globalThis.window.location.href;
}
```

定义映射到这些 JS 函数的 C# 方法：

```csharp
using System.Runtime.InteropServices.JavaScript;
using System.Runtime.Versioning;

[SupportedOSPlatform("browser")]
public partial class JsInterop
{
    [JSImport("showAlert", "MyInterop")]
    public static partial void ShowAlert(string message);

    [JSImport("getCurrentUrl", "MyInterop")]
    public static partial string GetCurrentUrl();
}
```

在启动时（通常在 `Program.cs` 中）使用 `JSHost.ImportAsync` 加载模块，然后就可以在应用任意位置调用这些方法：

```csharp
using System.Runtime.InteropServices.JavaScript;

await JSHost.ImportAsync("MyInterop", "../js/interop.js");

// 现在你可以调用：
JsInterop.ShowAlert("Hello from Avalonia!");
string url = JsInterop.GetCurrentUrl();
```

传给 `JSHost.ImportAsync` 的模块名必须与 `[JSImport]` 特性中的第二个参数一致。

### 从 JavaScript 调用 C#

使用 `[JSExport]` 特性把 C# 方法暴露给 JavaScript：

```csharp
[SupportedOSPlatform("browser")]
public partial class JsInterop
{
    [JSExport]
    public static string GetAppVersion() => "1.0.0";
}
```

在 JavaScript 中，可通过 .NET 运行时访问已导出的方法：

```javascript
export async function callDotNet() {
    const { getAssemblyExports } = await globalThis.getDotnetRuntime(0);
    const exports = await getAssemblyExports("MyApp.dll");
    const version = exports.MyNamespace.JsInterop.GetAppVersion();
    console.log(version);
}
```

### 访问全局函数

如果要从全局作用域导入函数（而不是模块），请在函数名前加上 `globalThis`，并省略模块名：

```csharp
[JSImport("globalThis.console.log")]
public static partial void ConsoleLog(string message);
```

### 类型封送

.NET 类型会自动封送为对应的 JavaScript 类型。如果需要显式控制封送行为，请使用 `[JSMarshalAs]` 特性：

```csharp
[JSImport("processData", "MyInterop")]
public static partial void ProcessData(
    [JSMarshalAs<JSType.Number>] long value);
```

你可以把 `Action`/`Func` 回调作为参数传递（会被封送为可调用的 JS 函数），同时 JS 对象和托管对象引用也都可以以代理对象的形式跨边界传递。

## 另请参阅

- [部署 WebAssembly](/docs/deployment/webassembly)
- [WebAssembly 故障排查](/troubleshooting/platform-specific-issues/webassembly)
