---
id: how-to-create-a-custom-data-binding-converter
title: 如何创建自定义数据绑定转换器
description: 通过实现 IValueConverter，在 Avalonia 绑定中转换源值与目标值。
doc-type: how-to
---


当内置数据绑定转换器无法满足你的转换需求时，你可以基于 [`IValueConverter`](/api/avalonia/data/converters/ivalueconverter) 接口编写自定义转换器。本文将介绍具体做法。

:::info
如果你想查看 _Microsoft_ 关于 `IValueConverter` 接口的文档，请参阅 [IValueConverter API reference](https://docs.microsoft.com/en-gb/dotnet/api/system.windows.data.ivalueconverter?view=netframework-4.7.1)。
:::

:::info
由于 `IValueConverter` 接口在 .NET Standard 2.0 中并不存在，Avalonia UI 在 `Avalonia.Data.Converters` 命名空间中提供了一份自己的定义。你可以查看 [Avalonia IValueConverter API documentation](/api/avalonia/data/converters/ivalueconverter)。
:::

在使用自定义转换器之前，你必须先在某个资源作用域中引用它。这个资源可以放在应用程序的任意层级。下面的示例把自定义转换器 `myConverter` 放在窗口资源中：

```xml
<Window xmlns="https://github.com/avaloniaui"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:local="clr-namespace:ExampleApp;assembly=ExampleApp">

  <Window.Resources>
    <local:MyConverter x:Key="myConverter"/>
  </Window.Resources>

  <TextBlock Text="{Binding Value, Converter={StaticResource myConverter}}"/>
</Window>
```

## 示例

下面这个数据绑定转换器示例，可以根据参数把文本转换为特定大小写：

```xml
<TextBlock Text="{Binding TheContent, 
    Converter={StaticResource textCaseConverter},
    ConverterParameter=lower}" />
```

上面的 XAML 假定 `textCaseConverter` 已经在资源中声明好了。

```csharp
public class TextCaseConverter : IValueConverter
{
    public static readonly TextCaseConverter Instance = new();

    public object? Convert(object? value, Type targetType, object? parameter, 
                                                            CultureInfo culture)
    {
        if (value is string sourceText && parameter is string targetCase
            && targetType.IsAssignableTo(typeof(string)))
        {
            switch (targetCase)
            {
                case "upper":
                case "SQL":
                    return sourceText.ToUpper();
                case "lower":
                    return sourceText.ToLower();
                case "title": // Every First Letter Uppercase
                    var txtinfo = new System.Globalization.CultureInfo("en-US",false)
                                    .TextInfo;
                    return txtinfo.ToTitleCase(sourceText);
                default:
                    // 无效选项，将返回下面的异常包装结果
                    break;
            }
        }
        // 转换器被用于错误的类型
        return new BindingNotification(new InvalidCastException(), 
                                                BindingErrorType.Error);
    }

    public object ConvertBack(object? value, Type targetType, 
                                object? parameter, CultureInfo culture)
    {
      throw new NotSupportedException();
    }
}
```

## 目标属性类型

有时你可能希望编写一个自定义转换器，它能根据目标属性的要求切换输出类型。之所以能做到这一点，是因为 `Convert` 方法会接收到一个 `targetType` 参数，你可以结合 `IsAssignableTo` 来判断目标类型。

在下面这个例子中，`animalConverter` 可以针对绑定的 `Animal` 类对象，返回一张图片，或者返回一个文本名称：  

```xml title='XAML'
<Image Width="42" 
       Source="{Binding Animal, Converter={StaticResource animalConverter}}"/>
<TextBlock 
       Text="{Binding Animal, Converter={StaticResource animalConverter}}" />
```

```csharp title='AnimalConverter.cs'
public class AnimalConverter : IValueConverter
{
    public static readonly AnimalConverter Instance = new();

    public object? Convert( object? value, Type targetType, 
                                    object? parameter, CultureInfo culture )
    {
        if (value is Animal animal)
        {
            if (targetType.IsAssignableTo(typeof(IImage)))
            {
                img = @"icons/generic-animal-placeholder.png"
                switch (animal)
                {
                    case Dog d:
                      img = d.IsGoodBoy ? @"icons/dog-happy.png" 
                                                      : @"icons/dog.png";
                      break;
                    case Cat:
                      img = @"icons/cat.png";
                      break;
                    // 其他动物类型
                }
                // see https://docs.avaloniaui.net/docs/guides/data-binding/how-to-create-a-custom-data-binding-converter
                return BitmapAssetValueConverter.Instance
                    .Convert(img, typeof(Bitmap), parameter, culture);
            }
            else if (targetType.IsAssignableTo(typeof(string)))
            {
                return !string.IsNullOrEmpty(animal.NickName) ? 
                    $"{animal.Name} \"{animal.NickName}\"" : animal.Name;
            }
        }
        // 转换器被用于错误的类型
        return new BindingNotification(new InvalidCastException(), 
                                                    BindingErrorType.Error);
        
    }

    public object ConvertBack( object? value, Type targetType, 
                                    object? parameter, CultureInfo culture )
    {
      throw new NotSupportedException();
    }
}
```

## FuncValueConverter 与 FuncMultiConverter

你也可以使用 `FuncValueConverter`。它有两个或三个泛型参数：

* **TIn**：定义期望的输入类型。如果你想在 `MultiBinding` 中使用该转换器，它也可以是数组类型。

* **TParam**（可选）：定义传入 `Binding.ConverterParameter` 的类型。

* **TOut**：定义期望的输出类型。


### 单向示例

```csharp
public static class MyConverters
{
    public static FuncValueConverter<decimal?, string> MyConverter { get; } =
        new FuncValueConverter<decimal?, string>(num => $"Your number is: '{num}'");

    public static FuncMultiValueConverter<decimal?, string> MyMultiConverter { get; } =
        new FuncMultiValueConverter<decimal?, string>(num => $"Your numbers are: '{string.Join(", ", num)}'");
}
```

### 双向示例

如果要支持双向绑定，可以额外传入一个 `convertBack` 函数：

```csharp
public static class MyConverters
{
    public static FuncValueConverter<double, string> TemperatureConverter { get; } =
        new(
            celsius => $"{celsius:F1} °C",
            text => double.TryParse(text?.Replace(" °C", ""), out var c) ? c : 0
        );
}
```

```xml
<StackPanel>
    <!-- 输入 -->
    <NumericUpDown x:Name="Num1" Value="3" />
    <NumericUpDown x:Name="Num2" Value="3" />
    <!-- 输出 -->
    <TextBlock Text="{Binding #Num1.Value, Converter={x:Static my:MyConverters.MyConverter}}" />
    <TextBlock>
        <TextBlock.Text>
            <MultiBinding Converter="{x:Static my:MyConverters.MyMultiConverter}">
                <Binding Path="#Num1.Value" />
                <Binding Path="#Num2.Value" />
            </MultiBinding>
        </TextBlock.Text>
    </TextBlock>
</StackPanel>
```

## 更多信息

:::info
有关如何绑定图片的更多说明，请参阅 [How To Bind Image Files](/docs/data-binding/how-to-bind-image-files)。
:::
