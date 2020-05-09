### 问题
网页缩小

### 为什么缩放了
查看webview组件的源码，发现了**updateMatrix**函数，这是个刷新操作，放在update中的，
首先，先拿到设备或浏览器像素比例,就是**devicePixelRatio**，还有缩放比**scaleX**、**scaleY**
去执行下面一句
 
 ``` java
scaleX /= dpr; 
 scaleY /= dpr; 
 var a = _mat4_temp.m00 * scaleX;
 var d = _mat4_temp.m05 * scaleY;
 var matrix = "matrix(" + a + "," + -b + "," + -c + "," + d + "," + tx + "," + -ty + ")";
```

得出来的a会被放在**transform:matrix**中,这是一个css3中的矩阵，使用矩阵对图像进行变换。

问题就在这里了，

> **引擎会根据设备的像素比例去缩放webview的网页大小**


### 怎么修改
我们得知网页的缩放是放在**transform:matrix**中，了解这个属性的人就知道要修改这里的**a,d**值，这是控制网页的缩放，**tx，和ty**是控制网页的偏移，不了解的可以去百度下这个属性。

接下来我们修改
``` java
a = 1
d = 1
```

缩放值修改了，我们计算下从原来的值缩放到1是放大了多少倍

``` java
var selfScale = 1/a;
```

修改webview的大小

``` java
this._w = node._contentSize.width/selfScale;
this._h = node._contentSize.height/selfScale;
this._updateSize(this._w, this._h);
```

> 每次刷新会重新绘制大小


这样修改视图大小会改为我们想要的大小（不进行缩放），但是组件的偏移会出现问题，继续修改。


发现了偏移量的计算中有一个计算会乘以缩放倍数，我们修改自己的缩放倍数
``` java
var w = this._div.clientWidth * 1;
var h = this._div.clientHeight * 1;
```

> 运行，完美显示


>PS. 完整代码（此代码修改在2.1.4中的编译后的cocos2d-js.js中）

 ``` java
//修改webview组件展示网页的缩放倍数
var scaleNum = 1;//缩放倍数（任意修改）
var selfScale = scaleNum / a;
this._w = node._contentSize.width / selfScale;
this._h = node._contentSize.height / selfScale;

var offsetX = container && container.style.paddingLeft ? parseInt(container.style.paddingLeft) : 0;

var offsetY = container && container.style.paddingBottom ? parseInt(container.style.paddingBottom) : 0;

this._updateSize(this._w, this._h);
var w = this._div.clientWidth * scaleNum;
var h = this._div.clientHeight * scaleNum;
var appx = w * _mat4_temp.m00 * node._anchorPoint.x;
var appy = h * _mat4_temp.m05 * node._anchorPoint.y;
var viewport = cc.view._viewportRect;
offsetX += viewport.x / dpr;
offsetY += viewport.y / dpr;
var tx = _mat4_temp.m12 * scaleX - appx + offsetX, ty = _mat4_temp.m13 * scaleY - appy + offsetY;

var matrix = "matrix(" + scaleNum + "," + -b + "," + -c + "," + scaleNum + "," + tx + "," + -ty + ")";
//修改代码结束
```

### 最后

无图无真相


![03a5c34c32a5dea002ae5f154396b7dc.png](https://s1.ax1x.com/2020/05/09/YQYD1K.png)
> 0.5倍缩放


![127f382d6acffa8d566e94ba7b2358d2.png](https://s1.ax1x.com/2020/05/09/YQYr6O.png)
> 1倍缩放

![b523234de501ef16e03ae72766e756f3.png](https://s1.ax1x.com/2020/05/09/YQYsXD.png)
> 2倍缩放


放上完整demo [cocos_webview](https://github.com/keien411/cocos_webview)


> 修改后的的js放在项目路径下的js文件夹中


 