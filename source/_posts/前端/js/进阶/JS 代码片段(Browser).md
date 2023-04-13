

## 开启全屏
```js
export const launchFullscreen = (element) => {
  if (element.requestFullscreen) {
    element.requestFullscreen()
  } else if (element.mozRequestFullScreen) {
    element.mozRequestFullScreen()
  } else if (element.msRequestFullscreen) {
    element.msRequestFullscreen()
  } else if (element.webkitRequestFullscreen) {
    element.webkitRequestFullScreen()
  }
}
```
## 关闭全屏
```js
export const exitFullscreen = () => {
  if (document.exitFullscreen) {
    document.exitFullscreen()
  } else if (document.msExitFullscreen) {
    document.msExitFullscreen()
  } else if (document.mozCancelFullScreen) {
    document.mozCancelFullScreen()
  } else if (document.webkitExitFullscreen) {
    document.webkitExitFullscreen()
  }
}
```

## 解析URL参数
```js
export const getSearchParams = () => {
  const searchPar = new URLSearchParams(window.location.search)
  const paramsObj = {}
  for (const [key, value] of searchPar.entries()) {
    paramsObj[key] = value
  }
  return paramsObj
}

// 假设目前位于 https://****com/index?id=154513&age=18;
getSearchParams(); // {id: "154513", age: "18"}
```
## 滚动到页面顶部
```js
export const scrollToTop = () => {
  const height = document.documentElement.scrollTop || document.body.scrollTop;
  if (height > 0) {
    window.requestAnimationFrame(scrollToTop);
    window.scrollTo(0, height - height / 8);
  }
}
```
## 滚动到元素位置
```js
export const smoothScroll = element =>{
  document.querySelector(element).scrollIntoView({
    behavior: 'smooth'
  });
};
```
## parseCookie
解析`Cookie`标头字符串并返回所有`cookie`的`name-value`对的对象。

* 使用`String.split(';')`将键值对彼此分开。
* 使用`Array.map()`和`String.split('=')`将键与每对中的值分开。
* 使用`Array.reduce()`和`decodeURIComponent()`创建一个包含所有键值对的对象。

```js
const parseCookie = str =>
  str
    .split(';')
    .map(v => v.split('='))
    .reduce((acc, v) => {
      acc[decodeURIComponent(v[0].trim())] = decodeURIComponent(v[1].trim());
      return acc;
    }, {});

parseCookie('foo=bar; equation=E%3Dmc%5E2'); // { foo: 'bar', equation: 'E=mc^2' }
```
## 获取选择的文本
```js
const getSelectedText = () => window.getSelection().toString()

getSelectedText(); // 'Lorem ipsum'
```