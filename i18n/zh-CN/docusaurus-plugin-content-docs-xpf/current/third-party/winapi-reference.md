---
id: winapi-reference
title: WinAPI Shim 参考
---

本页列出了通过 XPF 的 shim 层可用的 Win32 API。这些 shim 用于支持非 Windows 平台上的第三方 WPF 控件。它们并不是通用的 Win32 仿真层。

有关设置和配置，请参见 [Win32 API Shims](/xpf/third-party/win32-api-shims)。

:::note
“Shimmed” 表示 XPF 拦截该调用并提供跨平台实现。在某些情况下，其行为可能与原生 Win32 并不完全相同。标记为 (W) 的函数同时提供 ANSI 和 Unicode 变体。
:::

## user32.dll

### 窗口创建与生命周期

| Function | Description |
|---|---|
| `CreateWindowEx` (W) | 使用扩展样式创建一个窗口。返回由 XPF 管理的虚拟 HWND。 |
| `RegisterClass` | 注册一个窗口类。 |
| `RegisterClassExW` | 注册一个窗口类（扩展版）。 |
| `DestroyWindow` | 销毁一个通过 `CreateWindowEx` 创建的窗口。 |

### 窗口属性与状态

| Function | Description |
|---|---|
| `GetWindowRect` | 获取窗口在屏幕坐标中的边界矩形。 |
| `GetClientRect` | 获取客户区矩形。 |
| `GetWindowPlacement` | 获取窗口的显示状态和位置。 |
| `GetWindowInfo` | 获取窗口信息，包括样式和边框。 |
| `IsWindow` | 测试句柄是否为有效窗口。 |
| `IsWindowEnabled` | 测试窗口是否接受用户输入。 |
| `IsWindowVisible` | 测试窗口是否可见。 |
| `GetWindowLong` / `SetWindowLong` | 获取或设置窗口数据中的 32 位值。 |
| `GetWindowLongPtr` (W) / `SetWindowLongPtr` | 获取或设置窗口数据中的指针大小值。用于窗口样式和过程。 |

### 窗口位置与布局

| Function | Description |
|---|---|
| `SetWindowPos` | 设置窗口的大小、位置和 Z 顺序。 |
| `AdjustWindowRectEx` | 计算给定客户区大小所需的窗口大小。 |
| `BeginDeferWindowPos` / `EndDeferWindowPos` | 批量处理多个窗口位置更改以提升性能。 |

### 窗口层级

| Function | Description |
|---|---|
| `GetActiveWindow` | 返回调用线程上的活动窗口。 |
| `GetTopWindow` | 返回最顶层的子窗口。 |
| `GetWindow` | 检索相关窗口（下一个、上一个、所有者、子级）。 |
| `GetDesktopWindow` | 返回桌面窗口的句柄。 |
| `FindWindow` | 按类名或标题查找顶层窗口。 |
| `WindowFromPoint` | 返回给定屏幕坐标处的窗口。 |
| `EnumChildWindows` | 枚举父窗口的子窗口。 |
| `EnumThreadWindows` | 枚举线程拥有的窗口。 |

### 窗口显示

| Function | Description |
|---|---|
| `SetWindowRgn` | 设置窗口的可见区域。 |
| `RedrawWindow` | 重绘窗口或区域。 |
| `InvalidateRect` | 将矩形标记为需要重绘。 |
| `SetWindowDisplayAffinity` | 控制窗口是否可被屏幕捕获工具捕获。存根实现。 |

### 系统菜单

| Function | Description |
|---|---|
| `GetSystemMenu` | 返回窗口系统菜单的句柄（单击窗口图标时显示的菜单）。 |

### 焦点与输入

| Function | Description |
|---|---|
| `GetFocus` | 返回具有键盘焦点的窗口。 |
| `SetForegroundWindow` | 将窗口置于前台并使其获得焦点。 |
| `GetCapture` / `ReleaseCapture` | 获取或释放鼠标捕获。 |
| `GetKeyState` | 返回某个虚拟键的状态（按下、抬起、切换）。 |
| `GetCursorPos` | 返回屏幕坐标中的光标位置。 |
| `GetMessagePos` / `GetMessageTime` | 返回最后一条消息的光标位置和时间。 |

### 消息

| Function | Description |
|---|---|
| `SendMessage` | 向窗口发送消息并等待处理。仅支持有限的消息。 |
| `PostMessage` | 将消息投递到窗口的消息队列。仅支持有限的消息。 |
| `DefWindowProc` (W) | 为窗口过程未处理的消息提供默认处理。 |
| `FormatMessageW` | 格式化系统错误消息字符串。 |

### 钩子

| Function | Description |
|---|---|
| `SetWindowsHookEx` | 安装一个钩子过程。仅支持有限的钩子类型。 |
| `UnhookWindowsHookEx` | 移除一个钩子过程。 |

### 光标

| Function | Description |
|---|---|
| `CreateCaret` | 为窗口创建插入光标（文本光标）。 |
| `ShowCaret` / `HideCaret` | 显示或隐藏插入光标。 |
| `DestroyCaret` | 销毁当前插入光标。 |
| `SetCaretPos` | 设置插入光标位置。 |

### 菜单

| Function | Description |
|---|---|
| `TrackPopupMenuEx` | 在指定位置显示快捷菜单。 |
| `EnableMenuItem` | 启用、禁用或灰显菜单项。 |

### 系统信息

| Function | Description |
|---|---|
| `GetSysColor` | 返回显示元素的当前颜色（按钮表面、窗口背景等）。 |
| `SystemParametersInfo` (W) | 获取或设置系统范围参数（滚动条大小、动画设置等）。 |
| `GetDoubleClickTime` | 返回双击两次点击之间的最大间隔。 |
| `GetSystemMetrics` | 返回系统度量值（屏幕大小、图标大小、滚动条尺寸等）。 |
| `GetCaretBlinkTime` | 返回插入光标闪烁间隔。 |

### 剪贴板

| Function | Description |
|---|---|
| `AddClipboardFormatListener` | 注册一个窗口以接收剪贴板更改通知。 |

## gdi32.dll

### 设备上下文

| Function | Description |
|---|---|
| `GetDC` / `ReleaseDC` | 获取或释放窗口的设备上下文。 |
| `CreateCompatibleDC` / `DeleteDC` | 创建或删除内存设备上下文。 |
| `GetDeviceCaps` | 返回设备能力（DPI、颜色深度等）。 |

### 绘图对象

| Function | Description |
|---|---|
| `CreateRectRgn` | 创建一个矩形区域。 |
| `CreateRoundRectRgn` | 创建一个带圆角的矩形区域。 |
| `CreateRectRgnIndirect` | 从 RECT 结构创建一个矩形区域。 |
| `DeleteObject` | 删除一个 GDI 对象（区域、画刷、画笔等）。 |
| `GetStockObject` | 返回一个预定义库存对象的句柄。 |

### 坐标映射

| Function | Description |
|---|---|
| `GetMapMode` / `SetMapMode` | 获取或设置设备上下文的映射模式。 |
| `SetWindowExtEx` / `SetViewportExtEx` | 设置用于坐标映射的窗口或视口范围。 |
| `OffsetRect` | 按指定偏移移动矩形。 |

## dwmapi.dll

| Function | Description |
|---|---|
| `DwmIsCompositionEnabled` | 返回桌面合成是否已启用。在非 Windows 上始终返回 true。 |
| `DwmExtendFrameIntoClientArea` | 将窗口框架扩展到客户区。 |
| `DwmGetWindowAttribute` | 获取 DWM 窗口属性。仅支持有限的属性。 |
| `DwmSetWindowAttribute` | 设置 DWM 窗口属性。仅支持有限的属性。 |

## shcore.dll / 监视器 API

| Function | Description |
|---|---|
| `MonitorFromPoint` | 返回包含某个点的监视器。 |
| `MonitorFromRect` | 返回与矩形相交面积最大的监视器。 |
| `MonitorFromWindow` | 返回包含窗口最大部分的监视器。 |
| `GetMonitorInfo` | 返回监视器的显示区域和工作区。 |
| `EnumDisplayMonitors` | 枚举显示监视器。 |
| `GetDpiForMonitor` | 返回监视器的 DPI。 |
| `GetDpiForWindow` | 返回窗口的 DPI。 |
| `GetProcessDpiAwareness` | 返回进程的 DPI 感知设置。 |

## imm32.dll（输入法编辑器）

| Function | Description |
|---|---|
| `ImmCreateContext` / `ImmDestroyContext` | 创建或销毁 IME 输入上下文。 |
| `ImmGetContext` / `ImmReleaseContext` | 获取或释放窗口的 IME 上下文。 |
| `ImmAssociateContext` | 将 IME 上下文与窗口关联。 |
| `ImmSetOpenStatus` / `ImmGetOpenStatus` | 打开或关闭 IME，或查询当前状态。 |
| `ImmNotifyIME` | 向 IME 发送通知。 |
| `ImmGetProperty` | 返回 IME 属性。 |
| `ImmGetCompositionString` (W) | 返回组合字符串（正在输入的文本）。 |
| `ImmSetCompositionFont` (W) | 设置用于显示组合字符串的字体。 |
| `ImmConfigureIMEW` | 打开 IME 配置对话框。 |
| `ImmSetCompositionWindow` | 设置组合窗口的位置。 |
| `ImmSetCandidateWindow` | 设置候选列表窗口的位置。 |
| `ImmGetDefaultIMEWnd` | 返回默认 IME 窗口句柄。 |

## kernel32.dll

| Function | Description |
|---|---|
| `GetCurrentThreadId` | 返回调用线程的 ID。 |
| `GetModuleFileName` | 返回已加载模块的完整路径。 |
| `GetModuleHandle` (W) | 按名称返回已加载模块的句柄。 |
| `LoadLibrary` (W) | 加载 DLL。由 shim 拦截以重定向 Win32 DLL 加载。 |
| `LoadString` (W) | 从可执行文件中加载字符串资源。 |
| `CloseHandle` | 关闭对象句柄。 |
| `RtlGetVersion` | 返回操作系统版本。在非 Windows 上返回模拟的 Windows 版本信息。 |

### 文件与内存映射

| Function | Description |
|---|---|
| `CreateFileMapping` | 创建或打开文件映射对象。 |
| `MapViewOfFile` / `UnmapViewOfFile` | 将文件映射视图映射到或取消映射出进程地址空间。 |
| `FindFirstFile` / `FindNextFile` / `FindClose` | 枚举目录中的文件。 |

### 内存

| Function | Description |
|---|---|
| `RtlMoveMemory` | 复制一块内存。 |

## shell32.dll

| Function | Description |
|---|---|
| `SHGetFileInfo` (W) | 返回有关文件的信息（图标、显示名称、类型）。 |
| `ExtractIconEx` (W) | 从可执行文件或 DLL 中提取图标。 |

## uxtheme.dll

| Function | Description |
|---|---|
| `IsThemeActive` | 返回视觉样式是否处于活动状态。 |
| `SetWindowThemeAttribute` | 为窗口设置主题属性。 |

## msctf.dll

| Function | Description |
|---|---|
| `TF_CreateThreadMgr` | 创建一个文本服务框架线程管理器。 |