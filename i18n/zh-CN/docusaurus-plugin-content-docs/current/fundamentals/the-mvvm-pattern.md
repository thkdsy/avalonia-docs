---
id: the-mvvm-pattern
title: MVVM 模式
description: 使用 Model-View-ViewModel 模式和数据绑定，将 UI 与逻辑分离。
doc-type: explanation
---

import MvvmPatternDiagram from '/img/concepts/architecture/mvvm/mvvm-architecture.png';
import MvvmDataBindingDiagram from '/img/concepts/architecture/mvvm/mvvm.png';

Model-View-ViewModel（MVVM）模式用于将应用程序的用户界面与其逻辑分离。它不会把显示代码和行为逻辑混在同一个文件中，而是将它们拆分为三个彼此独立的部分，并通过数据绑定进行通信。

<Image light={MvvmPatternDiagram} alt="Diagram showing the three components of the MVVM pattern: Model, View, and View Model." position="center" maxWidth={400} cornerRadius="true"/>

- **View**：用户看到的结构、布局和外观。在 Avalonia 中，视图通常定义在 AXAML 文件中，并只保留少量 code-behind。视图通过绑定从视图模型获取数据。
- **View model**：视图与模型之间的中介层。它向视图公开数据和命令，处理用户交互逻辑，并通过触发 `PropertyChanged` 事件来通知视图数据已发生变化。
- **Model**：应用程序的领域层，包括数据访问、业务逻辑和验证。例如仓储、数据传输对象和服务客户端等。

视图了解视图模型，视图模型了解模型，但反过来并不成立。模型不知道视图模型的存在，视图模型也不知道视图的存在。正是这种单向依赖链，使得每一层都可以独立测试和替换。

## 为什么使用 MVVM？

随着应用程序规模增长，如果继续把 UI 定义和应用逻辑放在同一个 code-behind 文件中，就会逐渐出现问题。控件之间的交互会越来越混乱，而且由于逻辑与 UI 平台紧密耦合，单元测试也会变得困难。

MVVM 通过把应用逻辑转移到 POCO（Plain Old CLR Objects）中来解决这个问题，这些对象不依赖 Avalonia，也不依赖任何 UI 框架。这样做的好处包括：

- **可测试性**：视图模型可以像普通类一样进行单元测试，而不需要启动 UI。
- **关注点分离**：UI 布局与应用逻辑可以独立演进。你可以重新设计视图，而无需修改视图模型。
- **天然适配 XAML**：Avalonia 的数据绑定系统为视图和视图模型提供了连接方式，因此 MVVM 与其天然契合。

## 何时使用 MVVM

与 [code-behind](/docs/fundamentals/code-behind) 模式相比，MVVM 会增加一定复杂度。对于小型且简单的应用程序来说，code-behind 可能更容易理解和维护。

你可以考虑以下两种策略：

1. 先从 code-behind 开始，如果应用程序逐渐变得难以维护，再迁移到 MVVM。
2. 如果你预计应用程序规模会持续增长，那么从一开始就使用 MVVM。

## Avalonia 中的 MVVM

### 视图与视图模型

你通常会使用 AXAML 文件及其 code-behind 来实现视图，并使用一个普通的 C#（或 F#）类来实现视图模型。每个视图都对应一个视图模型，其中包含该视图所需的全部逻辑。

视图通常由 Avalonia 的[内置控件](/controls)、[用户控件](/docs/fundamentals/ui-composition)以及可选的、你自己设计的[自定义控件](/docs/custom-controls)组合而成。

### 数据绑定

数据绑定是连接视图与视图模型的关键技术。你可以把这种关系理解为通过绑定连接在一起的两层结构：

<Image light={MvvmDataBindingDiagram} alt="Diagram showing data bindings connecting a view layer to a view model layer." position="center" maxWidth={400} cornerRadius="true"/>

有些绑定是双向的。例如，一个文本输入框可以双向绑定：视图模型中的变化会更新控件，而用户输入也会回流到视图模型。另一些绑定则是单向的。例如按钮的命令绑定通常只从视图流向视图模型。

由于视图模型既不引用视图，也不依赖 Avalonia 类型，因此它可以像普通代码一样进行单元测试。

### 模型层

模型代表 UI 之外的一切内容，例如数据存储、网络服务和业务规则。MVVM 并不会强制规定你必须如何组织模型层，但最重要的原则是分离。建议使用依赖注入把模型服务提供给视图模型，而不是让它们之间产生紧耦合。

## 另请参阅

- [Code-behind](/docs/fundamentals/code-behind)
- [UI composition](/docs/fundamentals/ui-composition)
- [Introduction to data binding](/docs/data-binding/introduction-to-data-binding)
