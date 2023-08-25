---
title: 现代CSS解决方案：原生支持的三角函数
date: 2023-08-25 14:11:48
tags:
  - CSS
---

## CSS 三角函数语法介绍
首先看看 CSS 三角函数的使用方式：

```css
.box {
  /* 设置元素的宽度为 sin(30deg) 的值 */
  width: calc(sin(30deg) * 100px);

  /* 反正弦 根据弧度值返回对应的角度值 */
  transform: rotate(asin(-0.2));

  /* 设置元素的高度为 cos(45deg) 的值 */
  height: calc(cos(45deg) * 100%);

  /* 设置元素的透明度为 tan(60deg) 的值 */
  opacity: calc(tan(60deg));
}
```

上述代码中，使用了 calc() 函数进行了计算，然后通过 sin()、cos() 和 tan() 函数对计算结果进行了进一步的处理，从而实现了不同的效果。

三角函数在 CSS3 中仅对弧度（radian）单位进行支持。如果想要在开发中使用三角函数，可以借助转换函数 deg() 和 rad() 将角度（degree）和弧度进行转换。

CSS3 的这些函数使得开发者可以更加方便处理一些复杂的数学问题，增强了 CSS 的表现力。

## 三角函数的运动轨迹
三角函数的运用，可以体现在动画当中。以正弦、余弦函数为例，代码如下：

```css
@property --angle {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

@property --dis {
  syntax: '<length>';
  inherits: false;
  initial-value: 0px;
}

.g-single {
  background: #000;
  margin: auto;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  animation: move 5s infinite ease-in-out;
  transform: translate(calc(var(--dis) - 40vw),
      calc(5 * sin(var(--angle)) * 1em));
}

@keyframes move {
  0% {
    --dis: 0px;
    --angle: 0deg;
  }

  100% {
    --dis: 80vw;
    --angle: 1080deg;
  }
}
```

上述的核心在于这一段代码 `-- transform: translate(calc(var(--dis) - 40vw), calc(5 * sin(var(--angle)) * 1em))`
内部使用了两个 CSS @property 变量：

1. x 轴方向是 `0px` 到 `80vw` 的水平位移动画
2. y 轴方向是 `5 * sin(0deg) * 1em` 到 `5 * sin(1080deg) * 1em` 的竖直动画

通过动画，动态的修改这两个变量的值，就可以得到一个三角函数曲线动画图形：

<iframe src="../../html/sinAnimation.html" scrolling="no" width="100%" height="200px" frameborder="0" ></iframe>

## CSS 钟表

钟表上1-12折12个数字按照圆形等间距排布，也是CSS三角函数的典型应用。

排版定位的相关代码：
```css
.clock-face time {
  --x: calc(var(--radius) + (var(--radius) * cos(var(--index) * 30deg)));
  --y: calc(var(--radius) + (var(--radius) * sin(var(--index) * 30deg)));
  display: grid;
  place-content: center;
  height: 2em; width: 2em;
  position: absolute;
  left: var(--x);
  top: var(--y);
}

.clock-face time:nth-child(1) { --index: 9; }
.clock-face time:nth-child(2) { --index: 10; }
.clock-face time:nth-child(3) { --index: 11; }
.clock-face time:nth-child(4) { --index: 0; }
.clock-face time:nth-child(5) { --index: 1; }
.clock-face time:nth-child(6) { --index: 2; }
.clock-face time:nth-child(7) { --index: 3; }
.clock-face time:nth-child(8) { --index: 4; }
.clock-face time:nth-child(9) { --index: 5; }
.clock-face time:nth-child(10) { --index: 6; }
.clock-face time:nth-child(11) { --index: 7; }
.clock-face time:nth-child(12) { --index: 8; }
```

实现的效果（完整的代码可以查看一下iframe的源代码信息）：

<iframe src="../../html/cssClock.html" width="100%" height="340px" scrolling="no" frameborder="0" ></iframe>

## CSS 数学函数

- sqrt() 求平方根
- pow() 幂指数
- exp() 自然常数e为底的指数函数
- log() 对数函数
- abs() 绝对值
- round() 四舍五入
- sign() 正负零判断
- ....

## 总结

CSS 原生支持的三角函数，给 CSS 打开了更多的可能性，这也导致 CSS 的复杂度也是愈来愈高，CSS 已经不再是非常纯粹的负责样式了，很多时候，很多计算也可以直接在 CSS 当中完成。至于好坏，或许已经成了一个哲学问题。

记录两个css项目，最后一个相当炸裂：

[css-doodle](https://css-doodle.com/)

[css-fps-demo](https://keithclark.co.uk/labs/css-fps/)