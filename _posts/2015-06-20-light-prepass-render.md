---
layout: post
title: Light Pre-pass Render
tags: [Note, Graphics, Render]
---

### 光照方程

常用的就是Phong与Blinn-Phong了，先复习一下其原理：

下图中N是平面法线，L是光的方向，V是视角方向，R是L对N的镜像，H是L加V得到的单位向量。

![phong](/public/content/2015-06-20/phong.png)

对于Phong模型：

- N与L的点积，控制漫反射分量
- R与V的点积的n次方，控制镜面反射分量（n代表shininess）

![phong model](/public/content/2015-06-20/phong_model.png)

公式中

L对N的镜像R的计算：

![reflect](/public/content/2015-06-20/reflect.png)

在Blinn-Phong模型中，通过替换 R*V 为 H=(L+V)/|L+V| 来加快计算过程。

### 实现



### 附录