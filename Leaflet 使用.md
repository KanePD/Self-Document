# Leaflet 使用

最近在Angular项目中，用到了地图，由于种种原因放弃了百度地图api使用，最后选择了leaflet，简单介绍一下。

## 介绍：

Leaflet 是一个为移动设备设计的交互式地图的开源的 javascript库， 并只有38k，包含了大多数开发者需要的地图特点。

## 准备：下载 leaflet 文件

访问： [Leaflet下载官网](https://leafletjs.com/download.html)

## 在单一的HTML页面中使用Leaflet

- 创建一个文件夹 leaflet_test
- 文件夹下创建一个index.html
- 将上述下载的leaflet文件放到leaflet_test文件夹下
- 在index.html插入如下代码

```hmtl
<!-- 引入 文件 -->
<link rel="stylesheet" href="./leaflet.css" />
<script src="./leaflet.js"> </script>
<!-- 增加地图高度 -->
<style>
#mapDiv { height: 300px; }
</style>
<!-- 创建一个 地图的div id 必须有 但是自定义 -->
<div id="mapDiv"></div>
<script>
		//到 mapbox 官网注册并创建下面的access token都是免费的，不过有5w次的浏览限制
		var url = 'https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token=pk.eyJ1Ijoia2FuZXdhbmciLCJhIjoiY2pwM2UxNHNkMGF1MzNwc2FtMnNhdXJsMCJ9.KZpCBtizDeltZO6JhGc6_w';
		//初始化 地图
		var leafletMap = L.map('mapDiv').setView([41, 123], 5);
		//将图层加载到地图上，并设置最大的聚焦还有map样式
		L.tileLayer(url, {
				maxZoom: 18,
				id: 'mapbox.streets'
		}).addTo(leafletMap);
		//增加一个marker ，地图上的标记，并绑定了一个popup，默认打开
		L.marker([41, 123]).addTo(leafletMap)
				.bindPopup("<b>Hello world!</b><br />I am a popup.").openPopup();
		//增加一个圈，设置圆心、半径、样式
		L.circle([41, 123], 500, {
				color: 'red',
				fillColor: '#f03',
				fillOpacity: 0.5
		}).addTo(leafletMap).bindPopup("I am a circle.");
		//增加多边形
		L.polygon([
				[41, 123],
				[39, 121],
				[41, 126]
		]).addTo(leafletMap).bindPopup("I am a polygon.");
		//为点击地图的事件 增加popup
		var popup = L.popup();
		function onMapClick(e) {
				popup
						.setLatLng(e.latlng)
						.setContent("You clicked the map at " + e.latlng.toString())
						.openOn(leafletMap);
		}
		leafletMap.on('click', onMapClick);
</script>
```

- 上述代码可直接使用，下面先上效果图，再解释代码的含义

![1543556507296](C:\Users\kanewang\AppData\Local\Temp\1543556507296.png)

- 代码解释：
  - 1.引入Leaflet css 与js 文件，官网要求，css在前 js在后。不过我简单试过没啥变化。
  - 2.增加 地图css样式，设置高度，这个官网要求必须的。***不设置会不显示地图* **
  - 创建装地图的div 记住这个div的 id
  - 开始创建地图，我们先去准备一个 [瓦片图层](https://baike.baidu.com/item/%E7%93%A6%E7%89%87%E5%9C%B0%E5%9B%BE/8006049?fr=aladdin) 本文使用的是 [mapbox](https://www.mapbox.com)
  - 到 map box上注册账号，登录后 创建一个access token，copy到代码url accesstoken后面，使用我的也好使
  - 初始化map 使用 L.map('mapDiv').setView([51.505, -0.09], 13),其中[51.505, -0.09]地理位置，13是变焦的大小
  - 将图层加到map上。其中url是图层的资源的url，maxZoom 是最大的聚焦【mapbox官网最大的也是18了】，id 是 地图的样式并不是我们平常认识的id，本文选择了 street 的map
  - mapbox 支持的地图样式,共6个默认的（默认提供的，可以自己创建上传）
    - Mapbox Incidents V1 [id=mapbox.mapbox-incidents-v1]
    - Mapbox Statellite[mapbox.satellite]
    -  ...
  - 加标记、加圆圈、加多边形，再为地图每个位置点击增加事件
- 至此，简单的应用就完成了。

## 在单一页面中使用中国各种地图

为什么单独说，因为leaflet提供了一个插件 [Leaflet.ChineseTmsProviders](https://github.com/htoooth/Leaflet.ChineseTmsProviders)可以访问 [github](https://github.com/htoooth/Leaflet.ChineseTmsProviders/tree/master/src)主页查看。下面我们看看如何使用它。

- 准备去github上下载插件的js文件 [github下载](https://github.com/htoooth/Leaflet.ChineseTmsProviders/tree/master/src)，下载后同样放到与leaflet包同一路径下
- 上一下插件代码：

```javascript
L.TileLayer.ChinaProvider = L.TileLayer.extend({

    initialize: function(type, options) { // (type, Object)
        var providers = L.TileLayer.ChinaProvider.providers;

        var parts = type.split('.');

        var providerName = parts[0];
        var mapName = parts[1];
        var mapType = parts[2];

        var url = providers[providerName][mapName][mapType];
        options.subdomains = providers[providerName].Subdomains;

        L.TileLayer.prototype.initialize.call(this, url, options);
    }
});

L.TileLayer.ChinaProvider.providers = {
    TianDiTu: {
        Normal: {
            Map: "http://t{s}.tianditu.cn/DataServer?T=vec_w&X={x}&Y={y}&L={z}",
            Annotion: "http://t{s}.tianditu.cn/DataServer?T=cva_w&X={x}&Y={y}&L={z}"
        },
        Satellite: {
            Map: "http://t{s}.tianditu.cn/DataServer?T=img_w&X={x}&Y={y}&L={z}",
            Annotion: "http://t{s}.tianditu.cn/DataServer?T=cia_w&X={x}&Y={y}&L={z}"
        },
        Terrain: {
            Map: "http://t{s}.tianditu.cn/DataServer?T=ter_w&X={x}&Y={y}&L={z}",
            Annotion: "http://t{s}.tianditu.cn/DataServer?T=cta_w&X={x}&Y={y}&L={z}"
        },
        Subdomains: ['0', '1', '2', '3', '4', '5', '6', '7']
    },

    GaoDe: {
        Normal: {
            Map: 'http://webrd0{s}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=8&x={x}&y={y}&z={z}'
        },
        Satellite: {
            Map: 'http://webst0{s}.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}',
            Annotion: 'http://webst0{s}.is.autonavi.com/appmaptile?style=8&x={x}&y={y}&z={z}'
        },
        Subdomains: ["1", "2", "3", "4"]
    },

    Google: {
        Normal: {
            Map: "http://www.google.cn/maps/vt?lyrs=m@189&gl=cn&x={x}&y={y}&z={z}"
        },
        Satellite: {
            Map: "http://www.google.cn/maps/vt?lyrs=s@189&gl=cn&x={x}&y={y}&z={z}"
        },
        Subdomains: []
    },

    Geoq: {
        Normal: {
            Map: "http://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineCommunity/MapServer/tile/{z}/{y}/{x}",
            Color: "http://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineStreetColor/MapServer/tile/{z}/{y}/{x}",
            PurplishBlue: "http://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineStreetPurplishBlue/MapServer/tile/{z}/{y}/{x}",
            Gray: "http://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineStreetGray/MapServer/tile/{z}/{y}/{x}",
            Warm: "http://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineStreetWarm/MapServer/tile/{z}/{y}/{x}",
            Cold: "http://map.geoq.cn/ArcGIS/rest/services/ChinaOnlineStreetCold/MapServer/tile/{z}/{y}/{x}"
        },
        Subdomains: []

    }
};

L.tileLayer.chinaProvider = function(type, options) {
    return new L.TileLayer.ChinaProvider(type, options);
};

```

- 更改index.html文件

```javascript
<script src="./leaflet.js"></script>
<link rel="stylesheet" href="./leaflet.css" />
<script src="leaflet.ChineseTmsProviders.js"></script>

<style>
//#mapDiv { height: 300px; }
.test { height: 300px; }
</style>
<div id="mapDiv" class='test'></div>
<script>
    //插件把 定义了多个国内的瓦片图层，我们只需要通过提供的方法访问到相应的图层即可
    //从插件代码可以看出 需要传入 providerName.mapName.mapType 从插件代码中查找所需要的值
    var test = L.tileLayer.chinaProvider('Geoq.Normal.Map', {
        maxZoom: 18,
        minZoom: 5
    });
	//此处可以定义多个图层，并可以再页面中进行选择
	var baseLayers = {"测试地图":test}
    
    var map = L.map("mapDiv", {
        center: [41.80, 123.43],
        zoom: 7,
        layers: [test],
        zoomControl: false
    });
	L.control.layers(baseLayers, null).addTo(map);
    L.control.zoom({
        zoomInTitle: '放大',
        zoomOutTitle: '缩小'
    }).addTo(map);
</script>
```

- 上结果图，这次只是使用插件没有其他功能。中心点是沈阳市。

![1543803973634](C:\Users\kanewang\AppData\Local\Temp\1543803973634.png)

## Leaflet 画线装饰插件

本次开发使用了另一个插件[polylineDecorator.js](https://github.com/bbecquet/Leaflet.PolylineDecorator)

- index.html

```javascript
//如下代码需要上一节的代码
	var arrow = L.polyline([[41.80, 123.43], [41.07, 123.00]], {opacity: 1,color: 'firebrick'}).bindPopup('I am red:').addTo(map);//
	var arrow2 = L.polyline([[41.80, 123.43], [40.13, 124.37]], {opacity: 1,color: 'lightgreen'}).bindPopup('I am green:').addTo(map);
	var arrow3 = L.polyline([[41.07, 123.00], [40.13, 124.37]], {opacity: 1,color: 'lightgreen'}).bindPopup('I am green:').addTo(map);
	
    var arrowHead = L.polylineDecorator(arrow, {
        patterns: [
            {offset: '30%' ,endOffset:'90%',repeat: 1000, symbol: L.Symbol.arrowHead({pixelSize: 10, polygon: false,pathOptions: {stroke: true,weight:2,color: 'firebrick'}})}
        ]
    }).addTo(map);
```

- 上图：在一条线上增加了一个箭头，还有很多的装饰，可访问github主页查看example 的代码

![1543817401324](C:\Users\kanewang\AppData\Local\Temp\1543817401324.png)

## 在 angular中使用 leaflet.js

由于leaflet是 javascript库，而angular 使用的typescript 语言，这就存在一个问题。经过查询，发现了ts的一个功能。.d.ts文件的使用。

**注：如下内容，仅做介绍了，在我们的项目中是成功的引入了leaflet了，如果不好使，请大家自行查询一下吧。推荐一个介绍文章  **[JavaScript 和 TypeScript 交叉口](https://www.cnblogs.com/silin6/p/7793753.html)

在项目的根目录下创建文件【angular框架中是 src下】 index.d.ts

文件内容：

```typescript
declare var L:any; //leaflet
```

当然在angular.json需要引入相关的 leaflet的js文件

```json
在projects->architect->build->options->scripts中加入leaflet.js的路径
```

相关的内容请查阅angular相关的介绍 [Angular Cli Stories](https://github.com/angular/angular-cli/wiki/stories)

