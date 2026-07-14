---
id: splitbutton
title: SplitButton
---

import SplitButtonPaletteFlyoutScreenshot from '/img/controls/buttons/splitbutton/splitbutton-palette-flyout.png';

[`SplitButton`](/api/avalonia/controls/splitbutton) 的行为类似一个带有主区域和次区域的 [`Button`](/controls/input/buttons/button)，这两个部分都可以分别按下。主区域的行为与普通 `Button` 相同，而次区域会打开一个包含附加操作的 [`Flyout`](/controls/menus/menuflyout)。

## 这是合适的控件吗？

`SplitButton` 只适合由相似操作组成。从本质上说，这个控件用于将一组常见操作归类到一起，其中一个操作相较其他操作有更明确的优先级。最常用的操作应作为默认操作，并显示在 SplitButton 的主区域中。使用频率较低的操作应被放到次区域按下时显示的 flyout 中。

:::info
无论用户按下主区域，还是在 flyout 中选择某个次级操作，对应动作都应该立即执行。所有被按下的操作，无论主次，都应是即时操作。
:::

## 常用属性

| 属性 | 说明 |
| --------- | -------------------------------------------------------------- |
| `Content` | 显示在主区域中的内容 |
| [`Flyout`](/api/avalonia/controls/flyout) | 当次区域被点击时显示的 `Flyout` |
| `Command` | 当主按钮被点击时要调用的命令 |
| `HotKey` | 激活主按钮操作的键盘快捷键 |

## 伪类

| 伪类 | 说明 |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `:pressed` | 当整个 `SplitButton` 通过键盘输入（如 Space 或 Enter）被按下时设置。在这种状态下，不区分主区域和次区域。 |
| `:flyout-open` | 当 `Flyout` 打开时设置。 |

## 示例

### 基本示例

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             Padding="20">
  <SplitButton Content="Content">
    <SplitButton.Flyout>
        <MenuFlyout Placement="Bottom">
            <MenuItem Header="Item 1">
                <MenuItem Header="Subitem 1" />
                <MenuItem Header="Subitem 2" />
                <MenuItem Header="Subitem 3" />
            </MenuItem>
            <MenuItem Header="Item 2"
                      InputGesture="Ctrl+A" />
            <MenuItem Header="Item 3" />
        </MenuFlyout>
    </SplitButton.Flyout>
  </SplitButton>
</UserControl>
```

</XamlPreview>

### 颜色选择示例

`SplitButton` 的一个常见用途是在编辑器中为文本着色。按下 `SplitButton` 的主区域会将当前颜色应用到选中文本上；按下次区域则会打开一个 `Flyout`，让用户选择并应用另一种颜色。还需要注意的是，当用户在 `Flyout` 中指定另一种颜色时，选中文本的颜色会立即改变，同时当前颜色也会被更新。

<Image light={SplitButtonPaletteFlyoutScreenshot} alt="" position="center" maxWidth={400} cornerRadius="true"/>

```xml
<!-- We have the following DataTemplate defined -->
<DataTemplate DataType="Color">
   <Border CornerRadius="4" Width="20" Height="20" BorderBrush="Gray" BorderThickness="1">
    <Border.Background>
       <SolidColorBrush Color="{Binding}" />
   </Border.Background>
  </Border>
</DataTemplate>
```

```xml
<!-- SelectedColor, ChangeColorCommand and AvailableColors are properties of our ViewModel -->
<SplitButton Content="{Binding SelectedColor}" 
             Command="{Binding ChangeColorCommand}">
  <SplitButton.Flyout>
    <Flyout Placement="Bottom">
      <ListBox ItemsSource="{Binding AvailableColors}" 
               SelectedItem="{Binding SelectedColor}" 
               Height="200" Width="200">
        <ListBox.ItemsPanel>
          <ItemsPanelTemplate>
            <WrapPanel />
          </ItemsPanelTemplate>
        </ListBox.ItemsPanel>
      </ListBox>
    </Flyout>
  </SplitButton.Flyout>
</SplitButton>
```

### 导出按钮示例

`SplitButton` 的另一个常见示例是导出按钮。当按下主区域时，数据会使用默认设置进行导出；而按下次区域时，则可以指定额外的导出选项，例如“导出为 PNG”、“导出为 JPG”等。

<XamlPreview>

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             Padding="20">
  <SplitButton Content="Export to PDF"
               Command="{Binding ExportCommand}"
               CommandParameter=".pdf">
     <SplitButton.Flyout>
          <MenuFlyout Placement="RightEdgeAlignedTop">
              <MenuItem Header="Export to PNG"
                        Command="{Binding ExportCommand}"
                        CommandParameter=".png" />
              <MenuItem Header="Export to JPG"
                        Command="{Binding ExportCommand}"
                        CommandParameter=".jpg" />
         </MenuFlyout>
      </SplitButton.Flyout>
  </SplitButton>
</UserControl>
```

</XamlPreview>

## 另请参阅

- [SplitButton API reference](/api/avalonia/controls/splitbutton)
- [`SplitButton.cs` source code on GitHub](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/SplitButton/SplitButton.cs)
