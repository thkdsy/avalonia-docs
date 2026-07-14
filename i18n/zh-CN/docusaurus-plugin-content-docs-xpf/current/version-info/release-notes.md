---
id: release-notes
title: 发布说明
---

## XPF 1.6.3 (2026-04-22)

* Avalonia 版本从 11.3.12 更新到 11.3.14
* 为 flow documents 添加了节支持
* 修复了某些情况下 PNG 解码器使用错误像素格式的问题
* 修复了在 macOS 上复制位图时红色和蓝色通道互换的问题
* 更新了以下依赖项：
  * System.Security.Cryptography.Xml 8.0.0 到 8.0.3（修复安全漏洞）
  * SixLabors.ImageSharp 从 3.1.9 到 3.1.12（修复安全漏洞）

## XPF 1.6.2 (2026-03-20)

* 修复了在 macOS 上当剪贴板为空时右键单击 TextBox 导致崩溃的问题
* 修复了从按钮关闭 popup 时需要额外点击一次的问题（macOS，Linux）
* 修复了单击 ComboBox 内部滚动条的问题
* 修复了单击已打开子菜单的菜单项的问题
* 修复了在 Menu 中打开 ComboBox 的问题
* 修复了包含大量项的菜单占用过多空间并遮挡父元素的问题

## XPF 1.6.1 (2026-02-19)

* Avalonia 版本从 11.3.11 更新到 11.3.12
* 支持从剪贴板粘贴 macOS 位图格式
* 修复了 AvaloniaHost 会获取所有输入的问题
* 修复了在 TabControl 中切换选项卡会获取所有输入的问题
* 修复了 WinForms 对话框在没有 FilterIndex 时崩溃的问题
* [Accessibility] 支持 ItemType 和 ItemStatus 属性
* [Accessibility] 修复了添加子项时缺少通知的问题
* 实现了 BringWindowToTop、DestroyWindow、IsChild、SetActiveWindow、SetFocus API shim

已知问题：

* 从按钮关闭 popup 可能需要额外点击一次（macOS，Linux）

## XPF 1.6.0 (2026-01-19)

* Avalonia 版本从 11.3.1 更新到 11.3.11
* 以下区域进行了重大重构：
  * 鼠标捕获和捕获丢失
  * 剪贴板（位图支持、自定义格式、往返）
* 新增对 macOS 上 Ctrl+Click 的支持，以及将 F10 作为系统键
* 新增对 System.Windows.Documents.Typography 属性的支持
* 新增对文件对话框中筛选索引的支持
* popup 修复（定位、对话框显示时禁用）
* MessageBox 功能与 Windows 保持一致（标题、最大尺寸）
* 文本修复和改进（行距、内联块、字体加载和匹配）
* 自动化修复和改进
* DPI 和缩放修复和改进-
* 使所有日志接收器均可用

已知问题：
* 在 TabControl 中切换选项卡不会使 UIA 树失效
* 获得焦点后 AvaloniaHost 会获取所有输入
* 从按钮关闭 popup 可能需要额外点击一次

## XPF 1.5.3 (2025-06-23)

* 修复 MediaContext 使用 Stopwatch 而不是系统时钟
* 调整 WinForms MessageBoxTheme
* 强制使用 WPF LineSpacing

## XPF 1.5.2 (2025-06-09)

* 将 MessageBox 宽度限制为 400px，以更接近 WIN32 行为
* 将 MessageBox 限制为当前屏幕高度的 80%
* 将 Avalonia 更新到稳定版 11.3.1

## XPF 1.5.1 (2025-05-21)

* 将 VB MsgBox 默认标题改为与 Windows 一致

## XPF 1.5.0 (2025-05-07)

* 将 TextAlignment 应用于溢出的 AVTextLine
* BmpBitmapDecoderHandle - 支持 1 位 BMP
* 修复某些情况下未触发 Window.Closing 事件的问题
* 修复将 ImageBrush 作为透明度遮罩使用时损坏的问题
* 修复径向渐变和虚线数组渲染
* 修复隐藏/显示对话框的问题。
* 当对话框被隐藏时，将对话任务标记为完成
* 通过裁剪边界限制子树内容边界
* 修复未指定文件名时文件对话框崩溃的问题
* 确保 draw rounded rectangle 在有效半径下执行
* 备用菜单捕获修复
* 如果没有主显示器，则使用第一个显示器
* 修复 AvalonEdit 完成窗口崩溃
* 添加对 FontFamily.FamilyTypefaces 的支持
* 实现 SetCursorPos API shim
* 提高位图性能
* 通过 BeginInvokeOnRender 调用 MediaContext 更新处理程序
* 移除 ribbon 控件中的本机调用
* 修复 x86 上的 win32 shim
* 为 APISHIM 调用添加日志记录
* 文件 ok 处理 
* 将 System.IO.Packaging 更新到 6.0.2
* 确保 paragraphWidth 向上取整
* 添加对 VB MsgBox 函数的支持
* 添加对 VB `InputBox` 的支持。
* 修复 RichTextBox 的 Border 位置
* 修复 XpfContain 从树中分离时的崩溃
* 处理 8bpp 位图的情况
* 重构字体加载
* 处理高度为零的 RectangleNode
* 修复字体度量舍入误差
* 修复帧解码完成
* 修复 PngBitmapDecoder alpha 通道
* 不要让 Font 和 FontFamily 共享同一个 FontMetrics
* 加载字体集合时处理重复字体
* 修复自定义字体字形模拟
* 确保本地化字体字符串映射到可用区域设置
* 简化解码后位图数据的目标像素格式
* 允许在没有 placement target 的情况下显示 popup。
* 添加对索引位图源的支持
* 实现 OpenFolderDialog
* 不要在 `Popup` 中调用 win32 `GetCursorPos`。

## XPF 1.4.0 (2025-01-08) 

* 移除 System.Configuration.ConfigurationManager 的使用
* 修复 ModifierKeys.MacControl 值
* 将 WebRequest/WebResponse 基类构造函数调用替换为 RuntimeHelpers.GetUninitializedObject + Frame 控件修复
* 修复按键重映射异常
* 在失去焦点时重置修饰键
* 简单的 3D 支持
* 修复 Paragraph TextAlignment
* 修复 MuceViewport3DVisual 中的编译错误
* 添加 GetAvaloniaTopLevelForWindow
* 用于浏览器等单视图平台的窗口框架
* 可调整大小的单视图窗口，修复装饰
* 实现在线许可票据
* 将剩余平台包含在许可证检查中
* 修复 avalon dock 调整大小问题
* 修复右下角光标
* 替换 binary formatter
* Stub ShowSystemMenu
* 修复 DragOver 暴露已释放的 IDataObject
* 当路径简化失败时不崩溃
* 添加 ScreenToClient win32 API shim。
* 修复嵌入 Avalonia 时的 XPF popup。
* 在 macOS 上默认使用 OpenGL
* 实现 JpegMetadata 并允许读取方向信息
* 修复 ElementProxy 中的 NRE
* 添加运行时标志以启用 DataObject 自定义格式的互操作
* 添加对读取和写入 exif MakerNote 的支持
* 修复“control is not in any visual tree”
* 修复 Cursors.None
* 修复 popup 内存泄漏
* Window 失活时释放鼠标捕获。
* 更新 Avalonia.Licensing
* 确保设置 visual root dpi
* 修复 BitmapSource 4 字节对齐
* 将某些元数据查询规范化为小写
* 如果未明确启用 ALC 支持，则使用默认 WPF 行为
* 在线未折叠时调整 TextLine 裁剪

## XPF 1.3.0 (2024-08-12)

* 启用基于 ECDSA 的许可证密钥
* 修复多个 Geometry API
* 缓存 FontCollections 并使其线程感知
* 修复几何命中测试中非描边段的问题
* 将主版本更新为 1.3
* 实现 BlockUIContainer
* 更新 Avalonia 版本
* 修复调度异常处理
* 支持将 XPF 加载到单独的 ALC 中以实现简陋的多线程
* 修复 MUCE 几何失效
* 修复大型图像解码
* 修复内联 TextDecorations 和 BaselineAlignment
* 修复 TextDecorations
* 修复 macOS 上的按键映射
* 修复 GetDpiForMonitor
* 实现 F32MonitorHandle 并使用新的 Screens API 
* 针对不正确的 glyph run 边界的临时修复
* 不要将鼠标事件作为触摸事件处理
* 从外部可用的 Window.Close 方法调用 InternalClose
* 初始 PDF 生成支持
* 添加对矩形几何中圆角半径的支持
* 添加对无描边几何段的支持
* 更新 avalonia：修复 win32 窗口已显示状态
* 更新 avalonia- 修复顶层拥有窗口无法工作的问题
* AvaloniaHostContainer 需要将 transform origin 设置为零，而不是默认的 50%50%
* 如果窗口重新激活，则让焦点保留在 avalonia host 中
* 添加虚拟 AdjustWindowRectEx
* 修复保存对话框中文件名设置错误的问题
* 修复 Window.Icon
* 修复“如果按下另一个鼠标按钮，鼠标事件未正确触发”
* 手动订阅 MuceVisualBrush 的所有子树视觉无效化
* 修复 headless 平台，避免硬编码 Skia
* 小幅 patcher 改进
* 添加标志以禁用 devexpress 的 stackframe 调整
* 如果内容最终会被裁剪掉，则跳过渲染
* 更改我们池化 MuceRenderData 状态的方式
* 修复 FontAwesome.Sharp 字体加载
* 提升 image sharp 版本
* SHGetFileInfo 的 Stub
* 实现 bitmap.copypixels
* 修复 DrawingImage，针对无父级 VisualBrush 的临时修复
* 从所有分支发布包
* 为发布标签添加正则 semver 检查
* 更新 dotnet.yml 以去除版本末尾的反斜杠
* 更新 Common.props
* 移除 UseWinForms + 杂项
* 禁用 ImportWindowsDesktopTargets
* 允许用户通过 msbuild 属性或环境变量启用日志记录
* 添加 XpfSingleProject 属性
* 更新 AvaloniaUI.Xpf.WinApiShim.targets
* 浏览器兼容性改进
* 为 Browser 提供实验性的 WinAPI shim 支持
* 修复浏览器 winapi shim
* 修复浏览器 SDK
* 与 Screen API 相关的各种 WinAPI shim 修复
* 一些 mono 修复
* 支持 SystemInformation.MouseWheelScrollDelta
* 在创建新 win 时重置 popup _positionInfo


## XPF 1.2.0 (2024-05-29)

* 更新 ImageSharp
* 使 DragDrop 处理程序可用于任何 Control，而不只是 TopLevel
* 使用虚拟窗口句柄实现 GetActiveWindow
* 使 HwndWrapper 可用
* 忽略 avalonia window 的 size to content
* 修补 XPF 程序集以在 DllImport 时抛出异常
* 更新 GetSizeFromHwnd
* 将 Avalonia nuget 更新到 11.2 alpha
* 检查 XpfHost 是否实际上已附加到某个对象，以决定是否应延迟创建 popup
* 修复 ExclusivelyOwnedWindow 实际为 null 时的情况
* 移植 wpf popup 定位逻辑
* 如果窗口重新激活，则让焦点保留在 avalonia host 中
* 使 snoop 在更多情况下可用
* 添加对矩形几何中圆角半径的支持
* 在创建新窗口时重置 popup _positionInfo
* 修复 Telerik 的 RadTooltipWindow
* 从 .cur 文件中读取热点
* 如果内容最终会被裁剪掉，则跳过渲染
* 为 WPF 的 Brush.Transform 属性使用绝对变换原点
* 检查窗口在检查焦点时之前是否已被激活
* 修复 VisualBrush 回归问题
* 为 Pen 光标类型添加默认位图光标
* 初始 PDF 生成支持
* 添加 SystemInformation.MouseWheelScrollDelta
* 在初始状态下设置窗口位置时触发位置变更
* 当控件获得焦点时激活窗口
* 修复位图编码问题
* 在托管拖动期间阻止输入
* 实现 BlockUIContainer
* X11 - 跟踪窗口激活是否由于控件焦点而完成
* 当在 linux 上设置位置时，回退为设置 dragpoint
* Stub UnhookWindowsHookEx
* 与 Screen API 相关的各种 WinAPI shim 修复
* 映射更多像素格式
* Actipro 停靠修复
* 不要在 XPF 中从 ComboBox 调用 GetCapture
* 为 WriteableBitmap.AddDirtyRect 发送 MILCMD_BITMAP_INVALIDATE
* 为 PDF 文档正确配置 DPI 和页面大小元数据
* 不允许在最大化窗口上调整大小 x11
* ManagedWindowDragHelper - 跟踪之前的位置，并在处理 WM_MOVING 时更新位置
* MonitorFromWindow：不要对未附加的 visual 抛出异常
* 修复未处理的异常
* 修复一些 Geometry 问题
* 添加对无描边几何段的支持
* 添加 SKColorFilter free 回调
* 允许用户通过 msbuild 属性或环境变量启用日志记录
* 不要在 XPF 中从 MenuBase 调用 GetCapture
* 更新 XpfSkiaExtensions 的 PresentationCore ref
* 为 MessageBoxTheme.axaml 添加背景设置
* 在鼠标被捕获时阻止非客户端输入
* 修复 popup 的屏幕工作区
* 修复文本框粘贴时崩溃的问题
* 修复一些停靠问题

### 已知问题

* Actipro 停靠：撕离窗格时，macOS 上不显示预览
* DevExpress 停靠：在 X11 上并不总是显示拖放提示
* Syncfusion 停靠：所有平台上都有问题
* Telerik 停靠：初始拖动/撕离在 Windows 上会停止注册鼠标