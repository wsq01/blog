---
title: CSS代码片段
date: 2020-04-04 20:27:31
tags: CSS
categories: CSS
---

# 截断文本
如果文本长度超过一行，它将被截断并以省略号结束。
仅适用于单个行元素。
```html
<p class="truncate-text">If I exceed one line's width, I will be truncated.</p>
```
```css
.truncate-text {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  width: 200px;
}
```
```
如果宽度超过200像素...
```
## 说明
* `overflow: hidden`：防止文本溢出其尺寸(对于块，100 %宽度和自动高度)。
* `white-space: nowrap`：防止文本高度超过一行。
* `text-overflow: ellipsis`：使其在文本超出其维度时以省略号结尾。
* `width: 200px`：确保元素具有维度，以知道何时获取省略号

# 居中
## 网格居中
使用`grid`在父元素内水平和垂直居中放置子元素。
```html
<div class="grid-centering">
  <div class="child">Centered content.</div>
</div>
```
```css
.grid-centering {
  display: grid;
  justify-content: center;
  align-items: center;
}
```
#### 说明
* `display: grid`：启用网格。
* `justify-content: center`：将子元素水平居中。
* `align-items: center`：将子元素垂直居中。

## Flexbox垂直居中
使用`flexbox`在父元素内水平和垂直居中放置子元素。
```html
<div class="flexbox-centering">
  <div class="child">Centered content.</div>
</div>
```
```css
.flexbox-centering {
  display: flex;
  justify-content: center;
  align-items: center;
}
```
#### 说明
* `display: flex`：启用弹性箱。
* `justify-content: center`：将子元素水平居中。
* `align-items: center`：将子元素垂直居中。

## 定位居中
```html
```

# 动态阴影
创建与类似的阴影`box-shadow`而是基于元素本身的颜色。
```html
<div class="dynamic-shadow-parent">
  <div class="dynamic-shadow"></div>
</div>
```
```css
.dynamic-shadow-parent {
  position: relative;
  z-index: 1;
}
.dynamic-shadow {
  position: relative;
  width: 10rem;
  height: 10rem;
  background: linear-gradient(75deg, #6d78ff, #00ffb8);
}
.dynamic-shadow::after {
  content: '';
  width: 100%;
  height: 100%;
  position: absolute;
  background: inherit;
  top: 0.5rem;
  filter: blur(0.4rem);
  opacity: 0.7;
  z-index: -1;
}
```
## 说明
代码片段需要一些复杂的情况来正确堆叠上下文，这样伪元素将定位在元素本身的下面，同时仍然可见。
* `position: relative`：在父元素上为子元素建立笛卡尔定位上下文。
* `z-index: 1`：建立新的堆叠内容。
* `position: relative`：在子级上建立伪元素的定位上下文。
* `::after`：定义伪元素。
* `position: absolute`：从文档流中取出伪元素，并将其相对于父元素定位。
* `width: 100% 和height: 100%`：调整伪元素的大小以填充其父元素的尺寸，使其大小相等。
* `background: inherit`：使伪元素继承在元素上指定的线性渐变。
* `top: 0.5rem`：将伪元素从其父元素稍微向下偏移。
* `filter: blur(0.4rem)`：将模糊伪元素以在下面创建阴影的外观。
* `opacity: 0.7`：使伪元素部分透明。
* `z-index: -1`：将伪元素定位在父元素后面。

# 渐变文本
为文本提供渐变颜色。
```html
<p class="gradient-text">Gradient text</p>
```
```css
.gradient-text {
  background: -webkit-linear-gradient(pink, red);
  -webkit-text-fill-color: transparent;
  -webkit-background-clip: text;
}
```
{% asset_img img1.png %}
## 说明
* `background: -webkit-linear-gradient(...)`：为文本元素提供渐变背景。
* `webkit-text-fill-color: transparent`：使用透明颜色填充文本。
* `webkit-background-clip: text`：用文本剪辑背景，用渐变背景作为颜色填充文本。

# 鼠标光标渐变跟踪
渐变跟随鼠标光标的悬停效果。
```html
<button class="mouse-cursor-gradient-tracking">
  <span>Hover me</span>
</button>
```
```css
].mouse-cursor-gradient-tracking {
  position: relative;
  background: #7983ff;
  padding: 0.5rem 1rem;
  font-size: 1.2rem;
  border: none;
  color: white;
  cursor: pointer;
  outline: none;
  overflow: hidden;
}
.mouse-cursor-gradient-tracking span {
  position: relative;
}
.mouse-cursor-gradient-tracking::before {
  --size: 0;
  content: '';
  position: absolute;
  left: var(--x);
  top: var(--y);
  width: var(--size);
  height: var(--size);
  background: radial-gradient(circle closest-side, pink, transparent);
  transform: translate(-50%, -50%);
  transition: width 0.2s ease, height 0.2s ease;
}
.mouse-cursor-gradient-tracking:hover::before {
  --size: 200px;
}
```
```js
var btn = document.querySelector('.mouse-cursor-gradient-tracking')
btn.onmousemove = function(e) {
  var x = e.pageX - btn.offsetLeft
  var y = e.pageY - btn.offsetTop
  btn.style.setProperty('--x', x + 'px')
  btn.style.setProperty('--y', y + 'px')
}
```
## 注意
如果元素的父级具有定位上下文(`position: relative`)，您还需要减去它的偏移量。
```js
var x = e.pageX - btn.offsetLeft - btn.offsetParent.offsetLeft
var y = e.pageY - btn.offsetTop - btn.offsetParent.offsetTop
```
# 三角形
使用纯CSS创建三角形形状。
```html
<div class="triangle"></div>
```
```css
.triangle {
  width: 0;
  height: 0;
  border-top: 20px solid #333;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
}
```
## 说明
边框的颜色是三角形的颜色。 三角形尖端点的边对应于相反的`border-*`属性。 例如，`border-top`的颜色表示箭头指向下方。

`px`值来改变三角形的比例。
# 弹跳加载中
创建反弹加载程序动画。
```html
<div class="bouncing-loader">
  <div></div>
  <div></div>
  <div></div>
</div>
```
```css
@keyframes bouncing-loader {
  from {
    opacity: 1;
    transform: translateY(0);
  }
  to {
    opacity: 0.1;
    transform: translateY(-1rem);
  }
}
.bouncing-loader {
  display: flex;
  justify-content: center;
}
.bouncing-loader > div {
  width: 1rem;
  height: 1rem;
  margin: 3rem 0.2rem;
  background: #8385aa;
  border-radius: 50%;
  animation: bouncing-loader 0.6s infinite alternate;
}
.bouncing-loader > div:nth-child(2) {
  animation-delay: 0.2s;
}
.bouncing-loader > div:nth-child(3) {
  animation-delay: 0.4s;
}
```
{% asset_img gif1.gif %}
## 说明
注:`1rem`通常是`16px`。

* `@keyframes`定义了一个具有两种状态的动画，其中元素更改`opacity`并使用`transform: translateY()`在2D平面上进行`transform: translateY()`。
* `.bouncing-loader`是弹跳圆圈的父容器，使用`display: flex`和`justify-content: center`将它们放置在中心位置。
* `.bouncing-loader > div`，将父级的三个子`div`作为样式。`div`的宽度和高度为`1rem`，使用`border-radius: 50%`将它们从正方形变成圆形。
* `margin: 3rem 0.2rem`指定每个圆的上边距/下边距为`3rem`和左/右边距`0.2rem`以便它们不直接接触对方，给它们一些呼吸空间。
* `animation`是各种动画属性的缩写属性：使用`animation-name`，`animation-duration`，* `animation-iteration-count`， `animation-direction`。
* `nth-child(n)`目标是其父元素的第n个子元素。
* `animation-delay`分别用于第二个和第三个`div`，以便每个元素不会同时启动动画。

# 环形旋转器
创建可用于指示内容加载的圆环微调器。
```html
<div class="donut"></div>
```
```css
@keyframes donut-spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
.donut {
  display: inline-block;
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-left-color: #7983ff;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  animation: donut-spin 1.2s linear infinite;
}
```
{% asset_img gif2.gif %}
## 说明
使用半透明的`border`对于整个元素，除了用作圆环加载指示器的一侧。使用`animation`旋转元素。
# 悬停下划线动画
当文本悬停在上面时创建动画下划线效果。
```html
<p class="hover-underline-animation">Hover this text to see the effect!</p>
```
```css
.hover-underline-animation {
  display: inline-block;
  position: relative;
  color: #0087ca;
}
.hover-underline-animation::after {
  content: '';
  position: absolute;
  width: 100%;
  transform: scaleX(0);
  height: 2px;
  bottom: 0;
  left: 0;
  background-color: #0087ca;
  transform-origin: bottom right;
  transition: transform 0.25s ease-out;
}
.hover-underline-animation:hover::after {
  transform: scaleX(1);
  transform-origin: bottom left;
}
```
{% asset_img gif3.gif %}
## 说明
* `display: inline-block`使块`pp`成为`一inline-block`以防止下划线跨越整个父级宽度而不仅仅是内容(文本)。
* `position: relative`在元素上为伪元素建立笛卡尔定位上下文。
* `::after`定义伪元素。
* `position: absolute`从文档流中取出伪元素，并将其相对于父元素定位。
* `width: 100%`确保伪元素跨越文本块的整个宽度。
* `transform: scaleX(0)`最初将伪元素缩放为0，使其没有宽度且不可见。
* `bottom: 0 和left: 0`将其放置在块的左下方。
* `transition: transform 0.25s ease-out`意味着`transform`变化将通过`ease-out`时间功能在0.25秒内过渡。
* `transform-origin: bottom right`表示变换锚点位于块的右下方。
* `:hover::after`然后使用`scaleX(1)`将宽度转换为100％，然后将`transform-origin`更改为`bottom left`以便定位点反转，从而允许其在悬停时转换到另一个方向。

# 交错动画
通过设置不同的延迟时间，达到动画交错播放的效果。

举个栗子，比如有十个元素播放十个动画，将第二个元素的动画播放时间设定为比第一个元素晚0.5秒（也就是将延迟设为0.5秒），其他元素以此类推，这样它们就会错开来，形成一种独特的视觉效果。
```html
<div class="loading">
  <div class="dot"></div>
  <div class="dot"></div>
  <div class="dot"></div>
  <div class="dot"></div>
  <div class="dot"></div>
</div>
```
```less
.loading {
  $colors: #7ef9ff, #89cff0, #4682b4, #0f52ba, #000080;
  display: flex;
  animation-delay: 1s;

  .dot {
    position: relative;
    width: 2em;
    height: 2em;
    margin: 0.8em;
    border-radius: 50%;

    &::before {
      position: absolute;
      content: "";
      width: 100%;
      height: 100%;
      background: inherit;
      border-radius: inherit;
      animation: wave 2s ease-out infinite;
    }

    @for $i from 1 through 5 {
      &:nth-child(#{$i}) {
        background: nth($colors, $i);

        &::before {
          animation-delay: $i * 0.2s;
        }
      }
    }
  }
}

@keyframes wave {
  50%, 75% {
    transform: scale(2.5);
  }

  80%, 100% {
    opacity: 0;
  }
}
```
{% asset_img gif4.gif %}
