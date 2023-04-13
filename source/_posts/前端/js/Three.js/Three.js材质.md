


# 材质(Material)
材质的抽象基类。材质描述了对象的外观。它们的定义方式与渲染器无关。

所有其他材质类型都继承了以下属性和方法（尽管它们可能具有不同的默认值）。
## 构造函数(Constructor)
```js
Material()
```
该方法创建一个通用材质。
## 属性(Properties)
| 属性 | 参数类型 | 描述 |
| :--: | :--: | :--: |
| id | Integer | 此材质实例的唯一编号。 |
| name | String | 对象的可选名称（不必是唯一的）。默认值为空字符串。 |
| opacity | Float | 表明材质的透明度。值0.0表示完全透明，1.0表示完全不透明。如果材质的`transparent`属性未设置为`true`，则材质将保持完全不透明，此值仅影响其颜色。默认值为1.0。 |
| transparent | Boolean | 定义此材质是否透明。这对渲染有影响，因为透明对象需要特殊处理，并在非透明对象之后渲染。设置为true时，通过设置材质的opacity属性来控制材质透明的程度。默认值为false。 |
| visible | Boolean | 此材质是否可见。默认为true。 |
| side | Integer | 定义将要渲染哪一面，正面，背面或两者。默认为THREE.FrontSide。其他选项有THREE.BackSide和THREE.DoubleSide。 |
| needsUpdate | Boolean | 指定需要重新编译材质。如果为true，则会使用新的材质属性刷新它的缓存 |
| blending | 决定物体上的材质如何跟背景融合，设置为CustomBlending才能使用自定义blendSrc, blendDst，默认值为NormalBlending。 |
| blendSrc | 指定物体如何跟背景融合，默认值为SrcAlphaFactor。 |
| blendDst | 定义混合时如何使用背景，默认值为OneMinusSrcAlphaFactor。 |
| blendEquation | 使用混合时所采用的混合方程式。默认值为AddEquation，即将两个颜色值相加。 |
| deptTest | 是否在渲染此材质时启用深度测试。默认为 true。 |

# 常用材质

| 名称 | 描述 |
| :--: | :--: |
| MeshBasicMaterial | 基础网格材质，可以用它赋予几何体一种简单的颜色，或显示几何体的线框 |
| MeshDepthMaterial | 深度网格材质，根据网格到相机的距离决定如何给网格染色 |
| MeshNormalMaterial | 法线网格材质，一种把法向量映射到RGB颜色的材质。 |
| MeshLambertMaterial | Lambert网格材质 |
| MeshPhongMaterial | Phong网格材质，一种用于具有镜面高光的光泽表面的材质。 |
| MeshPhysicalMaterial | 物理网格材质 |
| LineBasicMaterial | 基础线条材质，一种用于绘制线框样式几何体的材质。 |
| LineDashedMaterial | 虚线材质，一种用于绘制虚线样式几何体的材质。|

## 基础网格材质(MeshBasicMaterial)
一个以简单着色（平面或线框）方式来绘制几何体的材质。

这种材质不受光照的影响。
```js
MeshBasicMaterial( parameters: Object )
```
#### 属性(Properties)

| 名称 | 数据类型 | 描述 |
| :--: | :--: | :--: |
| color | Color | 材质的颜色(Color)，默认值为白色 (0xffffff)。 |
| wireframe | Boolean | 将几何体渲染为线框。默认值为false（即渲染为平面多边形）。 |
| wireframeLinecap | String | 定义线两端的外观。可选值为 'butt'，'round' 和 'square'。默认为'round'。 |
| wireframeLinejoin | String | 定义线连接节点的样式。可选值为 'round', 'bevel' 和 'miter'。默认值为 'round'。 |
| wireframeLinewidth | Float | 控制线框宽度。默认值为1。 |
| fog | Boolean | 材质是否受雾影响。默认为true。 |

```js
var meshMaterial = new THREE.MeshBasicMaterial({color: 0x7777ff});
```
## 深度网格材质(MeshDepthMaterial)
一种按深度绘制几何体的材质。深度基于相机远近平面。白色最近，黑色最远。
```js
MeshDepthMaterial( parameters: Object )
```

| 名称 | 数据类型 | 描述 |
| :--: | :--: | :--: |
| wireframe | Boolean | 将几何体渲染为线框。默认值为false（即渲染为平面多边形）。 |
| wireframeLinewidth | Float | 控制线框宽度。默认值为1。 |

## Lambert网格材质(MeshLambertMaterial)
一种非光泽表面的材质，没有镜面高光。

该材质使用基于非物理的Lambertian模型来计算反射率。 这可以很好地模拟一些表面（例如未经处理的木材或石材），但不能模拟具有镜面高光的光泽表面（例如涂漆木材）。
```js
MeshLambertMaterial( parameters : Object )
```

| 名称 | 数据类型 | 描述 |
| :--: | :--: | :--: |
| color | Color | 材质的颜色(Color)，默认值为白色 (0xffffff)。 |
| wireframe | Boolean | 将几何体渲染为线框。默认值为false（即渲染为平面多边形）。 |
| wireframeLinecap | String | 定义线两端的外观。可选值为 'butt'，'round' 和 'square'。默认为'round'。 |
| wireframeLinejoin | String | 定义线连接节点的样式。可选值为 'round', 'bevel' 和 'miter'。默认值为 'round'。 |
| wireframeLinewidth | Float | 控制线框宽度。默认值为1。 |
| fog | Boolean | 材质是否受雾影响。默认为true。 |
| emissive | Color | 材质的放射（光）颜色，基本上是不受其他光照影响的固有颜色。默认为黑色。 |
| aoMapIntensity | Float | 环境遮挡效果的强度。默认值为1。零是不遮挡效果。 |

```js
var meshMaterial = new THREE.MeshLambertMaterial({color: 0x7777ff})
```
## 法线网格材质(MeshNormalMaterial)
