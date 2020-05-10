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
// right是同理
```
# background-position
在CSS中，背景图片的定位方法有3种：
* 关键字：`background-position: top left;`
* 像素：`background-position: 0px 0px;`
* 百分比：`background-position: 0% 0%;`

上面这三句语句，都将图片定位在背景的左上角，表面上看效果是一样的，实际上第三种定位机制与前两种完全不同。

前两种定位，都是将背景图片左上角的原点，放置在规定的位置。
但是第三种定位，也就是百分比定位，不是这样。它的放置规则是，图片本身（x%,y%）的那个点，与背景区域的（x%,y%）的那个点重合。