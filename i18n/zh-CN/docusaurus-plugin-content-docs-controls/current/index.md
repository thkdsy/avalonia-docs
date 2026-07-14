---
id: index
title: 控件
sidebar_label: 首页
hide_table_of_contents: true
---

import DocsCard from '@site/src/components/global/DocsCard';
import DocsCards from '@site/src/components/global/DocsCards';

<head>
  <title>Avalonia 文档：控件库</title>
  <meta
    name="description"
    content="Avalonia 提供了大量高质量 UI 控件，包括按钮、列表、选项卡等，帮助你快速而轻松地构建应用界面。"
  />
  <style>{`
    :root {
      --doc-item-container-width: 60rem;
    }
  `}</style>
</head>

Avalonia 应用由称为控件的高级构建块组成，它们可以帮助你快速搭建应用的 UI。Avalonia 提供了大量控件，包括按钮、列表、选项卡等。本节页面将介绍 Avalonia 控件的核心功能。更详细的信息请参阅 [API 参考](/api)。


<DocsCards>
<DocsCard header="边框" href="/controls/layout/containers/border" icon="/icons/border-icon@2x.png">
  <p>一种装饰器控件，可在其子内容周围绘制边框和背景。</p>
</DocsCard>

<DocsCard header="按钮" href="/controls/input/buttons/button" icon="/icons/button-icon@2x.png">
  <p>按钮让用户执行操作，是与应用交互和进行导航的基础方式之一。</p>
</DocsCard>

<DocsCard header="日历" href="/controls/input/date-and-time/calendar" icon="/icons/calendar-icon@2x.png">
  <p>完整的日历视图，让用户能够以可视化方式浏览并选择日期。</p>
</DocsCard>

<DocsCard header="复选框" href="/controls/input/selectors/checkbox" icon="/icons/checkbox-icon@2x.png">
  <p>复选框适用于二元选择场景，例如功能开关、问卷选项或任务清单。</p>
</DocsCard>

<DocsCard header="ComboBox" href="/controls/input/selectors/combobox" icon="/icons/combobox-icon@2x.png">
  <p>ComboBox 会显示当前选中项，并在点击时展开可选项列表。</p>
</DocsCard>

<DocsCard header="日期选择器" href="/controls/input/date-and-time/datepicker" icon="/icons/datepicker-icon@2x.png">
  <p>日期选择器为用户提供紧凑的日期选择界面。</p>
</DocsCard>

<DocsCard header="网格" href="/controls/layout/panels/grid" icon="/icons/grid-icon@2x.png">
  <p>Grid 是功能强大的布局控件，可按行和列排列其他控件。</p>
</DocsCard>

<DocsCard header="图像" href="/controls/media/image" icon="/icons/image-icon@2x.png">
  <p>用于显示图像，并控制它们如何与其他 UI 元素协同工作。</p>
</DocsCard>

<DocsCard header="标签" href="/controls/data-display/text-display/label" icon="/icons/label-icon@2x.png">
  <p>一种文本标签控件，可为其目标控件提供访问键支持。</p>
</DocsCard>

<DocsCard header="ListBox" href="/controls/data-display/collections/listbox" icon="/icons/listbox-icon@2x.png">
  <p>列表用于展示多行信息，例如联系人、语言或音乐风格等。</p>
</DocsCard>

<DocsCard header="Markdown" href="/controls/data-display/text-display/markdown" icon="/icons/markdownrenderer-icon@2x.png">
  <p>可在应用中直接渲染并显示 Markdown 格式的文本内容。</p>
</DocsCard>

<DocsCard header="媒体播放器" href="/controls/media/mediaplayercontrol" icon="/icons/mediaplayer-icon@2x.png">
  <p>功能完整的媒体播放器控件，可用于播放音频和视频内容。</p>
</DocsCard>

<DocsCard header="菜单" href="/controls/menus/menu" icon="/icons/menu-icon@2x.png">
  <p>菜单可用于组织用户选项，并提升导航体验。</p>
</DocsCard>

<DocsCard header="屏幕键盘" href="/controls/input/text-input/virtualkeyboard" icon="/icons/onscreenkeyboard-icon@2x.png">
  <p>面向无物理键盘设备的虚拟键盘，适用于触控文本输入场景。</p>
</DocsCard>

<DocsCard header="面板" href="/controls/layout/panels/panel" icon="/icons/panel-icon@2x.png">
  <p>所有面板元素的基类，用于定位并排列子控件。</p>
</DocsCard>

<DocsCard header="单选按钮" href="/controls/input/buttons/radiobutton" icon="/icons/radiobutton-icon@2x.png">
  <p>单选按钮适用于呈现一组互斥选项。</p>
</DocsCard>

<DocsCard header="矩形" href="/docs/graphics-animation/drawing-graphics" icon="/icons/rectangle-icon@2x.png">
  <p>用于在 UI 中绘制矩形和正方形的基础图形元素。</p>
</DocsCard>

<DocsCard header="ScrollViewer" href="/controls/layout/containers/scrollviewer" icon="/icons/scrollviewer-icon@2x.png">
  <p>当子内容超出可用空间时，提供滚动能力的容器控件。</p>
</DocsCard>

<DocsCard header="滑块" href="/controls/input/selectors/slider" icon="/icons/slider-icon@2x.png">
  <p>滑块允许用户通过沿轨道移动旋钮来选择数值。</p>
</DocsCard>

<DocsCard header="分栏视图" href="/controls/layout/containers/splitview" icon="/icons/splitview-icon@2x.png">
  <p>一种带可折叠侧栏和内容区域的容器，非常适合导航布局。</p>
</DocsCard>

<DocsCard header="堆叠面板" href="/controls/layout/panels/stackpanel" icon="/icons/stackpanel-icon@2x.png">
  <p>将子控件排列成单行，可沿水平方向或垂直方向堆叠。</p>
</DocsCard>

<DocsCard header="选项卡控件" href="/controls/navigation/tabcontrol" icon="/icons/tabcontrol-icon@2x.png">
  <p>选项卡支持标签式导航，这是现代应用中的常见导航模式。</p>
</DocsCard>

<DocsCard header="文本块" href="/controls/data-display/text-display/textblock" icon="/icons/textblock-icon@2x.png">
  <p>用于显示少量只读文本的轻量级控件。</p>
</DocsCard>

<DocsCard header="文本框" href="/controls/input/text-input/textbox" icon="/icons/textbox-icon@2x.png">
  <p>文本框用于创建文本输入区域，几乎是各类应用中的基础控件。</p>
</DocsCard>

<DocsCard header="切换开关" href="/controls/input/buttons/togglebutton" icon="/icons/toggleswitch-icon@2x.png">
  <p>切换开关让用户在开与关之间切换，适用于设置等二元选项。</p>
</DocsCard>

<DocsCard header="树视图" href="/controls/data-display/structured-data/treeview" icon="/icons/treeview-icon@2x.png">
  <p>以可展开的树形结构显示层级数据。</p>
</DocsCard>

<DocsCard header="统一网格" href="/controls/layout/panels/uniformgrid" icon="/icons/uniformgrid-icon@2x.png">
  <p>一种会自动将所有单元格调整为相同大小的网格面板，用于创建均匀布局。</p>
</DocsCard>

<DocsCard header="WebView" href="/controls/web/nativewebview" icon="/icons/webview-icon@2x.png">
  <p>通过原生浏览器引擎将 Web 内容直接嵌入到应用中。</p>
</DocsCard>

</DocsCards>
