---
id: data-validation
title: 数据验证
description: 在 Avalonia 中使用 DataAnnotationsValidationPlugin 验证用户输入。
doc-type: overview
---

本页说明如何在 Avalonia 中验证用户输入的数据。

## 数据注解验证插件

Avalonia 的数据注解验证支持允许你验证与 `ViewModel` 属性关联的任意 [`Validation-Attributes`](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationattribute)。

你可以验证内置验证特性、[`CustomValidationAttribute`](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.customvalidationattribute)，以及你自己从 `ValidationAttribute` 派生出的特性。

### Enable `DataAnnotationsValidationPlugin`

从 Avalonia v12 开始，数据注解验证插件默认是禁用的。要启用这个插件：

1. 打开你项目中的 `Program.cs` 文件。
2. 在 `AppBuilder` 中添加这一行：

```csharp
.WithDataAnnotationsValidation()
```

如果你正在迁移旧项目，并且之前曾通过 `BindingPlugins.DataValidators.Remove(plugin);` 或类似方式手动移除了数据注解验证插件，那么现在可以删除这段代码了，因为它已经不再需要。

### 示例：`EMail` 属性为必填项，且必须是有效的电子邮件地址

:::note
`RaiseAndSetIfChanged` 是 ReactiveUI 提供的方法。本示例依赖 ReactiveUI 才能工作。
:::

```csharp
[Required]
[EmailAddress]
public string? EMail
{
    get { return _EMail; }
    set { this.RaiseAndSetIfChanged(ref _EMail, value); }
}
```

## 自定义验证消息的外观

[`DataValidationErrors class`](/api/avalonia/controls/datavalidationerrors) 用于显示验证错误消息。它可以作为控件放置在任何支持数据验证的控件的 `ControlTemplate` 中，例如 `TextBox`。

你可以在 `.axaml` 文件中设置一个 `Style`，来自定义错误消息的外观。

### 示例：自定义数据验证错误消息

```xml
<Style Selector="DataValidationErrors">
  <Setter Property="Template">
    <ControlTemplate>
      <DockPanel LastChildFill="True">
        <ContentControl DockPanel.Dock="Right"
                        ContentTemplate="{TemplateBinding ErrorTemplate}"
                        DataContext="{TemplateBinding Owner}"
                        Content="{Binding (DataValidationErrors.Errors)}"
                        IsVisible="{Binding (DataValidationErrors.HasErrors)}"/>
        <ContentPresenter Name="PART_ContentPresenter"
                          Background="{TemplateBinding Background}"
                          BorderBrush="{TemplateBinding BorderBrush}"
                          BorderThickness="{TemplateBinding BorderThickness}"
                          CornerRadius="{TemplateBinding CornerRadius}"
                          ContentTemplate="{TemplateBinding ContentTemplate}"
                          Content="{TemplateBinding Content}"
                          Padding="{TemplateBinding Padding}"/>
      </DockPanel>
    </ControlTemplate>
  </Setter>
  <Setter Property="ErrorTemplate">
    <DataTemplate x:DataType="{x:Type x:Object}">
      <Canvas Width="14" Height="14" Margin="4 0 1 0" 
              Background="Transparent">
        <Canvas.Styles>
          <Style Selector="ToolTip">
            <Setter Property="Background" Value="LightRed"/>
            <Setter Property="BorderBrush" Value="Red"/>
          </Style>
        </Canvas.Styles>
        <ToolTip.Tip>
          <ItemsControl ItemsSource="{Binding}"/>
        </ToolTip.Tip>
        <Path Data="M14,7 A7,7 0 0,0 0,7 M0,7 A7,7 0 1,0 14,7 M7,3l0,5 M7,9l0,2" 
              Stroke="Red" 
              StrokeThickness="2"/>
      </Canvas>
    </DataTemplate>
  </Setter>
</Style>
```

## 另请参阅

- [Data Binding](/docs/data-binding/introduction-to-data-binding)：将数据绑定到控件。
- [Community Toolkit MVVM](https://learn.microsoft.com/en-us/windows/communitytoolkit/mvvm/observablevalidator)：使用 `ObservableValidator` 实现验证。
