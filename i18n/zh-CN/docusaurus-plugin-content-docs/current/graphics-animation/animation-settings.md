---
id: animation-settings
title: 动画设置
description: 关键帧动画的配置选项，包括缓动、填充模式和播放方式。
doc-type: reference
---

import LinearEasingScreenshot from '/img/reference/animations-and-graphics/animation-settings/linear-easing.png';
import BackEaseInScreenshot from '/img/reference/animations-and-graphics/animation-settings/back-ease-in.png';
import BackEaseInOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/back-ease-in-out.png';
import BackEaseOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/back-ease-out.png';
import BounceEaseInScreenshot from '/img/reference/animations-and-graphics/animation-settings/bounce-ease-in.png';
import BounceEaseInOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/bounce-ease-in-out.png';
import BounceEaseOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/bounce-ease-out.png';
import CircularEaseInScreenshot from '/img/reference/animations-and-graphics/animation-settings/circular-ease-in.png';
import CircularEaseInOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/circular-ease-in-out.png';
import CircularEaseOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/circular-ease-out.png';
import CubicEaseInScreenshot from '/img/reference/animations-and-graphics/animation-settings/cubic-ease-in.png';
import CubicEaseInOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/cubic-ease-in-out.png';
import CubicEaseOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/cubic-ease-out.png';
import ElasticEaseInScreenshot from '/img/reference/animations-and-graphics/animation-settings/elastic-ease-in.png';
import ElasticEaseInOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/elastic-ease-in-out.png';
import ElasticEaseOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/elastic-ease-out.png';
import ExponentialEaseInScreenshot from '/img/reference/animations-and-graphics/animation-settings/exponential-ease-in.png';
import ExponentialEaseInOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/exponential-ease-in-out.png';
import ExponentialEaseOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/exponential-ease-out.png';
import QuadraticEaseInScreenshot from '/img/reference/animations-and-graphics/animation-settings/quadratic-ease-in.png';
import QuadraticEaseInOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/quadratic-ease-in-out.png';
import QuadraticEaseOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/quadratic-ease-out.png';
import QuarticEaseInScreenshot from '/img/reference/animations-and-graphics/animation-settings/quartic-ease-in.png';
import QuarticEaseInOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/quartic-ease-in-out.png';
import QuarticEaseOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/quartic-ease-out.png';
import QuinticEaseInScreenshot from '/img/reference/animations-and-graphics/animation-settings/quintic-ease-in.png';
import QuinticEaseInOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/quintic-ease-in-out.png';
import QuinticEaseOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/quintic-ease-out.png';
import SineEaseInScreenshot from '/img/reference/animations-and-graphics/animation-settings/sine-ease-in.png';
import SineEaseInOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/sine-ease-in-out.png';
import SineEaseOutScreenshot from '/img/reference/animations-and-graphics/animation-settings/sine-ease-out.png';

本节介绍如何自定义 `Animation` 的播放行为。

## 缓动函数

`Easing` 函数用于描述动画属性在整个动画时间内，从起始值变化到结束值的速度变化方式。`Avalonia.Animation.Easings` 中包含以下缓动函数：

| 默认 |
|---------------------------------------------------------------|
| `LinearEasing`<br/><Image light={LinearEasingScreenshot} alt="Graph showing linear easing curve" position="center" maxWidth={400} cornerRadius="true"/> |

| Ease-In                                                                 | Ease-Out                                                                  | Ease-In-Out                                                                   |
|-------------------------------------------------------------------------|---------------------------------------------------------------------------|-------------------------------------------------------------------------------|
| `SineEaseIn`<br/><Image light={SineEaseInScreenshot} alt="Graph showing SineEaseIn curve" position="center" maxWidth={400} cornerRadius="true"/>               | `SineEaseOut`<br/><Image light={SineEaseOutScreenshot} alt="Graph showing SineEaseOut curve" position="center" maxWidth={400} cornerRadius="true"/>               | `SineEaseInOut`<br/><Image light={SineEaseInOutScreenshot} alt="Graph showing SineEaseInOut curve" position="center" maxWidth={400} cornerRadius="true"/>               |
| `QuadraticEaseIn`<br/><Image light={QuadraticEaseInScreenshot} alt="Graph showing QuadraticEaseIn curve" position="center" maxWidth={400} cornerRadius="true"/>     | `QuadraticEaseOut`<br/><Image light={QuadraticEaseOutScreenshot} alt="Graph showing QuadraticEaseOut curve" position="center" maxWidth={400} cornerRadius="true"/>     | `QuadraticEaseInOut`<br/><Image light={QuadraticEaseInOutScreenshot} alt="Graph showing QuadraticEaseInOut curve" position="center" maxWidth={400} cornerRadius="true"/>     |
| `CubicEaseIn`<br/><Image light={CubicEaseInScreenshot} alt="Graph showing CubicEaseIn curve" position="center" maxWidth={400} cornerRadius="true"/>             | `CubicEaseOut`<br/><Image light={CubicEaseOutScreenshot} alt="Graph showing CubicEaseOut curve" position="center" maxWidth={400} cornerRadius="true"/>             | `CubicEaseInOut`<br/><Image light={CubicEaseInOutScreenshot} alt="Graph showing CubicEaseInOut curve" position="center" maxWidth={400} cornerRadius="true"/>             |
| `QuarticEaseIn`<br/><Image light={QuarticEaseInScreenshot} alt="Graph showing QuarticEaseIn curve" position="center" maxWidth={400} cornerRadius="true"/>         | `QuarticEaseOut`<br/><Image light={QuarticEaseOutScreenshot} alt="Graph showing QuarticEaseOut curve" position="center" maxWidth={400} cornerRadius="true"/>         | `QuarticEaseInOut`<br/><Image light={QuarticEaseInOutScreenshot} alt="Graph showing QuarticEaseInOut curve" position="center" maxWidth={400} cornerRadius="true"/>         |
| `QuinticEaseIn`<br/><Image light={QuinticEaseInScreenshot} alt="Graph showing QuinticEaseIn curve" position="center" maxWidth={400} cornerRadius="true"/>         | `QuinticEaseOut`<br/><Image light={QuinticEaseOutScreenshot} alt="Graph showing QuinticEaseOut curve" position="center" maxWidth={400} cornerRadius="true"/>         | `QuinticEaseInOut`<br/><Image light={QuinticEaseInOutScreenshot} alt="Graph showing QuinticEaseInOut curve" position="center" maxWidth={400} cornerRadius="true"/>         |
| `ExponentialEaseIn`<br/><Image light={ExponentialEaseInScreenshot} alt="Graph showing ExponentialEaseIn curve" position="center" maxWidth={400} cornerRadius="true"/> | `ExponentialEaseOut`<br/><Image light={ExponentialEaseOutScreenshot} alt="Graph showing ExponentialEaseOut curve" position="center" maxWidth={400} cornerRadius="true"/> | `ExponentialEaseInOut`<br/><Image light={ExponentialEaseInOutScreenshot} alt="Graph showing ExponentialEaseInOut curve" position="center" maxWidth={400} cornerRadius="true"/> |
| `CircularEaseIn`<br/><Image light={CircularEaseInScreenshot} alt="Graph showing CircularEaseIn curve" position="center" maxWidth={400} cornerRadius="true"/>       | `CircularEaseOut`<br/><Image light={CircularEaseOutScreenshot} alt="Graph showing CircularEaseOut curve" position="center" maxWidth={400} cornerRadius="true"/>       | `CircularEaseInOut`<br/><Image light={CircularEaseInOutScreenshot} alt="Graph showing CircularEaseInOut curve" position="center" maxWidth={400} cornerRadius="true"/>       |
| `BackEaseIn`<br/><Image light={BackEaseInScreenshot} alt="Graph showing BackEaseIn curve" position="center" maxWidth={400} cornerRadius="true"/>               | `BackEaseOut`<br/><Image light={BackEaseOutScreenshot} alt="Graph showing BackEaseOut curve" position="center" maxWidth={400} cornerRadius="true"/>               | `BackEaseInOut`<br/><Image light={BackEaseInOutScreenshot} alt="Graph showing BackEaseInOut curve" position="center" maxWidth={400} cornerRadius="true"/>             |
| `ElasticEaseIn`<br/><Image light={ElasticEaseInScreenshot} alt="Graph showing ElasticEaseIn curve" position="center" maxWidth={400} cornerRadius="true"/>         | `ElasticEaseOut`<br/><Image light={ElasticEaseOutScreenshot} alt="Graph showing ElasticEaseOut curve" position="center" maxWidth={400} cornerRadius="true"/>         | `ElasticEaseInOut`<br/><Image light={ElasticEaseInOutScreenshot} alt="Graph showing ElasticEaseInOut curve" position="center" maxWidth={400} cornerRadius="true"/>         |
| `BounceEaseIn`<br/><Image light={BounceEaseInScreenshot} alt="Graph showing BounceEaseIn curve" position="center" maxWidth={400} cornerRadius="true"/>           | `BounceEaseOut`<br/><Image light={BounceEaseOutScreenshot} alt="Graph showing BounceEaseOut curve" position="center" maxWidth={400} cornerRadius="true"/>           | `BounceEaseInOut`<br/><Image light={BounceEaseInOutScreenshot} alt="Graph showing BounceEaseInOut curve" position="center" maxWidth={400} cornerRadius="true"/>           |

此外，你也可以通过继承 `Easing`，或为 `SplineEasing`、`SpringEasing` 提供参数，来定义自己的缓动方式。

## FillModes

`Animation` 的 `FillMode` 属性定义了：动画结束后，以及多次运行之间存在延迟时，动画属性值应如何保持。

下表描述了支持的行为：

| 取值       | 说明 |
|------------|-----------------------------------------------------------------------------------------------------------|
| `None`     | 动画结束后不会保留值；如果动画有延迟，也不会提前应用第一个值。 |
| `Forward`  | 最后一个插值结果会保留到目标属性上。 |
| `Backward` | 在动画延迟期间会显示第一个插值值。 |
| `Both`     | 同时应用 `Forward` 和 `Backward` 两种行为。 |

## PlaybackDirection

`PlaybackDirection` 定义了 `Animation` 的播放方式。下表描述了可用设置：

| 取值               | 说明 |
|--------------------|---------------------------------------------------------|
| `Normal`           | 动画按正常方向播放。 |
| `Reverse`          | 动画按反方向播放。 |
| `Alternate`        | 动画先正向播放，再反向播放。 |
| `AlternateReverse` | 动画先反向播放，再正向播放。 |

## IterationCount

`Animation` 元素上的 `IterationCount` 用于设置动画应重复播放多少次。这个设置有两种格式：

| 取值       | 说明 |
|------------|--------------------------------------------------|
| `N`        | `N` 为整数，表示播放 N 次。N 可以为 0。 |
| `Infinite` | 无限重复播放。 |

## 另请参阅

- [关键帧动画](/docs/graphics-animation/keyframe-animations)：在 XAML 中定义关键帧动画。
- [控件过渡](/docs/graphics-animation/control-transitions)：使用过渡为属性变化做动画处理。
- [缓动函数](/docs/graphics-animation/easing-functions)：所有可用的缓动函数。
