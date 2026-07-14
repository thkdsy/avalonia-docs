---
id: maskedtextbox
title: MaskedTextBox
---

import MaskedTextPhoneBoxScreenshot from '/img/reference/controls/maskedtextbox/maskedtextbox-phone.gif';

`MaskedTextBox` 提供一个用于键盘输入的区域，但可以通过由特殊字符组成的掩码模式来限制允许的格式和字符。

掩码模式还可以包含会直接显示在输入结果中的字面字符，这些字符不能被用户覆盖输入。

## 常用属性

你最常使用的通常是这些属性：

| 属性    | 说明                                                                  |
|-------------|------------------------------------------------------------------------------|
| `Mask`      | 要使用的掩码模式。下表列出了可用的特殊掩码字符。 |
| `AsciiOnly` | 将输入限制为 ASCII 字母 a-z 和 A-Z。                            |
| `Text`      | 最终得到的输入文本，包含所有字面字符。                   |

## 掩码字符

`Mask` 属性接受一个字符串，其中可以同时包含固定字符和以下特殊字符：

| 掩码字符 | 说明                                                                                                                                                                             |
|:--------------:|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|      `0`       | 必填数字。接受 0 到 9 之间的任意单个数字。                                                                                                             |
|      `9`       | 可选数字或空格。                                                                                                                                                               |
|      `#`       | 可选数字或空格。如果该位置未填写，则在 `Text` 属性中会显示为空格。允许输入加号（+）和减号（-）。                         |
|      `L`       | 必填字母。将输入限制为 ASCII 字母 a-z 和 A-Z。                                                                                                                      |
|      `?`       | 可选字母。将输入限制为 ASCII 字母 a-z 和 A-Z。                                                                                                                      |
|      `&`       | 必填字符。如果 `AsciiOnly` 属性为 true，则该元素行为类似于 `L`。                                                                                      |
|      `C`       | 可选字符。可接受任意非控制字符。如果 `AsciiOnly` 属性为 true，则该元素行为类似于 `?`。                                                    |
|      `A`       | 必填字母数字。如果 `AsciiOnly` 属性为 true，则只接受 ASCII 字母 a-z 和 A-Z。该掩码元素的行为类似于 `a`。        |
|      `a`       | 可选字母数字。如果 `AsciiOnly` 属性为 true，则只接受 ASCII 字母 a-z 和 A-Z。该掩码元素的行为类似于 `A`。 |
|      `.`       | 小数占位符。实际显示的字符会根据控件 `FormatProvider` 属性确定的格式提供程序，使用相应的小数分隔符。           |
|      `,`       | 千分位占位符。实际显示的字符会根据控件 `FormatProvider` 属性确定的格式提供程序，使用相应的千位分隔符。  |
|      `:`       | 时间分隔符。实际显示的字符会根据控件 `FormatProvider` 属性确定的格式提供程序，使用相应的时间分隔符。                   |
|      `/`       | 日期分隔符。实际显示的字符会根据控件 `FormatProvider` 属性确定的格式提供程序，使用相应的日期分隔符。                   |
|      `$`       | 货币符号。实际显示的字符会根据控件 `FormatProvider` 属性确定的格式提供程序，使用相应的货币符号。                 |
|      `<`       | 小写转换。将后续所有字符转换为小写。                                                                                                                           |
|      `>`       | 大写转换。将后续所有字符转换为大写。                                                                                                                             |
|      `\|`      | 取消之前的大小写转换设置。                                                                                                                                              |
|      `\`       | 转义字符。将一个掩码字符转义为普通字面字符。                                                                                                                            |

转义字符（反斜杠）可用于将特殊字符作为普通字面字符包含进来。例如，如果要包含美元符号：

`Mask="\$999,000.00"`

## 示例

这是一个基本示例：

<XamlPreview>

```xml
<StackPanel xmlns="https://github.com/avaloniaui"
            Margin="20">
  <TextBlock Margin="0 5">International phone number:</TextBlock>
  <MaskedTextBox Mask="(+09) 000 000 0000" />
  <TextBlock Margin="0 15 0 5">UK VAT number:</TextBlock>
  <MaskedTextBox Mask="GB 000 000 000" />
</StackPanel>
```

</XamlPreview>

## 另请参阅

- [`MaskedTextBox.cs` GitHub 源代码](https://github.com/AvaloniaUI/Avalonia/blob/master/src/Avalonia.Controls/MaskedTextBox.cs)
