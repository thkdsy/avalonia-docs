---
id: custom-flyout
title: 如何创建自定义浮出控件
description: 通过在 Avalonia 中继承 FlyoutBase 来创建自定义浮出控件。
doc-type: how-to
---

自定义浮出控件可让你显示按需出现、附着在目标控件上的丰富且自包含的 UI。通过继承 `FlyoutBase`，你可以构建承载任意内容的浮出控件，从简单图片到完全可交互的表单都可以。

## 创建基础自定义浮出控件

要创建自定义浮出控件类型，请继承 `FlyoutBase`。你需要重写抽象方法 `CreatePresenter()`，以指定 [`Flyout`](/api/avalonia/controls/flyout) 应该使用哪个 presenter 来显示内容。这里可以返回任意类型的控件，但要注意它是内部弹出层的根内容，因此通常需要设置背景、边框、圆角等样式，以便与其他弹出控件保持一致。如果你愿意，也仍然可以使用普通的 `FlyoutPresenter`。

下面的示例创建了一个承载图片的简单 `Flyout`。

```csharp
public class MyImageFlyout : FlyoutBase
{
    public static readonly StyledProperty<IImage> ImageProperty = AvaloniaProperty.Register<MyImageFlyout, IImage>(nameof(Image));

    [Content]
    public IImage Image { get; set; }

    protected override Control CreatePresenter()
    {
        // In this example, we'll use the default FlyoutPresenter as the root content, and add an Image control to show our content
        return new FlyoutPresenter
        {
            Content = new Image
            {
                // 这里使用绑定，这样属性更新时图片会自动更新
                [!Image.SourceProperty] = this[!ImageProperty]
            }
        };
    }
}
```

## 显示与关闭

你可以通过调用 `ShowAt` 并传入它应锚定到的控件，以编程方式显示浮出控件。要关闭它，则调用 `Hide`。

```csharp
// 显示附着到某个控件上的浮出控件
var flyout = new MyImageFlyout { Image = myBitmap };
flyout.ShowAt(targetButton);

// 以代码方式关闭
flyout.Hide();
```

## 处理浮出控件事件

`FlyoutBase` 暴露了 `Opened` 和 `Closed` 事件，你可以订阅它们来响应可见性变化。

```csharp
flyout.Opened += (s, e) => { /* 浮出控件现已可见 */ };
flyout.Closed += (s, e) => { /* 浮出控件已关闭 */ };
```

## 带交互内容的自定义浮出控件

浮出控件并不局限于被动展示。你可以在 presenter 中包含按钮、文本输入框和其他交互控件。下面的示例创建了一个确认浮出控件：当用户点击按钮时，它会引发一个事件，然后自动关闭自身。

```csharp
public class ConfirmFlyout : FlyoutBase
{
    public event EventHandler? Confirmed;

    protected override Control CreatePresenter()
    {
        var confirmButton = new Button { Content = "Confirm" };
        confirmButton.Click += (s, e) =>
        {
            Confirmed?.Invoke(this, EventArgs.Empty);
            Hide();
        };

        return new FlyoutPresenter
        {
            Content = new StackPanel
            {
                Spacing = 8,
                Children =
                {
                    new TextBlock { Text = "Are you sure?" },
                    confirmButton
                }
            }
        };
    }
}
```

## 在 XAML 中使用

你可以在 XAML 中直接使用 `Flyout` 附加属性，将自定义浮出控件分配给支持它的控件，例如 `Button`。请确保在文件顶部声明了浮出控件类所在的 XML 命名空间。

```xml
<Button Content="Show Image">
    <Button.Flyout>
        <local:MyImageFlyout Image="{StaticResource MyImage}" />
    </Button.Flyout>
</Button>
```

## 另请参阅

- [自定义控件](/docs/custom-controls)
