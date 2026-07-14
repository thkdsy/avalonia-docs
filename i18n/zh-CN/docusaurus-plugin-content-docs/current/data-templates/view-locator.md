---
id: view-locator
title: 视图定位器
description: 使用实现了 IDataTemplate 的 ViewLocator 为视图模型自动解析对应视图。
doc-type: explanation
---

在 MVVM 应用中，视图模型负责承载应用逻辑，但并不了解 UI。*视图定位器* 正是用来弥合这一差距的：它会为给定的视图模型自动解析出正确的视图。当 Avalonia 在某个内容区域中遇到视图模型对象时，视图定位器就会决定应创建并显示哪个视图。

:::info
ViewLocator 不是必需的。你也可以通过在 XAML 中定义 [DataTemplates](/docs/data-templates/data-template-collection) 来实现同样的效果。Avalonia 默认项目模板中包含 ViewLocator，只是为了让 MVVM 应用开箱即用更方便。
:::

## 工作原理

ViewLocator 实现了 [`IDataTemplate`](/api/avalonia/controls/templates/idatatemplate) 接口，这意味着它会参与 Avalonia 的标准数据模板解析流程。当 [`ContentControl`](/api/avalonia/controls/contentcontrol) 或类似的呈现控件需要显示一个不是控件本身的对象时，它会查找匹配的 `IDataTemplate`。已注册的 ViewLocator 会匹配视图模型对象，并构建对应的视图。

`IDataTemplate` 接口包含两个成员：

- `Match(object data)`：如果此模板可以处理给定对象，则返回 `true`。
- `Build(object data)`：创建并返回要显示的控件。

## 默认实现

Avalonia 项目模板中默认包含的 ViewLocator 使用一套命名约定：它会把完整限定类型名中的 `"ViewModel"` 替换为 `"View"`，然后通过反射解析出对应的视图类型。

例如，`MyApp.ViewModels.MainViewModel` 会解析成 `MyApp.Views.MainView`。

```csharp
public class ViewLocator : IDataTemplate
{
    public Control Build(object data)
    {
        var name = data.GetType().FullName!.Replace("ViewModel", "View");
        var type = Type.GetType(name);

        if (type != null)
        {
            return (Control)Activator.CreateInstance(type)!;
        }

        return new TextBlock { Text = "Not Found: " + name };
    }

    public bool Match(object data)
    {
        return data is ViewModelBase;
    }
}
```

`Match` 会对所有继承自 `ViewModelBase` 的对象返回 `true`。`Build` 使用 `Activator.CreateInstance` 来构造视图；如果找不到对应视图类型，则返回一个用于报错的 `TextBlock`。

:::tip
基于反射的方式很适合快速上手，但它与 Native AOT 不兼容，也不具备编译期安全性。对于生产应用，建议考虑下面介绍的替代方案。
:::

## 注册视图定位器

请在 `App.axaml` 中注册你的 ViewLocator，这样它就可以在整个应用中使用：

```xml
<Application xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Class="MyApp.App"
             xmlns:local="using:MyApp"
             RequestedThemeVariant="Default">
    <Application.DataTemplates>
        <local:ViewLocator />
    </Application.DataTemplates>

    <Application.Styles>
        <FluentTheme />
    </Application.Styles>
</Application>
```

由于 `ViewLocator` 实现了 `IDataTemplate`，因此它会和你定义的其他数据模板一起放在 `DataTemplates` 集合中。

## 使用视图定位器

注册完成后，只要某个位置将视图模型作为内容显示，视图定位器就会自动解析对应视图。最常见的模式，是让 `ContentControl` 绑定到某个视图模型属性：

```xml
<ContentControl Content="{Binding CurrentPage}" />
```

当 `CurrentPage` 被设置为 `SettingsViewModel` 的实例时，视图定位器会创建一个 `SettingsView`，并将其显示在 `ContentControl` 中。之后如果把 `CurrentPage` 切换成其他视图模型，显示的视图也会自动切换。

你也可以直接把视图模型设置为窗口的 `DataContext`：

```csharp
DataContext = new MainViewModel(); // ViewLocator 会解析出 MainView
```

## 替代方案

默认基于反射的 ViewLocator 很适合原型开发，但它也有一些限制：无法在编译期验证视图是否存在、不支持 AOT，也无法向视图注入依赖。下面这些替代方案可以解决这些问题。

### 模式匹配

可以使用 C# 模式匹配，通过显式的类型映射来替代反射。这种方式兼容 AOT，并具备编译期安全性：

```csharp
public class ViewLocator : IDataTemplate
{
    public Control Build(object data)
    {
        return data switch
        {
            MainViewModel => new MainView(),
            SettingsViewModel => new SettingsView(),
            ProfileViewModel => new ProfileView(),
            _ => new TextBlock { Text = $"No view for {data.GetType().Name}" }
        };
    }

    public bool Match(object data) => data is ViewModelBase;
}
```

你需要为每个视图模型在 `switch` 表达式中添加一行映射。这是一种有意的权衡：用少量的手工维护，换取编译期检查和 AOT 兼容性。

### XAML 数据模板

你也可以完全跳过 ViewLocator，直接在 XAML 中把视图与视图模型的映射定义为标准数据模板：

```xml
<Application.DataTemplates>
    <DataTemplate DataType="{x:Type vm:MainViewModel}">
        <views:MainView />
    </DataTemplate>
    <DataTemplate DataType="{x:Type vm:SettingsViewModel}">
        <views:SettingsView />
    </DataTemplate>
</Application.DataTemplates>
```

这种方式完全使用 Avalonia 内置的模板解析机制，不需要编写自定义代码。关于模板匹配与搜索顺序的详细说明，请参阅 [Data Template Collection](/docs/data-templates/data-template-collection)。

### 依赖注入

当视图需要通过构造函数注入服务时，可以将模式匹配与 DI 容器结合使用：

```csharp
public class ViewLocator : IDataTemplate
{
    private readonly IServiceProvider _services;

    public ViewLocator(IServiceProvider services)
    {
        _services = services;
    }

    public Control Build(object data)
    {
        return data switch
        {
            MainViewModel => _services.GetRequiredService<MainView>(),
            SettingsViewModel => _services.GetRequiredService<SettingsView>(),
            _ => new TextBlock { Text = $"No view for {data.GetType().Name}" }
        };
    }

    public bool Match(object data) => data is ViewModelBase;
}
```

由于这个 ViewLocator 需要构造函数参数，因此应在代码中注册，而不是在 XAML 中注册：

```csharp
public override void OnFrameworkInitializationCompleted()
{
    var services = BuildServiceProvider();
    DataTemplates.Add(new ViewLocator(services));

    base.OnFrameworkInitializationCompleted();
}
```

有关如何设置服务提供器，请参阅 [Dependency Injection](/docs/app-development/dependency-injection)。

### 源生成器

对于大型应用，如果手动维护映射关系并不现实，那么可以使用源生成器在编译期生成 ViewLocator 代码。这样既没有运行时开销，也能完全兼容 AOT。

提供此能力的社区包包括：

- **[StaticViewLocator](https://github.com/wieslawsoltes/StaticViewLocator)**：一个可自动发现并注册视图/视图模型配对关系的 NuGet 包。

如果你想自己编写源生成器，可以参阅 [Microsoft 的源生成器文档](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview)。

## 选择哪种方案

| 方案 | 兼容 AOT | 编译期安全 | 支持 DI | 维护成本 |
|---|---|---|---|---|
| 反射（默认） | 否 | 否 | 否 | 无 |
| 模式匹配 | 是 | 是 | 可选 | 每个视图模型加一行 |
| XAML 数据模板 | 是 | 是 | 否 | 每个视图模型加一个模板 |
| DI + 模式匹配 | 是 | 是 | 是 | 每个视图模型加一行 |
| 源生成器 | 是 | 是 | 视实现而定 | 无（自动生成） |

## 另请参阅

- [数据模板简介](/docs/data-templates/introduction-to-data-templates)：Avalonia 如何选择和应用数据模板。
- [数据模板集合](/docs/data-templates/data-template-collection)：按类型定义多个模板。
- [在代码中创建数据模板](/docs/data-templates/creating-data-templates-in-code)：实现 `IDataTemplate` 并使用 `FuncDataTemplate<T>`。
- [依赖注入](/docs/app-development/dependency-injection)：为应用配置服务注册。
