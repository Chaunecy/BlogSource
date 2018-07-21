---
title: three.js 使用 Sprite 加载纹理
date: 2018-07-14 15:38:42
categories:
	- 编程语言
tags:
	- JavaScript
	- three.js
	- WebGL
---

## Sprite

three.js 中的精灵是一个永远朝向相机的平面。

[例子](http://threejs-outsidelook.oss-cn-shanghai.aliyuncs.com/r89/source/examples/index.html?q=spr#webgl_sprites)

<!-- more -->

## 使用 Sprite 绘制文字

``` js
function createTextTextureBySprite() {
    let canvas = document.createElement('canvas');
    let ctx = canvas.getContext('2d');
    ctx.fillStyle = '#ffff00';
    ctx.font = 'Bold 50px 微软雅黑';
    ctx.lineWidth = 4;
    /*
    把整个 canvas 作为纹理，所以字尽量大一些，撑满整个 canvas 画布。
    但也要小心文字溢出画布。
    */
    ctx.fillText('Hello World!', 16, 16);
    let texture = new THREE.Texture(canvas);
    texture.needsUpdate = true;
    
    let material = new THREE.SpriteMaterial({
        map: texture,
        // transparent: true, // 避免遮挡其他图形
    });
    let textMesh = new THREE.Sprite(material);
    /*
    精灵很小，要放大
    */
    textMesh.scale.set(50.0, 25.0, 75.0);
    /*
    WebGL 3D 世界中的位置
    */
    textMesh.position.set(220, 500, 220);
    return textMesh;
}
```

