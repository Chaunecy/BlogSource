---
title: POI 标注随场景同步缩放
date: 2018-07-29 14:17:24
categories:
tags:
	- three.js
---

在对地图进行缩放时，POI、Marker 也要同步缩放，否则会影响用户体验（失真、文字过小等）。

## 使用three.js 的做法

``` javascript
        function getScale(size) {
            var distance = camera.position.distanceTo(controls.target);
            var DEG2RAD = 180.0 / Math.PI;
            var top = Math.tan(camera.fov / 2 * DEG2RAD) * distance;
            var meterPerPixel = 2 * top / window.innerHeight;
            var scaleX, scaleY;
            
            scaleX = Math.max(meterPerPixel / 5, 1);
            return [scaleX, scaleX, 1.0];
        }
```

<!-- more -->

## 出处

[地址](https://www.cnblogs.com/dojo-lzz/p/7143276.html)

``` javascript
function getPoiScale(position, poiRect) {
    // 设置 scale 的基本思路是计算出 3d 中的长度单位与屏幕单位的
    // 比值，然后将 canvas 的像素宽高映射到 3d 世界中的长度。
    let pos = position ? position : this.controls.target;
    let distance = this.camera.position.distanceTo(pos);
    let top = Math.tan(this.camera.fov / 2 * DEG2RAD)  * distance;
    let meterPerPixel = 2 * top / window.innerHeight;
    let scaleX = poiRect.w * meterPerPixel;
    let scaleY = poiRect.h * meterPerPixel;
    return [scaleX, scaleY, 1.0];
}
```

