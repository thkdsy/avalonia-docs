---
id: attached-properties
title: 如何创建附加属性
description: 定义可设置到其他控件上的附加属性，例如布局定位属性。
doc-type: how-to
---

当你需要为 Avalonia 元素添加一些并不属于其自身类定义的额外属性时，附加属性就是正确的工具。你也可以利用它们创建能够修改宿主控件行为的功能。例如，可以通过附加属性把一个命令绑定到某个事件上。

下面的示例展示了如何以兼容 MVVM 的方式使用命令，并将其绑定到某个事件上。这并不是唯一做法（像 [Avalonia Behaviors](https://github.com/wieslawsoltes/AvaloniaBehaviors) 这样的项目提供了更完整的解决方案），但它很好地说明了两个关键概念：

* 如何在 Avalonia 中创建附加属性
* 如何让它们与 MVVM 配合使用

## 注册附加属性

使用 `AvaloniaProperty.RegisterAttached` 方法来注册附加属性。按照约定：

* 附加属性对应的 **public static** CLR 字段命名为 _XxxxProperty_。
* 附加属性的 name 参数为 _Xxxx_（不带 _Property_ 后缀）。
* 你必须提供两个 **public static** 方法：_SetXxxx(element, value)_ 和 _GetXxxx(element)_。

注册调用需要指定属性类型、owner 类型，以及该属性可被设置到的目标类型。

验证回调可以通过返回修正后的值来清理输入值，也可以通过返回 `AvaloniaProperty.UnsetValue` 来丢弃这次修改。getter 和 setter 方法本身只应负责读取和写入值。绑定系统会识别这种命名约定，并直接设置这些属性。

下面的示例创建了两个彼此配合的附加属性：一个 `Command` 属性，以及一个在执行命令时传递进去的 `CommandParameter`。

```csharp
/// <summary>
/// Container class for attached properties. Must inherit from <see cref="AvaloniaObject"/>.
/// </summary>
public class DoubleTappedBehav : AvaloniaObject
{
    static DoubleTappedBehav()
    {
        CommandProperty.Changed.AddClassHandler<Interactive>(HandleCommandChanged);
    }

    /// <summary>
    /// Identifies the <seealso cref="CommandProperty"/> avalonia attached property.
    /// </summary>
    /// <value>Provide an <see cref="ICommand"/> derived object or binding.</value>
    public static readonly AttachedProperty<ICommand> CommandProperty = AvaloniaProperty.RegisterAttached<DoubleTappedBehav, Interactive, ICommand>(
        "Command", default(ICommand), false, BindingMode.OneTime);

    /// <summary>
    /// Identifies the <seealso cref="CommandParameterProperty"/> avalonia attached property.
    /// Use this as the parameter for the <see cref="CommandProperty"/>.
    /// </summary>
    /// <value>Any value of type <see cref="object"/>.</value>
    public static readonly AttachedProperty<object> CommandParameterProperty = AvaloniaProperty.RegisterAttached<DoubleTappedBehav, Interactive, object>(
        "CommandParameter", default(object), false, BindingMode.OneWay, null);


    /// <summary>
    /// <see cref="CommandProperty"/> changed event handler.
    /// </summary>
    private static void HandleCommandChanged(Interactive interactElem, AvaloniaPropertyChangedEventArgs args)
    {
        if (args.NewValue is ICommand commandValue)
        {
             // Add non-null value
             interactElem.AddHandler(InputElement.DoubleTappedEvent, Handler);
        }
        else
        {
             // remove prev value
             interactElem.RemoveHandler(InputElement.DoubleTappedEvent, Handler);
        }
        // local handler fcn
        static void Handler(object s, RoutedEventArgs e)
        {
            if (s is Interactive interactElem)
            {
                // This is how we get the parameter off of the gui element.
                object commandParameter = interactElem.GetValue(CommandParameterProperty);
                ICommand commandValue = interactElem.GetValue(CommandProperty);
                if (commandValue?.CanExecute(commandParameter) == true)
                {
                    commandValue.Execute(commandParameter);
                }
            }
        }
    }


    /// <summary>
    /// Accessor for Attached property <see cref="CommandProperty"/>.
    /// </summary>
    public static void SetCommand(AvaloniaObject element, ICommand commandValue)
    {
        element.SetValue(CommandProperty, commandValue);
    }

    /// <summary>
    /// Accessor for Attached property <see cref="CommandProperty"/>.
    /// </summary>
    public static ICommand GetCommand(AvaloniaObject element)
    {
        return element.GetValue(CommandProperty);
    }

    /// <summary>
    /// Accessor for Attached property <see cref="CommandParameterProperty"/>.
    /// </summary>
    public static void SetCommandParameter(AvaloniaObject element, object parameter)
    {
        element.SetValue(CommandParameterProperty, parameter);
    }

    /// <summary>
    /// Accessor for Attached property <see cref="CommandParameterProperty"/>.
    /// </summary>
    public static object GetCommandParameter(AvaloniaObject element)
    {
        return element.GetValue(CommandParameterProperty);
    }
}

```

这个类会监听 `CommandProperty` 的变化，并使用路由事件系统来附加或移除处理器。处理器通过 `GetValue()` 读取这些属性值。

## 在 XAML 中使用附加属性

在 XAML 中声明命名空间后，你就可以使用点号语法来设置附加属性。绑定也会像预期那样正常工作。

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:loc="clr-namespace:MyApp.Behaviors"
             x:Class="MyApp.Views.TestView">
    <ListBox ItemsSource="{Binding Accounts}"
             SelectedIndex="{Binding SelectedAccountIdx, Mode=TwoWay}"
             loc:DoubleTappedBehav.Command="{Binding EditCommand}"
             loc:DoubleTappedBehav.CommandParameter="test77"
             >
      <ListBox.ItemTemplate>
        <DataTemplate>
          <TextBlock Text="{Binding }" />          
        </DataTemplate>
      </ListBox.ItemTemplate>
    </ListBox>
</UserControl>
```

## 在视图模型中处理命令

虽然本示例中的 `CommandParameter` 使用的是静态值，但它同样也可以进行数据绑定。与下面这个视图模型配合使用时，只要发生双击手势，就会执行 `EditCommandExecuted`。

```csharp
public class TestViewModel : ReactiveObject
{
    public ObservableCollection<Profile> Accounts { get; } = new ObservableCollection<Profile>();

    public ReactiveCommand<object, Unit> EditCommand { get; set; }

    public TestViewModel()
    {
        EditCommand = ReactiveCommand.CreateFromTask<object, Unit>(EditCommandExecuted);
    }

    private async Task<Unit> EditCommandExecuted(object p)
    {
        // p contains "test77"

        return Unit.Default;
    }
}
```

## 另请参阅

- [Defining Properties](/docs/custom-controls/defining-properties)
- [Creating Custom Controls](/docs/custom-controls)
- [Routed Events](/docs/input-interaction/routed-events)
