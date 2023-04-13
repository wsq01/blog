


| 光源名称 | 描述 |
| :--: | :--: |
| AmbientLight | 它的颜色会添加到整个场景和所有对象的当前颜色上 |
| PointLight | 从一个点向各个方向发射的光源 |
| SportLight | 这种光源有聚光的效果，类似台灯、手电筒 |
| DirectionalLight | 平行光是沿着特定方向发射的光 |
| HemisphereLight | 可以用来创建更加自然的室外光线、模拟反光面和光线微弱的天空 |

# AmbientLight(环境光)
环境光会均匀的照亮场景中的所有物体。环境光会影响整个场景。环境光的光线没有特定的来源，而且这个光源也不会影响阴影的生成，即环境光不能用来投射阴影，因为它没有方向。

不能将环境光作为场景中的唯一光源，需要配合其他光源使用，目的是弱化阴影或添加一些颜色。
## 创建AmbientLight
```js
new THREE.AmbientLight( color: Integer, intensity: Float )
```
参数说明：
* `color`：(参数可选）颜色的`rgb`数值。缺省值为`0xffffff`。
* `intensity`：(参数可选)光照的强度。缺省值为 1。

```js
var ambiColor = "#0c0c0c"
var ambientLight = new THREE.AmbientLight(ambiColor)
scene.add(ambientLight)
```
# PointLight(点光源)
点光源从一个点向各个方向发射的光源。一个常见的例子是模拟一个灯泡发出的光。该光源可以投射阴影。
## 创建点光源
```js
new THREE.PointLight( color: Integer, intensity: Float, distance: Number, decay: Float )
```
参数说明：
* `color`：光源颜色(可选参数)，十六进制光照颜色，缺省值`0xffffff`。
* `intensity`：光照强度(可选参数)，当设置为 0 时，什么都看不到，设成 2，得到的是两倍的亮度，缺省值 1。
* `distance`：光源照射的距离，这个距离表示从光源到光照强度为 0 的位置。 当设置为 0 时，光永远不会消失(距离无穷大)。缺省值 0。
* `decay`：沿着光照距离的衰退量。缺省值 1。在`physically correct`模式中，`decay = 2`。

```js
var pointColor = "#ccffcc"
var pointLight = new THREE.PointLight(pointColor)
pointLight.distance = 100
scene.add(pointLight)
```
# SpotLight(聚光灯)
光线从一个点沿一个方向射出，随着光线照射的变远，光线圆锥体的尺寸也逐渐增大。该光源可以投射阴影。
## 创建聚光灯
```js
new THREE.SpotLight( color: Integer, intensity: Float, distance: Float, angle: Radians, penumbra: Float, decay: Float )
```
参数说明：
* `color`：(可选参数)十六进制光照颜色。 缺省值`0xffffff`。
* `intensity`：(可选参数)光照强度。 缺省值 1。
* `distance`：从光源发出光的最大距离，其强度根据光源的距离线性衰减。
* `angle`：光线散射角度，最大为`Math.PI/2`。
* `penumbra`：聚光锥的半影衰减百分比。在 0 和 1 之间的值，默认为 0。
* `decay`：沿着光照距离的衰减量。

```js
var pointColor = "#ffffff"
var spotLight = new THREE.SpotLight(pointColor)
spotLight.position.set(-40, 60, -10)
spotLight.castShadow = true
spotLight.shadowCameraNear = 2
spotLight.shadowCameraFar = 200
spotLight.shadowCameraFov = 30
spotLight.target = plane
spotLight.distance = 0
spotLight.angle = 0.4

scene.add(spotLight);
```
# DirectionalLight(平行光)
平行光是沿着特定方向发射的光。这种光的表现像是无限远，从它发出的光线都是平行的。常常用平行光来模拟太阳光的效果；太阳足够远，因此我们可以认为太阳的位置是无限远，所以我们认为从太阳发出的光线也都是平行的。平行光可以投射阴影。
## 创建平行光
```js
new THREE.DirectionalLight( color: Integer, intensity: Float )
```
```js
var pointColor = "#ff5808"
var directionalLight = new THREE.DirectionalLight(pointColor)
directionalLight.position.set(-40, 60, -10)
directionalLight.castShadow = true
directionalLight.shadowCameraNear = 2
directionalLight.shadowCameraFar = 200
directionalLight.shadowCameraLeft = -50
directionalLight.shadowCameraRight = 50
directionalLight.shadowCameraTop = 50
directionalLight.shadowCameraBottom = -50

directionalLight.distance = 0
directionalLight.intensity = 0.5
directionalLight.shadowMapHeight = 1024
directionalLight.shadowMapWidth = 1024

scene.add(directionalLight)
```
# 半球光(HemisphereLight)
光源直接放置于场景之上，光照颜色从天空光线颜色渐变到地面光线颜色。半球光不能投射阴影。
## 创建半球光
```js
new THREE.HemisphereLight( skyColor: Integer, groundColor: Integer, intensity: Float )
```
参数说明：
* `skyColor`：(可选参数) 天空中发出光线的颜色。缺省值`0xffffff`。
* `groundColor`：(可选参数) 地面发出光线的颜色。缺省值`0xffffff`。
* `intensity`：(可选参数) 光照强度。缺省值 1。

```js
var hemiLight = new THREE.HemisphereLight(0x0000ff, 0x00ff00, 0.6)
hemiLight.position.set(0, 500, 0)
scene.add(hemiLight)
```
# 平面光光源（RectAreaLight）
平面光光源从一个矩形平面上均匀地发射光线。这种光源可以用来模拟像明亮的窗户或者条状灯光光源。

注意事项:
* 不支持阴影。
* 只支持`MeshStandardMaterial`和`MeshPhysicalMaterial`两种材质。
* 你必须在你的场景中加入`RectAreaLightUniformsLib`，并调用`init()`。

## 创建平面光
```js
RectAreaLight( color : Integer, intensity : Float, width : Float, height : Float )
```
参数说明：
* `color`：(可选参数) 十六进制数字表示的光照颜色。缺省值为`0xffffff`。
* `intensity`：(可选参数) 光源强度／亮度。缺省值为 1。
* `width`：(可选参数) 光源宽度。缺省值为 10。
* `height`：(可选参数) 光源高度。缺省值为 10。

```js
var areaLight = new THREE.RectAreaLight(0xff0000, 3)
areaLight.position.set(-10, 10, -35)
areaLight.rotation.set(-Math.PI / 2, 0, 0)
areaLight.width = 4
areaLight.height = 9.9
scene.add(areaLight)
```
