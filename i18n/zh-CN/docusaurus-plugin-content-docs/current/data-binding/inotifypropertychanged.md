---
id: inotifypropertychanged
title: 如何使用 INotifyPropertyChanged
description: 实现 INotifyPropertyChanged，以便在视图模型属性变化时通知 UI。
doc-type: how-to
---

## 简介
`INotifyPropertyChanged` 接口是 Model-View-ViewModel（MVVM）设计模式中的一个关键组成部分，它有助于构建可扩展且易于维护的应用程序。通过在属性变化时发出通知，它能够让 View 自动更新，从而改善应用程序各组件之间的协作。

## 什么是 INotifyPropertyChanged？

`INotifyPropertyChanged` 是 .NET 提供的一个接口，类可以通过实现它来表示某个属性值已经发生变化。这在数据绑定场景中特别有用，因为一旦被绑定的数据发生变化，UI 就可以自动更新。

`INotifyPropertyChanged` 接口只包含一个事件成员：`PropertyChanged`。当某个属性值发生变化时，对象会触发 `PropertyChanged` 事件，通知所有绑定到它的元素该属性已发生变化。

## 为什么 INotifyPropertyChanged 在 MVVM 中很重要？
在 MVVM 模式中，ViewModel 封装了 View 的交互逻辑，同时也封装了来自 Model 的数据。View 会绑定到 ViewModel 中的属性，而 ViewModel 再向外暴露 Model 对象中包含的数据。

为了让 MVVM 模式按预期工作，底层数据一旦变化，View 就需要随之更新。这正是 `INotifyPropertyChanged` 发挥作用的地方。通过在 ViewModel 中实现这个接口，你可以把 Model 中的数据变化通知给 View，从而自动更新 UI。

## 实现 INotifyPropertyChanged
下面是一个实现 `INotifyPropertyChanged` 的示例：

```csharp
public class MyViewModel : INotifyPropertyChanged
{
    private string _name;

    public string Name
    {
        get { return _name; }
        set
        {
            _name = value;
            OnPropertyChanged(nameof(Name));
        }
    }

    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged(string propertyName)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

在这段代码中，每当 `Name` 属性被设置为新值时，就会调用 `OnPropertyChanged` 方法，从而触发 `PropertyChanged` 事件。所有绑定到该属性的 UI 元素随后都会更新，以反映新的值。

## 使用 MVVM Toolkit 简化 INotifyPropertyChanged
虽然实现 `INotifyPropertyChanged` 本身并不复杂，但当你的 ViewModel 中有很多属性时，这项工作就会变得相当繁琐。幸运的是，.NET Community Toolkit 的 MVVM 库提供了更高效的实现方式：借助 `ObservableObject` 类、`[ObservableProperty]` 特性以及 Source Generator 自动生成代码。

下面展示了如何使用 `ObservableObject` 达到与前面相同的效果：

```csharp
using CommunityToolkit.Mvvm.ComponentModel;

public partial class MyViewModel : ObservableObject
{
    [ObservableProperty]
    private string _name;
}
```

在这段代码中，`ObservableObject` 类实现了 `INotifyPropertyChanged`，而 `[ObservableProperty]` 特性用于表明 `_name` 是一个可观察属性。随后，Source Generator 会在幕后生成所需的样板代码，包括属性的 getter 和 setter，并在属性变化时自动调用 `OnPropertyChanged` 方法。这样既能让实现更简洁，也能减少出错概率。

MVVM Toolkit 提供了一整套工具，用来简化你在 .NET 应用程序中实现 MVVM 模式的过程，其中也包括简化 `INotifyPropertyChanged` 的使用。借助 Source Generator，你的代码在保持相同功能的前提下，会更加高效且更易阅读。

## 另请参阅

- [The MVVM Pattern](/docs/fundamentals/the-mvvm-pattern): Introduction to the MVVM architectural pattern.
- [Data Validation](/docs/data-binding/binding-validation): Validation in data binding with INotifyDataErrorInfo.
- [Introduction to Data Binding](/docs/data-binding/introduction-to-data-binding): Data binding overview.






