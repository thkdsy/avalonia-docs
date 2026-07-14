---
id: routed-events
title: 路由事件
---

import InputEventRoutingDiagram from '/img/concepts/ui-concepts/user-input/pointer-pressed-routing.png';

Avalonia 中的大多数事件都实现为路由事件。所谓路由事件，是指它们会在整棵元素树上触发，而不仅仅局限于触发该事件的控件本身。

## 什么是路由事件？

一个典型的 Avalonia 应用通常包含许多元素。无论这些元素是通过代码创建，还是在 XAML 中声明，它们都会存在于一棵表示彼此关系的元素树中。根据事件定义的不同，事件路由可能朝两个方向之一传播；但一般来说，路由会从源元素开始，然后沿元素树向上“冒泡”，直到到达元素树根部（通常是页面或窗口）。如果你以前接触过 HTML DOM，那么这个冒泡概念应该会很熟悉。

### 路由事件的典型使用场景

下面简要总结了推动路由事件概念出现的几个场景，以及为什么普通 CLR 事件不足以应对这些场景：

**控件组合与封装：** Avalonia 中许多控件都拥有丰富的内容模型。例如，你可以将图片放进 [`Button`](/api/avalonia/controls/button) 中，这实际上扩展了按钮的可视树。但新增的图片不能破坏按钮的命中测试行为；即使用户点击的是技术上属于图片的像素区域，按钮也仍应对内容点击作出 `Click` 响应。

**单一处理器挂载点：** 在 Windows Forms 中，如果多个元素都可能触发某类事件，你通常需要把同一个处理器重复附加多次。路由事件则允许你只挂载一次处理器，如下例所示：

```xml
 <Border Height="50" Width="300">
  <StackPanel Orientation="Horizontal" Button.Click="CommonClickHandler">
    <Button Name="YesButton">Yes</Button>
    <Button Name="NoButton">No</Button>
    <Button Name="CancelButton">Cancel</Button>
  </StackPanel>
</Border>
```

```csharp
private void CommonClickHandler(object sender, RoutedEventArgs e)
{
  var source = e.Source as Control;
  switch (source.Name)
  {
    case "YesButton":
      // 在这里执行操作……
      break;
    case "NoButton":
      // 执行操作……
      break;
    case "CancelButton":
      // 执行操作……
      break;
  }
  e.Handled=true;
}
```

**类处理：** 路由事件允许由类本身定义静态处理器。类处理器有机会在任何实例级处理器执行之前优先处理该事件。

**无需反射即可引用事件：** 某些代码和标记语法场景需要一种方式来标识具体事件。路由事件会创建一个 [`RoutedEvent`](/api/avalonia/interactivity/routedevent) 字段作为标识符，这提供了一种稳健的事件识别方式，而不依赖静态或运行时反射。

### 路由事件是如何实现的

路由事件本质上是一个 CLR 事件，它由 `RoutedEvent` 类的实例提供支撑，并注册到 Avalonia 的事件系统中。注册得到的 `RoutedEvent` 实例通常会作为注册该事件、也就是“拥有”该路由事件的类上的 `public static readonly` 字段保留下来。它与同名 CLR 事件（有时称为“包装事件”）之间的连接，是通过重写 CLR 事件的 `add` 和 `remove` 实现来完成的。通常情况下，`add` 和 `remove` 会保留为默认的隐式实现，以支持使用语言本身的事件语法来添加和移除处理器。从概念上看，路由事件的支撑和连接机制，与 Avalonia 属性作为由 `AvaloniaProperty` 支撑并注册到属性系统中的 CLR 属性非常相似。

The following example shows the declaration for a custom `Tap` routed event, including the registration and exposure of the `RoutedEvent` identifier field and the `add` and `remove` implementations for the `Tap` CLR event.

```csharp
public class SampleControl: Control
{
  public static readonly RoutedEvent<RoutedEventArgs> TapEvent =
    RoutedEvent.Register<SampleControl, RoutedEventArgs>(nameof(Tap), RoutingStrategies.Bubble);

  // Provide CLR accessors for the event
  public event EventHandler<RoutedEventArgs> Tap
  { 
    add => AddHandler(TapEvent, value);
    remove => RemoveHandler(TapEvent, value);
  }
}
```

### 路由事件处理器与 XAML

如果要在 XAML 中为事件添加处理器，你需要把事件名声明为监听该事件的元素上的一个属性。这个属性的值就是你实现的处理器方法名，并且该方法必须存在于对应 code-behind 文件所属的类中。

```xml
<Button Click="b1SetColor">按钮</Button>
```

为标准 CLR 事件添加处理器的 XAML 语法，与添加路由事件处理器的语法是相同的，因为你实际上是在向 CLR 事件包装器添加处理器，而该包装器底层实现的正是路由事件。

## 路由策略

路由事件会使用以下三种路由策略之一：

* **冒泡（Bubbling）：** 首先调用事件源上的事件处理器，然后路由事件会沿着父元素逐级向上传播，直到抵达元素树根部。大多数路由事件都使用冒泡策略。冒泡路由事件通常用于报告来自不同控件或其他 UI 元素的输入或状态变化。
* **直接（Direct）：** 只有源元素自身有机会响应该事件并调用处理器。这类似于 Windows Forms 中事件的“路由”方式。不过，与标准 CLR 事件不同，直接路由事件也支持类处理（后文会解释）。
* **隧道（Tunneling）：** 首先调用元素树根部的事件处理器，然后路由事件沿着通往事件源节点（即触发该路由事件的元素）的路径，经过各级子元素向下传播。隧道路由事件常用于控件组合场景，以便有选择地抑制来自组合部件的事件，或将其替换为更符合完整控件语义的事件。Avalonia 提供的输入事件通常同时会触发隧道事件和冒泡事件。

## 为什么要使用路由事件？

作为应用开发者，你并不总是需要知道或关心自己处理的事件是否实现为路由事件。路由事件确实具有特殊行为，但如果你只是在事件源元素本身上处理该事件，这些行为通常并不明显。

路由事件真正强大的地方，在于你是否用到了前面提到的那些场景：例如在公共根节点上定义统一处理器、组合你自己的控件，或定义自定义控件类。

路由事件监听器和事件源不需要在它们的继承层级中共享某个共同事件。任何控件都可以成为任何路由事件的监听器。因此，你可以把整个 API 中可用的路由事件集合视为一种概念上的“接口”，让应用中原本彼此分离的元素交换事件信息。对于输入事件来说，这种“接口”概念尤其适用。

路由事件还可以用于沿元素树传递信息，因为事件数据会在路由路径上的每个元素之间持续传递。某个元素可以修改事件数据，而这些变化会对路由中的下一个元素可见。

除了路由机制本身外，Avalonia 中某个事件之所以实现为路由事件而不是标准 CLR 事件，通常还有另外两个原因。如果你要实现自己的事件，也可以参考这些原则：

* 某些样式和模板特性要求被引用的事件必须是路由事件。这就是前面提到的“事件标识符”场景。
* 路由事件支持类处理机制，允许类通过静态方法在任何已注册的实例处理器执行之前优先处理事件。这在控件设计中非常有用，因为你的类可以强制执行某些基于事件的类行为，而不会被某个实例上的事件处理意外屏蔽掉。

上面这些考虑因素，都会在本主题后续章节中分别讨论。

## 为路由事件添加并实现事件处理器

如果要在 XAML 中添加事件处理器，请将事件名作为元素属性写上，并把该属性值设为实现了适当委托签名的事件处理器名称，如下例所示。

```xml
<Button Click="b1SetColor">按钮</Button>
```

`b1SetColor` 是已实现处理器的名称，其中包含处理 `Click` 事件的代码。`b1SetColor` 的签名必须与 `RoutedEventHandler<RoutedEventArgs>` 委托一致，这个委托正是 `Click` 事件所使用的事件处理器委托。所有路由事件处理器委托的第一个参数都表示附加了处理器的元素，第二个参数则表示该事件的数据。

```csharp
void b1SetColor(object sender, RoutedEventArgs args)
{
  // 处理 Click 事件的逻辑
}
```

`RoutedEventHandler<RoutedEventArgs>` 是最基础的路由事件处理器委托。对于某些特定控件或场景专用的路由事件，处理器所使用的委托类型也可能更加具体，以便传递更专门的事件数据。例如，在常见输入场景中，你可能会处理 `PointerPressed` 路由事件，此时处理器应实现 `RoutedEventHandler<PointerPressedEventArgs>` 委托。使用更具体的委托后，你就可以在处理器中直接处理 `PointerPressedEventArgs`，并读取 `PointerEventArgs.Pointer` 属性，以获取触发按下事件的指针信息。

在纯代码创建的应用中，为路由事件添加处理器也很直接。你始终可以通过辅助方法 `AddHandler` 来添加路由事件处理器（底层 `add` 调用的其实也是这个方法）。不过，Avalonia 中现有的大多数路由事件都实现了 `add` 和 `remove` 的包装逻辑，因此也支持使用语言本身的事件语法来添加处理器，这通常比辅助方法更直观。下面是使用辅助方法的示例：

```csharp
void MakeButton()
{
    Button b2 = new Button();
    b2.AddHandler(Button.ClickEvent, Onb2Click);
}

void Onb2Click(object sender, RoutedEventArgs e)
{
    // 处理 Click 事件的逻辑
}
```

下一个示例展示了 C# 的运算符语法：

```csharp
void MakeButton2()
{
  Button b2 = new Button();
  b2.Click += Onb2Click2;
}

void Onb2Click2(object sender, RoutedEventArgs e)
{
  // 处理 Click 事件的逻辑
}
```

**Handled 的概念**

所有路由事件都共享同一个事件数据基类 `RoutedEventArgs`。`RoutedEventArgs` 定义了 `Handled` 属性，它是一个布尔值。`Handled` 属性的作用，是让路由路径上的任意事件处理器都能通过把 `Handled` 设为 `true`，将该路由事件标记为*已处理*。当某个元素上的处理器处理完事件后，共享的事件数据仍会继续传递给路由中的后续监听器。

`Handled` 的值会影响路由事件在继续沿路径传播时如何被报告和处理。如果某个路由事件的事件数据中 `Handled` 为 `true`，那么其他元素上监听该事件的处理器通常就不会再针对这次事件实例被调用。无论这些处理器是在 XAML 中附加，还是通过 `+=` 这样的语言语法附加，都是如此。对大多数常见场景来说，把 `Handled` 设为 `true` 会“停止”隧道路由或冒泡路由中的常规处理流程；同样，对于在路由中某一点由类处理器先行处理的事件也适用。

不过，Avalonia 还提供了一个 `handledEventsToo` 机制，让监听器即使在事件数据的 `Handled` 为 `true` 时，仍然可以继续响应该路由事件。换句话说，把事件标记为已处理，并不意味着事件路由会被真正完全中断。`handledEventsToo` 机制只能在代码中使用：

* 在代码中，不要使用适用于普通 CLR 事件的语言级事件语法，而应调用 Avalonia 的 `AddHandler<TEventArgs>(RoutedEvent<TEventArgs>, EventHandler<TEventArgs> handler, RoutingStrategies, bool)` 方法来添加处理器，并将 `handledEventsToo` 参数设为 `true`。

除了在路由事件中的行为效果外，`Handled` 这个概念还会影响你如何设计应用程序以及如何编写事件处理代码。你可以把 `Handled` 理解为路由事件暴露出来的一种简单协议。至于如何使用这个协议，取决于你的具体需求，但它的设计意图大致如下：

* 如果某个路由事件已经被标记为已处理，那么沿这条路由上的其他元素通常就不再需要再次处理它。
* 如果某个路由事件尚未被标记为已处理，那么说明更早出现在路由上的监听器要么没有注册处理器，要么即使注册了处理器，也选择不修改事件数据并将 `Handled` 设为 `true`。（当然，也有可能当前监听器本身就是路由中的第一个节点。）此时，当前监听器上的处理器通常有三种可能的做法：
  * 完全不做任何事情；事件保持未处理状态，并继续路由到下一个监听器。
  * 执行一些响应逻辑，但判断其影响还不足以将事件标记为已处理；事件继续路由到下一个监听器。
  * 执行响应逻辑，并认为该逻辑足够关键，因此将传入处理器的事件数据标记为已处理。此时事件仍会继续路由到下一个监听器，但事件数据中的 `Handled=true`，因此只有设置了 `handledEventsToo` 的监听器才有机会继续执行处理器。

前面提到的路由行为也强化了这种设计理念：如果之前某个处理器已经将 `Handled` 设为 `true`，那么再去附加那些仍希望继续响应该路由事件的处理器就会更麻烦一些（尽管在代码或样式中依然是可行的）。

在实际应用中，人们经常只在触发事件的对象本身上处理冒泡路由事件，而完全不关心它的路由特性。不过，即便如此，仍然建议在事件数据中将该路由事件标记为已处理，以防止元素树中更上层的某个元素恰好也附加了同一个路由事件处理器，从而带来意料之外的副作用。

## 类处理器

如果你定义的类以某种方式继承自 `AvaloniaObject`，那么也可以为一个已声明或继承到你这个类上的路由事件定义并附加类处理器。每当某个路由事件传播到该类的某个元素实例时，类处理器都会在附加到该实例上的任何实例级监听器处理器之前先被调用。

某些 Avalonia 控件天生就会对特定路由事件进行类处理。这可能会让你从表面上觉得该路由事件似乎从未被触发，但实际上它只是被类处理器提前处理了；如果你使用合适的技术，你的实例处理器仍然可能继续处理它。此外，许多基类和控件还公开了可重写的虚方法，可用于覆盖类处理行为。

如果要在你自己的控件中附加类处理器，请在静态构造函数中使用 `AddClassHandler` 方法：

```csharp
static MyControl()
{
    MyEvent.AddClassHandler<MyControl>((x, e) => x.OnMyEvent(e));
}

protected virtual void OnMyEvent(MyEventArgs e)
{
    // 在这里处理事件。
}
```

## Avalonia 中的附加事件

XAML 语言还定义了一种特殊事件类型，称为*附加事件*。附加事件允许你把某个特定事件的处理器添加到任意元素上。处理该事件的元素不需要自己定义或继承这个附加事件；同样，可能触发该事件的对象和目标处理实例，也都不需要把这个事件定义为自身的类成员，或“拥有”它。

Avalonia 输入系统大量使用附加事件。不过，几乎所有这类附加事件都会通过基础元素转发，因此这些输入事件最终表现为基础元素类上的等价非附加路由事件。例如，底层的 `Tapped` 事件可以直接在任意 `InputElement` 上处理，而无需在 XAML 或代码中显式使用附加事件语法。

## XAML 中的限定事件名

还有一种语法形式，看起来很像 _类型名_._事件名_ 的附加事件语法，但严格来说它并不是附加事件的用法，那就是为子元素所触发的路由事件附加处理器。你会把处理器附加到一个公共父元素上，以便利用事件路由机制，即使这个父元素本身并不拥有该路由事件作为成员。请再次看一下前面在[本页较早位置](#top-level-scenarios-for-routed-events)分析过的示例。

```xml
<Border Height="50" Width="300">
  <StackPanel Orientation="Horizontal" Button.Click="CommonClickHandler">
    <Button Name="YesButton">是</Button>
    <Button Name="NoButton">否</Button>
    <Button Name="CancelButton">取消</Button>
  </StackPanel>
</Border>
```

在这里，添加处理器的父元素监听器是 `StackPanel`。但它实际上添加的是一个由 `Button` 类声明并触发的路由事件处理器。也就是说，虽然该事件是 `Button` “拥有”的，但路由事件系统允许你把任何路由事件的处理器附加到任意控件实例监听器上——前提是这个控件原本就支持附加普通公共语言运行时（CLR）事件监听器。这类限定事件属性名默认通常位于 Avalonia 默认的 xmlns 命名空间中，但对于自定义路由事件，你也可以显式指定带前缀的命名空间。

## 输入事件

在 Avalonia 平台中，路由事件最常见的用途之一就是输入事件。输入事件通常成对出现，其中一个是冒泡事件，另一个是隧道事件。也有一些输入事件只提供冒泡版本，或者只提供直接路由版本。

Avalonia 中成对出现的输入事件，其实现方式决定了：一次单独的用户输入动作（例如按下鼠标按钮）会按顺序触发这一对路由事件。首先触发隧道事件，并沿隧道路径传播；随后触发冒泡事件，并沿冒泡路径传播。这两个事件实际上共享同一个事件数据实例，因为实现冒泡事件触发的类在调用 `RaiseEvent` 时，会监听来自隧道事件的事件数据，并将其复用于新的事件触发过程中。监听隧道事件的处理器拥有第一次机会将该路由事件标记为已处理（先执行类处理器，再执行实例处理器）。如果隧道路径上的某个元素已经将其标记为已处理，那么这个已经被处理过的事件数据就会继续被传递给后续的冒泡事件，而附加在对应冒泡输入事件上的常规处理器通常将不会再被调用。从外部效果上看，就好像那个被处理过的冒泡事件根本没有被触发一样。这种处理行为对于控件组合非常有用：在这类场景中，你往往希望所有基于命中测试的输入事件或基于焦点的输入事件，都由最终控件本身统一对外报告，而不是由其内部的组合部件分别暴露。由于最终控件在组合结构中更接近根部，因此它有机会优先对隧道事件进行类处理，并在支撑控件类的代码中将该路由事件“替换”为一个更贴近完整控件语义的新事件。

为了说明输入事件处理的工作方式，请看下面这个输入事件示例。在下图的树结构中，`leaf element #2` 是 `PointerPressed` 事件的源头：

<Image light={InputEventRoutingDiagram} alt="事件路由示意图" position="center" maxWidth={400} cornerRadius="true"/>

事件处理顺序如下：

1. 根元素上的 `PointerPressed`（隧道）。
2. 中间元素 #1 上的 `PointerPressed`（隧道）。
3. 源元素 #2 上的 `PointerPressed`（隧道）。
4. 源元素 #2 上的 `PointerPressed`（冒泡）。
5. 中间元素 #1 上的 `PointerPressed`（冒泡）。
6. 根元素上的 `PointerPressed`（冒泡）。

路由事件处理器委托会提供对两个对象的引用：一个是触发事件的对象，另一个是处理器被调用时所在的对象。处理器所在的对象由 `sender` 参数表示；事件最初触发的对象则由事件数据中的 `Source` 属性表示。某个路由事件也可能既由同一个对象触发，又由同一个对象处理，此时 `sender` 和 `Source` 就是相同的（在上面事件顺序列表中的第 3 步和第 4 步就是这种情况）。

由于存在隧道和冒泡机制，父元素往往会收到一些输入事件，而这些事件的 `Source` 实际上是它们的某个子元素。如果你需要知道真正的源元素是谁，可以通过访问 `Source` 属性来识别它。

通常情况下，一旦输入事件被标记为 `Handled`，后续处理器就不会再被调用。一般来说，只要某个处理器已经完成了与你的应用逻辑相关的输入处理，就应尽快把该输入事件标记为已处理。

这一规则的例外是：如果某个输入事件处理器在注册时被明确设置为忽略事件数据中的 `Handled` 状态，那么它依然会在隧道路径或冒泡路径上继续被调用。

某些类会选择对某些输入事件进行类处理，通常其目的在于：在该控件内部重新定义某种用户输入事件的含义，并据此触发一个新的事件。

## 另请参阅

- [添加交互性](/docs/input-interaction/adding-interactivity)：事件与命令概览。
- [指针事件](/docs/input-interaction/pointer)：指针设备事件与捕获。
- [手势](/docs/input-interaction/gestures)：构建于指针事件之上的更高层手势事件。
