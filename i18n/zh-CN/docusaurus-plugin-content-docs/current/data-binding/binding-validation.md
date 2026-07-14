---
id: binding-validation
title: 数据绑定中的验证
description: 使用 DataAnnotations、INotifyDataErrorInfo 或基于异常的方式验证绑定数据。
doc-type: how-to
---

Avalonia 支持通过标准的 .NET 验证机制进行数据验证。当绑定属性未通过验证时，控件会显示错误指示以及对应的验证消息。

## 使用数据注解进行验证

最简单的方式，是在视图模型属性上使用 `System.ComponentModel.DataAnnotations` 特性。这种方式可以与 CommunityToolkit.Mvvm 的 `ObservableValidator` 基类配合使用：

```csharp
using System.ComponentModel.DataAnnotations;
using CommunityToolkit.Mvvm.ComponentModel;

public partial class RegistrationViewModel : ObservableValidator
{
    [ObservableProperty]
    [NotifyDataErrorInfo]
    [Required(ErrorMessage = "Name is required")]
    [MinLength(2, ErrorMessage = "Name must be at least 2 characters")]
    private string _name = "";

    [ObservableProperty]
    [NotifyDataErrorInfo]
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid email address")]
    private string _email = "";

    [ObservableProperty]
    [NotifyDataErrorInfo]
    [Range(18, 120, ErrorMessage = "Age must be between 18 and 120")]
    private int _age;
}
```

`[NotifyDataErrorInfo]` 特性会让 CommunityToolkit.Mvvm 在属性变化时执行验证，并触发相应的 `INotifyDataErrorInfo` 事件。

使用 `TwoWay` 模式绑定这些属性：

```xml
<StackPanel Spacing="8">
    <TextBox Text="{Binding Name}" PlaceholderText="Name" />
    <TextBox Text="{Binding Email}" PlaceholderText="Email" />
    <NumericUpDown Value="{Binding Age}" Watermark="Age" />
</StackPanel>
```

当验证失败时，控件默认会显示红色边框，并通过工具提示显示错误消息。

## INotifyDataErrorInfo

`INotifyDataErrorInfo` 是 .NET 中用于属性级验证的标准接口。Avalonia 会自动识别任何实现了该接口的视图模型所产生的验证错误：

```csharp
public class LoginViewModel : INotifyPropertyChanged, INotifyDataErrorInfo
{
    private string _username = "";
    private readonly Dictionary<string, List<string>> _errors = new();

    public string Username
    {
        get => _username;
        set
        {
            _username = value;
            ValidateUsername();
            OnPropertyChanged();
        }
    }

    private void ValidateUsername()
    {
        ClearErrors(nameof(Username));

        if (string.IsNullOrWhiteSpace(Username))
            AddError(nameof(Username), "Username is required");
        else if (Username.Length < 3)
            AddError(nameof(Username), "Username must be at least 3 characters");
    }

    // INotifyDataErrorInfo 的实现
    public bool HasErrors => _errors.Count > 0;

    public event EventHandler<DataErrorsChangedEventArgs>? ErrorsChanged;

    public IEnumerable GetErrors(string? propertyName)
    {
        if (propertyName is not null && _errors.TryGetValue(propertyName, out var errors))
            return errors;
        return Array.Empty<string>();
    }

    private void AddError(string propertyName, string error)
    {
        if (!_errors.ContainsKey(propertyName))
            _errors[propertyName] = new List<string>();

        _errors[propertyName].Add(error);
        ErrorsChanged?.Invoke(this, new DataErrorsChangedEventArgs(propertyName));
    }

    private void ClearErrors(string propertyName)
    {
        if (_errors.Remove(propertyName))
            ErrorsChanged?.Invoke(this, new DataErrorsChangedEventArgs(propertyName));
    }

    // INotifyPropertyChanged ...
    public event PropertyChangedEventHandler? PropertyChanged;
    protected void OnPropertyChanged([CallerMemberName] string? name = null)
        => PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
}
```

## 自定义验证特性

你可以创建自定义验证特性，以复用验证逻辑：

```csharp
public class NotEqualToAttribute : ValidationAttribute
{
    private readonly string _otherProperty;

    public NotEqualToAttribute(string otherProperty)
    {
        _otherProperty = otherProperty;
    }

    protected override ValidationResult? IsValid(
        object? value, ValidationContext context)
    {
        var otherValue = context.ObjectType
            .GetProperty(_otherProperty)?
            .GetValue(context.ObjectInstance);

        if (Equals(value, otherValue))
            return new ValidationResult(
                ErrorMessage ?? $"Must not equal {_otherProperty}");

        return ValidationResult.Success;
    }
}
```

然后把它用在你的视图模型上：

```csharp
[ObservableProperty]
[NotifyDataErrorInfo]
[Required]
private string _password = "";

[ObservableProperty]
[NotifyDataErrorInfo]
[Required]
[NotEqualTo(nameof(Password), ErrorMessage = "New password must differ from current")]
private string _newPassword = "";
```

## 验证错误显示

### 默认行为

默认情况下，Avalonia 会通过以下方式显示验证错误：
- 控件周围出现红色边框
- 鼠标悬停时显示包含错误消息的工具提示
- 在控件角落显示红色装饰标记

验证错误也会通过 [`DataValidationErrors`](/api/avalonia/controls/datavalidationerrors) 的自动化对象，自动暴露给屏幕阅读器和其他辅助技术。详情请参阅 [Accessibility](/docs/app-development/accessibility#data-validation-errors)。

### 自定义错误显示

你可以使用 `DataValidationErrors` 控件来自定义错误的显示方式。通常做法是通过控件主题重写 `DataValidationErrors` 的模板：

```xml
<Style Selector="DataValidationErrors">
    <Setter Property="Template">
        <ControlTemplate>
            <DockPanel>
                <ContentControl DockPanel.Dock="Top"
                                ContentTemplate="{TemplateBinding ErrorTemplate}"
                                DataContext="{TemplateBinding Owner}"
                                Content="{Binding (DataValidationErrors.Errors)}"
                                IsVisible="{Binding (DataValidationErrors.HasErrors)}" />
                <ContentPresenter Name="PART_ContentPresenter"
                                  Background="{TemplateBinding Background}"
                                  BorderBrush="{TemplateBinding BorderBrush}"
                                  BorderThickness="{TemplateBinding BorderThickness}"
                                  Padding="{TemplateBinding Padding}"
                                  Content="{TemplateBinding Content}" />
            </DockPanel>
        </ControlTemplate>
    </Setter>
    <Setter Property="ErrorTemplate">
        <DataTemplate>
            <ItemsControl ItemsSource="{Binding}" Margin="0,0,0,4">
                <ItemsControl.ItemTemplate>
                    <DataTemplate>
                        <TextBlock Text="{Binding ErrorContent}"
                                   Foreground="Red" FontSize="12" />
                    </DataTemplate>
                </ItemsControl.ItemTemplate>
            </ItemsControl>
        </DataTemplate>
    </Setter>
</Style>
```

### 在控件下方显示错误

一种常见做法是把错误消息显示在输入框下方，而不是使用工具提示：

```xml
<Style Selector="DataValidationErrors">
    <Setter Property="ErrorTemplate">
        <DataTemplate>
            <TextBlock Foreground="#EF4444" FontSize="12" Margin="0,2,0,0">
                <TextBlock.Text>
                    <MultiBinding StringFormat="{}{0}">
                        <Binding Path="[0].ErrorContent" />
                    </MultiBinding>
                </TextBlock.Text>
            </TextBlock>
        </DataTemplate>
    </Setter>
</Style>
```

## Validating on submit

To validate the entire form when the user clicks a submit button:

```csharp
public partial class RegistrationViewModel : ObservableValidator
{
    [RelayCommand(CanExecute = nameof(CanSubmit))]
    private void Submit()
    {
        ValidateAllProperties();

        if (HasErrors)
            return;

        // Proceed with submission
    }

    private bool CanSubmit() => !HasErrors;
}
```

`ValidateAllProperties()` runs all validation attributes on all properties at once, which is useful for catching errors on fields the user has not yet interacted with.

## Exception-based validation

Avalonia also catches exceptions thrown during binding updates and displays them as validation errors. This can be useful for simple type conversion validation:

```csharp
private int _quantity;
public int Quantity
{
    get => _quantity;
    set
    {
        if (value < 0)
            throw new ArgumentException("Quantity cannot be negative");
        _quantity = value;
        OnPropertyChanged();
    }
}
```

While this works, `INotifyDataErrorInfo` is the preferred approach because it supports multiple errors per property and async validation.

## 另请参阅

- [Data Binding Syntax](/docs/data-binding/data-binding-syntax): Binding modes and parameters.
- [INotifyPropertyChanged](/docs/data-binding/inotifypropertychanged): Change notification for view models.
- [The MVVM Pattern](/docs/fundamentals/the-mvvm-pattern): View model patterns and CommunityToolkit.Mvvm.
