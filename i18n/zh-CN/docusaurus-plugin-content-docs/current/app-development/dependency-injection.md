---
id: dependency-injection
title: 实现依赖注入
description: 在 Avalonia 应用中使用 Microsoft.Extensions.DependencyInjection 配置依赖注入。
doc-type: tutorial
---

[Dependency injection (DI)](https://en.wikipedia.org/wiki/Dependency_injection) 能帮助开发者编写更整洁、更模块化、也更易测试的代码。它的核心思路是：将独立的服务抽离出来，并在需要时进行创建和传递。

本指南会一步一步演示如何在 Avalonia 和 MVVM 模式中使用 DI。

## 前提条件

- 一个 Avalonia 项目（可以是通过模板创建的新项目，也可以是已有应用）
- .NET 8.0 SDK 或更高版本

## 步骤 0：场景与初始代码

假设你的应用中有一个 `MainViewModel`、一个 `BusinessService` 和一个 `Repository`。其中 `MainViewModel` 依赖 `IBusinessService`，而 `BusinessService` 又依赖 `IRepository`。一个简单实现可能如下：

```csharp
public partial class MainViewModel
{
    private readonly IBusinessService _businessService;

    public MainViewModel(IBusinessService businessService)
    {
        _businessService = businessService;
    }
}
```

```csharp
public class BusinessService : IBusinessService
{
    private readonly IRepository _repository;

    public BusinessService(IRepository repository)
    {
        _repository = repository;
    }
}
```

```csharp
public class Repository : IRepository
{
}
```

传统做法通常是直接实例化 `Repository`，再把它传给 `BusinessService`，然后再传给 `MainViewModel`，例如：

```csharp
var window = new MainWindow
{
    DataContext = new MainViewModel(new BusinessService(new Repository()))
}
```

对于不常使用、而且基本不会变化的简单构造函数，这种写法是可行的。但它的扩展性并不好，原因包括：
- 构造函数依赖越多，你就越需要手动实例化并传入更多对象。如果在本地直接创建依赖（例如 `new MainViewModel(new MyService())`），就会导致代码与某个具体依赖实例形成刚性耦合。
- 同样地，如果 `MainViewModel` 自己在构造函数中创建依赖，那它也会和依赖的创建方式直接耦合，从而带来很多相同的问题。
- 此外，如果 `MainViewModel` 在很多地方都会被实例化，那么一旦它的依赖发生变化（例如增加新依赖，或替换某个依赖实现），所有这些实例化位置都需要一起修改。

依赖注入通过抽象对象及其依赖的创建过程来解决这些问题。这样一来，服务可以被良好封装，并自动注入到所有声明需要它们的其他服务中。

## 步骤 1：安装 DI 的 NuGet 包
可用的 DI 容器有很多（例如 [DryIoC](https://github.com/dadhi/DryIoc)、[Autofac](https://github.com/autofac/Autofac)、[Pure.DI](https://github.com/DevTeam/Pure.DI)），但本指南聚焦于 `Microsoft.Extensions.DependencyInjection`。它是一个轻量、可扩展的依赖注入容器，并提供了一种基于约定的方式为 .NET 应用（包括 Avalonia 桌面应用）添加 DI。

在项目目录的终端中运行以下命令来安装该 DI 包：

```shell
dotnet add package Microsoft.Extensions.DependencyInjection
```

## 步骤 2：添加 ServiceCollectionExtensions
下面的代码为 `IServiceCollection` 创建了一个扩展方法。这个方法会把服务注册到服务集合中，使它们可以被注入使用。

```csharp
public static class ServiceCollectionExtensions
{
    public static void AddCommonServices(this IServiceCollection collection)
    {
        collection.AddSingleton<IRepository, Repository>();
        collection.AddTransient<BusinessService>();
        collection.AddTransient<MainViewModel>();
    }
}
```

## 步骤 3：修改 App.axaml.cs
接下来，修改 `App.axaml.cs` 类以使用 DI 容器。这样一来，前一步注册的视图模型就可以通过依赖注入容器来解析，然后把完整构建好的视图模型设置为 `MainWindow` 或 `MainView` 的数据上下文。

```csharp
public class App : Application
{
    public override void Initialize()
    {
        AvaloniaXamlLoader.Load(this);
    }

    public override void OnFrameworkInitializationCompleted()
    {
        // If you use CommunityToolkit, line below is needed to remove Avalonia data validation.
        // Without this line you will get duplicate validations from both Avalonia and CT
        BindingPlugins.DataValidators.RemoveAt(0);

        // Register all the services needed for the application to run
        var collection = new ServiceCollection();
        collection.AddCommonServices();

        // Creates a ServiceProvider containing services from the provided IServiceCollection
        var services = collection.BuildServiceProvider();

        var vm = services.GetRequiredService<MainViewModel>();
        if (ApplicationLifetime is IClassicDesktopStyleApplicationLifetime desktop)
        {
            desktop.MainWindow = new MainWindow
            {
                DataContext = vm
            };
        }
        else if (ApplicationLifetime is ISingleViewApplicationLifetime singleViewPlatform)
        {
            singleViewPlatform.MainView = new MainView
            {
                DataContext = vm
            };
        }

        base.OnFrameworkInitializationCompleted();
    }
}
```

## 验证结果

运行应用。如果 DI 容器配置正确，`MainWindow`（或 `MainView`）会正常显示，并且其 `DataContext` 会被设置为一个已经完成所有依赖解析的 `MainViewModel` 实例。

## 另请参阅

- [Data Binding](/docs/data-binding/introduction-to-data-binding)：将视图模型绑定到视图。
- [MVVM Architecture](/docs/fundamentals/the-mvvm-pattern)：在 Avalonia 中使用 MVVM 模式。
