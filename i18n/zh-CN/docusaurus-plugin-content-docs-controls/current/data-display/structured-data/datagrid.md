---
id: datagrid
title: DataGrid
---

import Pill from '/src/components/global/Pill';
import DataGridNuGetScreenshot from '/img/controls/datagrid/datagrid-nuget.png';
import DataGridSortColumnScreenshot from '/img/controls/datagrid/datagrid-sort-column.gif';
import DataGridReorderColumnScreenshot from '/img/controls/datagrid/datagrid-reorder-column.gif';
import DataGridColumnTypesScreenshot from '/img/controls/datagrid/datagrid-column-types.gif';
import DataGridTemplateColumn from '/img/controls/datagrid/grid4.gif';
import DataGridColumnPreviewScreenshot from '/img/controls/datagrid/datagridtextcolumn.png';

<Pill variant="warning">Deprecated</Pill>

`DataGrid` 会在可自定义的网格中显示重复数据。该控件支持样式、模板和绑定。

`DataGrid` 需要绑定到视图模型中的可观察集合，该视图模型应位于相关的 **data context** 中。

:::info
如需回顾 **data context** 的概念，请参阅 [Data context](/docs/data-binding/data-context)。
:::

:::info
`DataGrid` 位于一个额外的 _Avalonia_ 包中。若要在项目中使用 `DataGrid`，你必须引用 **Avalonia.Controls.DataGrid** _NuGet_ 包，并同时引用它所使用的样式，见下文。
:::

### NuGet 包引用

你必须安装 `DataGrid` 对应的 _NuGet_ 包。可以通过以下任意方式完成安装。你也可以在 IDE 的项目菜单中使用 **Manage NuGet Packages**：

<Image light={DataGridNuGetScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

或者，你也可以在命令行中运行以下命令：

```bash
dotnet add package Avalonia.Controls.DataGrid
```

或者，直接在项目（`.csproj`）文件中添加包引用：

```xml
<PackageReference Include="Avalonia.Controls.DataGrid" Version="11.0.0" />
```

:::caution
请注意，你必须始终安装与当前使用的 _Avalonia_ 版本相匹配的 DataGrid 版本。
:::

### 引入 DataGrid 样式

你必须引用 `DataGrid` 的主题，以便包含它所依赖的额外样式。可以通过在应用程序（`App.axaml` 文件）中添加 `<StyleInclude>` 元素来实现。

例如（当你使用 `FluentTheme` 时）：

```xml
<Application.Styles>
    <FluentTheme />
    <StyleInclude Source="avares://Avalonia.Controls.DataGrid/Themes/Fluent.xaml"/>
</Application.Styles>
```

:::caution
DataGrid 的样式必须与你使用的整体主题保持匹配，否则会出现冲突和未解析资源。对于第三方主题，请查阅其文档和示例。
:::


### 常用属性

你最常使用的通常是以下属性：

| 属性 | 说明 |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `AutoGenerateColumns` | 是否根据绑定数据源属性名自动生成列。（默认值为 false。） |
| `ItemsSource` | 用作控件数据源的绑定集合。 |
| `IsReadOnly` | 为 true 时将绑定方向设为单向。默认值为 false——网格会接受对绑定数据的修改。 |
| `CanUserReorderColumns` | 指示用户是否可以通过拖动列表头来更改列显示顺序。（默认值为 false。） |
| `CanUserResizeColumns` | 指示用户是否可以通过指针调整列宽。（默认值为 false。） |
| `CanUserSortColumns` | 指示用户是否可以通过点击列表头来对列进行排序。（默认值为 true。） |

### 示例

这个示例会生成一个基础 `DataGrid`，其中列表头名称会根据项类自动生成。其数据源绑定到主窗口视图模型。

```xml
<DataGrid Margin="20" ItemsSource="{Binding People}" 
          AutoGenerateColumns="True" IsReadOnly="True" 
          GridLinesVisibility="All"
          BorderThickness="1" BorderBrush="Gray">
</DataGrid>
```

```csharp title='C# View Model'
using AvaloniaControls.Models;
using System.Collections.Generic;
using System.Collections.ObjectModel;

namespace AvaloniaControls.ViewModels
{
    public class MainWindowViewModel : ViewModelBase
    {
        public ObservableCollection<Person> People { get; }

        public MainWindowViewModel()
        {
            var people = new List<Person> 
            {
                new Person("Neil", "Armstrong"),
                new Person("Buzz", "Lightyear"),
                new Person("James", "Kirk")
            };
            People = new ObservableCollection<Person>(people);
        }
    }
}
```

```csharp title='C# Item Class'
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    
    public Person(string firstName , string lastName)
    {
        FirstName = firstName;
        LastName = lastName;
    }
}
```

<Image light={DataGridSortColumnScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

:::info
这些示例使用 MVVM 模式，并通过数据绑定连接到 `ObservableCollection`。有关数据绑定概念的更多信息，请参阅 [数据绑定简介](/docs/data-binding/introduction-to-data-binding)。
:::

项类中的属性名通常并不适合作为列标题。该示例为网格添加了自定义列表头名称，同时启用了列重排和列宽调整，并禁用了默认的列排序功能：

```xml
<DataGrid Margin="20" ItemsSource="{Binding People}"
          IsReadOnly="True"
          CanUserReorderColumns="True"
          CanUserResizeColumns="True"
          CanUserSortColumns="False"
          GridLinesVisibility="All"
          BorderThickness="1" BorderBrush="Gray">
  <DataGrid.Columns>
     <DataGridTextColumn Header="First Name"  Binding="{Binding FirstName}"/>
     <DataGridTextColumn Header="Last Name" Binding="{Binding LastName}" />
  </DataGrid.Columns>
</DataGrid>
```

<Image light={DataGridReorderColumnScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

这个示例展示了 `DataGrid` 如何接受修改、更新底层集合，并通过不同列类型编辑数据：

```xml
<DataGrid Margin="20" ItemsSource="{Binding People}"        
          GridLinesVisibility="All"
          BorderThickness="1" BorderBrush="Gray">
  <DataGrid.Columns>
     <DataGridTextColumn Header="First Name"  Binding="{Binding FirstName}"/>
     <DataGridTextColumn Header="Last Name" Binding="{Binding LastName}" />
     <DataGridCheckBoxColumn Header="Fictitious?" Binding="{Binding IsFictitious}" />
  </DataGrid.Columns>
</DataGrid>
```

```csharp title='C# View Model'
using AvaloniaControls.Models;
using System.Collections.Generic;
using System.Collections.ObjectModel;

namespace AvaloniaControls.ViewModels
{
    public class MainWindowViewModel : ViewModelBase
    {
        public ObservableCollection<Person> People { get; }

        public MainWindowViewModel()
        {
            var people = new List<Person> 
            {
                new Person("Neil", "Armstrong", false),
                new Person("Buzz", "Lightyear", true),
                new Person("James", "Kirk", true)
            };
            People = new ObservableCollection<Person>(people);
        }
    }
}
```

```csharp title='C# Item Class'
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public bool IsFictitious { get; set; }

    public Person(string firstName , string lastName, bool isFictitious)
    {
        FirstName = firstName;
        LastName = lastName;
        IsFictitious = isFictitious;
    }
}
```

<Image light={DataGridColumnTypesScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## DataGridTemplateColumn

你可以使用这种列类型来自定义数据网格列的显示和编辑方式。

这里有两个通过附加属性定义的数据模板：

<table><thead><tr><th width="269">数据模板</th><th>说明</th></tr></thead><tbody><tr><td><code>CellTemplate</code> </td><td>列值在显示状态（非编辑状态）下的呈现方式。 </td></tr><tr><td><code>CellEditingTemplate</code> </td><td>列值在编辑状态下使用的模板。</td></tr></tbody></table>

:::info
如果你没有设置编辑模板，该列将保持只读。
:::

### 示例

这个示例会在编辑人员年龄属性时显示一个数字增减控件：



```xml
<Window ...
  xmlns:model="using:AvaloniaControls.Models">
  
  <DataGrid Margin="20" ItemsSource="{Binding People}"
          GridLinesVisibility="All"
          BorderThickness="1" BorderBrush="Gray">
    <DataGrid.Columns>
      <DataGridTextColumn Header="First Name" Width="2*"
         Binding="{Binding FirstName}" />
      <DataGridTextColumn Header="Last Name" Width="2*"
         Binding="{Binding LastName}" />
      
      <DataGridTemplateColumn Header="Age" SortMemberPath="AgeInYears">
        <DataGridTemplateColumn.CellTemplate>
          <DataTemplate DataType="model:Person">
            <TextBlock Text="{Binding AgeInYears, StringFormat='{}{0} years'}" 
              VerticalAlignment="Center" HorizontalAlignment="Center" />
          </DataTemplate>
        </DataGridTemplateColumn.CellTemplate>
        <DataGridTemplateColumn.CellEditingTemplate>
          <DataTemplate DataType="model:Person">
            <NumericUpDown Value="{Binding AgeInYears}"  
               FormatString="N0" Minimum="0" Maximum="120"  
               HorizontalAlignment="Stretch"/>
          </DataTemplate>
        </DataGridTemplateColumn.CellEditingTemplate>
      </DataGridTemplateColumn>
    
    </DataGrid.Columns>
  </DataGrid>
</Window>
```


```csharp title='C# View Model'
using AvaloniaControls.Models;
using System.Collections.Generic;
using System.Collections.ObjectModel;

namespace AvaloniaControls.ViewModels
{
    public class MainWindowViewModel : ViewModelBase
    {
        public ObservableCollection<Person> People { get; }

        public MainWindowViewModel()
        {
            var people = new List<Person> 
            {
                new Person("Neil", "Armstrong",  55),
                new Person("Buzz", "Lightyear", 38),
                new Person("James", "Kirk", 44)
            };
            People = new ObservableCollection<Person>(people);
        }
    }
}
```


```csharp title='C# Item Class'
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int AgeInYears { get; set; } 

    public Person(string firstName, string lastName, int ageInYears)
    {
        FirstName = firstName;
        LastName = lastName;
        AgeInYears = ageInYears;
    }
}
```

<Image light={DataGridTemplateColumn} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## DataGridColumn

一个 `DataGrid` 可以包含多个数据网格列，而 _Avalonia_ 内置了两种可用于显示不同数据类型的列类型，以及一种可自定义列外观的模板列类型。

| 列类型 | 说明 |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DataGridTextColumn` | 使用文本框来显示和编辑列数据。你可以在此列类型中控制字体族、字号等属性。 |
| `DataGridCheckBoxColumn` | 当数据为 Boolean 时，使用复选框来显示和编辑列数据。当值可为空时，此列类型也支持三态复选框。 |
| `DataGridTemplateColumn` | 可用于自定义列数据在显示和编辑状态下的呈现方式。 |

### 显示行号

绑定到 `DataGridRow.Index`，即可在某一列中显示行号：

```xml
<DataGridTextColumn Header="#"
    Binding="{Binding $parent[DataGridRow].Index}"
    Width="60" IsReadOnly="True" />
```

### 常用属性

这些属性中的大多数都适用于上述三种列类型：

| 属性 | 说明 |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `Header` | 列标题内容。 |
| `HeaderTemplate` | 为该列使用数据模板。 |
| `IsReadOnly` | 该列是否为只读。如果数据网格本身是只读的，那么无论此属性取值如何，该列也都会是只读。 |
| `IsThreeState` | 仅适用于复选框列。当可空 Boolean 值为 null 时，启用第三种（填充）状态。 |
| `Width` | 列宽可以设置为绝对值或相对值（见下文）。 |

### 列宽

如果你没有为某列设置宽度，它会自动调整为适应内容大小，并在必要时为网格添加水平滚动条。

你可以将列宽设置为绝对值，例如：

```xml
<DataGridTextColumn Width="200" />
```

这会导致超出列宽的内容被隐藏。

或者，你也可以指定相对自动尺寸。这里使用 `*` 表示对可用宽度进行等比分配，也可以使用诸如 `2*` 这样的倍数。任何未指定宽度的列都会根据其内容自动确定大小。

例如，要将数据网格划分为 3 个等宽列：

```xml
<DataGridTextColumn Width="*" />
<DataGridTextColumn Width="*" />
<DataGridTextColumn Width="*" />
```

示例

这个示例通过让两列按相同比例扩展宽度，改进了一个数据网格：

```xml
<Window ... >
   <Design.DataContext>
       <vm:MainWindowViewModel/>
  </Design.DataContext>
  <DataGrid Margin="20" ItemsSource="{Binding People}"
          IsReadOnly="True"
          GridLinesVisibility="All"
          BorderThickness="1" BorderBrush="Gray">
    <DataGrid.Columns>
      <DataGridTextColumn Header="First Name" Width="*" 
              Binding="{Binding FirstName}"/>
      <DataGridTextColumn Header="Last Name" Width="*" 
              Binding="{Binding LastName}" />
    </DataGrid.Columns>
  </DataGrid>
</Window>
```

```csharp title='C# View Model'
using AvaloniaControls.Models;
using System.Collections.Generic;
using System.Collections.ObjectModel;

namespace AvaloniaControls.ViewModels
{
    public class MainWindowViewModel : ViewModelBase
    {
        public ObservableCollection<Person> People { get; }

        public MainWindowViewModel()
        {
            var people = new List<Person> 
            {
                new Person("Neil", "Armstrong"),
                new Person("Buzz", "Lightyear"),
                new Person("James", "Kirk")
            };
            People = new ObservableCollection<Person>(people);
        }
    }
}
```

```csharp title='C# Item Class'
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    
    public Person(string firstName , string lastName)
    {
        FirstName = firstName;
        LastName = lastName;
    }
}
```

它之所以能在预览窗格中工作，是因为 `<Design.DataContext>` 元素会创建一个用于绑定的视图模型：

<Image light={DataGridColumnPreviewScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

## See also

- [DataGrid API reference](https://api-docs.avaloniaui.net/docs/T_Avalonia_Controls_DataGrid)
