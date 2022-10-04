---
title: CSS常见问题
date: 2020-04-02 20:27:31
tags: CSS
categories: CSS
---

# display:inline-block
两个`display: inline-block`元素放到一起会产生一段空白。
```html
<div class="container">
  <div class="left">左</div>
  <div class="right">右</div>
</div>
```
```css
.left {
  font-size: 14px;
  background: red;
  display: inline-block;
  width: 100px;
  height: 100px;
}

.right {
  font-size: 14px;
  background: blue;
  display: inline-block;
  width: 100px;
  height: 100px;
}
```
{% asset_img img1.png %}

## 产生空白的原因
元素被当成行内元素排版的时候，元素之间的空白符（空格、回车换行等）都会被浏览器处理，根据CSS中`white-space`属性的处理方式（默认是normal，合并多余空白），原来HTML代码中的回车换行被转成一个空白符，在字体不为0的情况下，空白符占据一定宽度，所以`inline-block`的元素之间就出现了空隙。
## 解决办法
1. 将子元素标签的结束符和下一个标签的开始符写在同一行或把所有子标签写在同一行。
```html
<div class="container">
  <div class="left">左</div><div class="right">右</div>
</div>
```
2. 父元素中设置`font-size: 0`，在子元素上重置正确的`font-size`。
```css
.container{
  font-size: 0;
}
```
3. 为子元素设置`float: left`。
```css
.left{
  float: left;
  font-size: 14px;
  background: red;
  display: inline-block;
  width: 100px;
  height: 100px;
}
/* right是同理 */
```
# background-position
在CSS中，背景图片的定位方法有3种：
* 关键字：`background-position: top left;`
* 像素：`background-position: 0px 0px;`
* 百分比：`background-position: 0% 0%;`

上面这三句语句，都将图片定位在背景的左上角，表面上看效果是一样的，实际上第三种定位机制与前两种完全不同。

前两种定位，都是将背景图片左上角的原点，放置在规定的位置。
但是第三种定位，也就是百分比定位，不是这样。它的放置规则是，图片本身`（x%,y%）`的那个点，与背景区域的`（x%,y%）`的那个点重合。
# 删除 type="number" 末尾的箭头

{% asset_img 2.gif %}

```css
.no-arrow::-webkit-outer-spin-button,
.no-arrow::-webkit-inner-spin-button {
  -webkit-appearance: none;
}
```
# outline:none 删除输入状态线
当输入框被选中时，它默认会有一条蓝色的状态线，可以通过使用`outline: none`来移除它。

# 隐藏滚动条

{% asset_img 3.gif %}

```css
.box-hide-scrollbar::-webkit-scrollbar {
  display: none; /* Chrome Safari */
}
```
# 不允许选择文本
```css
.box p:last-child {
  user-select: none;
}
```
# 单行文本溢出时显示省略号
```css
overflow: hidden;
white-space: nowrap;
text-overflow: ellipsis;
max-width: 375px;
```
# 多行文本溢出时显示省略号
```css
overflow: hidden;
text-overflow: ellipsis;

display: -webkit-box;
/* set n lines, including 1 */
-webkit-line-clamp: 2;
-webkit-box-orient: vertical;
```
# 解决 img 5px 间距的问题
方案1：设置父元素字体大小为 0
```css
.img-container {
  font-size: 0;
}
```
方案2：将`img`元素设置为`display: block`
```css
img {
  display: block;
}
```
方案3：将`img`元素设置为`vertical-align: bottom`
```css
img {
  vertical-align: bottom;
}
```
解决方案4：给父元素设置`line-height: 5px`
```css
.img-container {
  line-height: 5px;
}
```
# 修改 input placeholder 样式
```css
.placehoder-custom::-webkit-input-placeholder {
  color: #babbc1;
  font-size: 12px;
}
```
# IOS 手机容器滚动条滑动不流畅
```
overflow: auto;
-webkit-overflow-scrolling: touch;
```
# 修改滚动条样式
```css
div::-webkit-scrollbar {
  display: none;
}
```
* `div::-webkit-scrollbar`滚动条整体部分
* `div::-webkit-scrollbar-thumb`滚动条里面的小方块，能向上向下移动或往左往右移动，取决于是垂直滚动条还是水平滚动条
* `div::-webkit-scrollbar-track`滚动条的轨道（里面装有 Thumb
* `div::-webkit-scrollbar-button`滚动条的轨道的两端按钮，允许通过点击微调小方块的位置
* `div::-webkit-scrollbar-track-piece`内层轨道，滚动条中间部分（除去
* `div::-webkit-scrollbar-corner`边角，即两个滚动条的交汇处
* `div::-webkit-resizer`两个滚动条的交汇处上用于通过拖动调整元素大小的小控件注意此方案有兼容性问题，一般需要隐藏滚动条时我都是用一个色块通过定位盖上去，或者将子级元素调大，父级元素使用`overflow-hidden`截掉滚动条部分。暴力且直接。

# 平滑滚动
```css
html {
  scroll-behavior: smooth;
}
```
# 禁用textarea文本框调整大小
```css
textarea.no-resize {
  resize: none;
}

textarea.horizontal-resize {
  resize: horizontal;
}

textarea.vertical-resize {
  resize: vertical;
}
```

# 首字下沉
```css
::first-letter {
  font-size: 250%;
}
```
# 改变输入框光标的颜色
```css
input {
  caret-color: red;
}
```
# 限制输入框值的范围
```css
input:in-range {
  background: rgba(0, 255, 0, .25);
}

input:out-of-range {
  background: rgba(255, 0, 0, .25);
}
```