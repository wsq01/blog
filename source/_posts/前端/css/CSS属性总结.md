


# :has() 选择器
`:has()`选择器允许我们根据子元素来为父元素设置样式。
```css
div { background: white; }
div:has(img) { background: grey; }
```
如果`div`元素内有`img`，则将其背景更改为灰色。
# 自定义滚动条

{% asset_img 1.png %}

* `div::-webkit-scrollbar`滚动条整体部分
* `div::-webkit-scrollbar-thumb`滚动条里面的小方块，能向上向下移动或往左往右移动，取决于是垂直滚动条还是水平滚动条
* `div::-webkit-scrollbar-track`滚动条的轨道（里面装有 Thumb
* `div::-webkit-scrollbar-button`滚动条的轨道的两端按钮，允许通过点击微调小方块的位置
* `div::-webkit-scrollbar-track-piece`内层轨道，滚动条中间部分（除去
* `div::-webkit-scrollbar-corner`边角，即两个滚动条的交汇处
* `div::-webkit-resizer`两个滚动条的交汇处上用于通过拖动调整元素大小的小控件注意此方案有兼容性问题，一般需要隐藏滚动条时我都是用一个色块通过定位盖上去，或者将子级元素调大，父级元素使用`overflow-hidden`截掉滚动条部分。暴力且直接。

```css
/* 设置滚动条的宽度 */

::-webkit-scrollbar{
  width: 10px;
}
/* 将轨道改为蓝色并设置圆角边框 */
::-webkit-scrollbar-track{
  background-color: blue;
  border-radius: 10px;
}
/* 将滑块（显示滚动量）改为灰色并设置圆角 */
::-webkit-scrollbar-thumb{
  background: gray;
  border-radius: 10px;
}
/* 悬停时显示为深灰色 */
::-webkit-scrollbar-thumb:hover{
  background: darkgray;
}
```
{% asset_img 2.png %}

# Scroll behavior
滚动行为可以实现平滑滚动，使页面在不同部分之间的过渡更加平滑。
```css
html{
  scroll-behavior:smooth;
}
```
# accent-color
改变用户界面的颜色，例如表单控件和复选框。
```css
input{
  accent-color: blue;
}
```
{% asset_img 3.png %}

# Aspect Ratio
在构建响应式组件时，经常检查高度和宽度可能会令人头疼，因为你必须保持纵横比。有时候视频和图片可能会显得拉伸。

这就是为什么我们可以使用纵横比属性。一旦设置了纵横比值，然后再设置宽度，高度就会自动设置。或者反之亦然。
```css
/* class为example的元素 */
.example{
  /* 设置纵横比 */
  aspect-ratio: 1 / .25;
  /* 设置宽度后，高度会自动设置 */
  width: 200px;
  /* 边框不是必需的，但这里只是为了看效果而添加的 */
  border: solid black 1px;
}
```
现在，我们设置了宽度，高度将自动设置为`50px`，以保持纵横比。
# position: sticky
当视口到达定义的位置时，元素会粘在那里。
```css
.some-component{
  position: sticky;
  top: 0;
} 
```
`sticky`定位有两个主要部分：
* 粘性元素——是我们使用`position: sticky`样式定义的元素。当视口位置与位置定义匹配时，元素将浮动，例如：`top: 0px`。
* 粘性容器——是包裹粘性项目的 HTML 元素。这是粘性项目可以浮动的最大区域。

当你使用`position: sticky`定义一个元素时，自动定义了父元素为粘性容器。容器是粘性项目的作用域，项目无法离开其粘性容器。

当一个具有`sticky`定位样式的元素被包裹起来，并且它是包裹元素内唯一的元素时，这个被定义为`sticky`定位的元素并不会"粘"住。原因是，当一个元素被赋予`sticky`定位样式时，粘性元素的容器是粘性元素可以粘住的唯一区域。这个元素没有其他元素可以浮动，因为它只能浮动在兄弟元素上，而作为唯一的子元素，它没有兄弟元素。