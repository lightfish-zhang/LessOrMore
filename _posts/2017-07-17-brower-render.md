---
layout: post
title: 简述浏览器如何渲染
date:   2017-7-17 12:30:00 +0800
categories: brower 
tag: [brower]
---

* content
{:toc}
 
## 前言

在`youtude`上学习了[Browser Rendering Optimization][1]的课程，做学习笔记整理

## 渲染的流程概括

- prase HTML 解析富文本
- recalculate style (DOM + CSS = render tree) 将DOM和CSS合成实际显示的树状节点
- multi surfaces, layouts and composite 合成图层
- raster, draw bitmap 矢量转位图，光栅器生成像素点

## 详细流程

### prase HTML

![brower-render-prase-html](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-prase-html.png)



### recalculate style

![brower-render-dom-css-recalculate-style](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-dom-css-recalculate-style.png)

![brower-render-dom-css-to-renderTree](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-dom-css-to-renderTree.png)

![brower-render-renderTree-only-display](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-renderTree-only-display.png)

![brower-render-renderTree-only-display2](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-renderTree-only-display2.png)

![brower-render-tree](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-tree.png)

### layouts and composite

![brower-render-layout](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-layout.png)


![brower-render-layout-composite](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-layout-composite.png)


### raster, draw bitmap

#### for page

![brower-render-raster-paint](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-raster-paint.png)

![brower-render-raster-paint-complete](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-raster-paint-complete.png)

![brower-render-raster](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-raster.png)

#### for image

![brower-render-raster-image-draw-bitmap](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-raster-image-draw-bitmap.png)

![brower-render-raster-image-decode-resize](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-raster-image-decode-resize.png)

![brower-render-layout-upload-gpu](https://raw.githubusercontent.com/lightfish-zhang/media-library/master/image/201707/brower-render-layout-upload-gpu.png)

## 附录

[1]:https://www.youtube.com/watch?v=hHvPD9m6ovM&index=2&list=PLAwxTw4SYaPl09X4Rljhy7dZinRCzbHz6