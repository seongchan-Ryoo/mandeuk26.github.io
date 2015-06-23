---
layout: post
title: Light Pre-pass Render
tags: [Notes, Graphics]
---

## 简介

Light Pre-pass 渲染器 <sup>[[0]](#ref)</sup> 是延迟渲染的一种修改版。
渲染步骤大致如下：

- 渲染场景法线与深度到MRT中
- 计算Light Buffer
- 从Light Buffer中获得光照参数，重建着色方程
- 进行Forward Rendering，排序由前至后

此方法的优点有：

- 更小的G-Buffer
- 支持MSAA
- 支持没有MRT的设备

## 光照方程

常用的就是Phong <sup>[[1]](#ref)</sup> 与Blinn-Phong <sup>[[2]](#ref)</sup> 了，先复习一下其原理：

![phong](/public/content/2015-06-20/phong.png)

<table>
 	<thead>
		<tr>
			<th>Name</th>
			<th>Detail</th>
			<th>Name</th>
			<th>Detail</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>N</td>
			<td>平面法线</td>
			<td>L</td>
			<td>光的方向</td>
		</tr>
		<tr>
			<td>V</td>
			<td>视角方向</td>
			<td>R</td>
			<td>L对N镜像</td>
		</tr>
		<tr>
			<td>H</td>
			<td>L + V</td>
			<td></td>
			<td></td>
		</tr>
	</tbody>
</table>

### Phong模型

![phong model](/public/content/2015-06-20/phong_model.png)

- N与L的点积，控制漫反射分量
- R与V的点积的n次方，控制镜面反射分量，n代表shininess

<table>
 	<thead>
		<tr>
			<th>Name</th>
			<th>Detail</th>
			<th>Name</th>
			<th>Detail</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>K</td>
			<td>Color</td>
			<td>Att</td>
			<td>Attenuation</td>
		</tr>
		<tr>
			<td>a</td>
			<td>Ambient</td>
			<td>s</td>
			<td>Specular</td>
		</tr>
		<tr>
			<td>d</td>
			<td>diffuse</td>
			<td>n</td>
			<td>shininess</td>
		</tr>
		<tr>
			<td>IT</td>
			<td>Intensity</td>
			<td>l</td>
			<td>Light</td>
		</tr>
	</tbody>
</table>

L对N的镜像R的计算：

![reflect](/public/content/2015-06-20/reflect.png)

### Blinn-Phong模型

![blinn phong model](/public/content/2015-06-20/blinn_phong_model.png)

- N与L的点积，控制漫反射分量
- N与H的点积的n次方，控制镜面反射分量，n代表shininess

## 渲染器设计

属于灯光的参数：

```
light
{
	color, 

	shininess/power
}
```

属于材质的参数：

```
material 
{
	specular_color, 
	specular_intensity, 

	diffuse_color, 
	diffuse_intensity,

	shininess/power
}
```

我们需要重建的光照方程，使用Blinn-Phong光照模型：

```
color = ambient + shadow * attenuation * (
	mat_diff * diff_intensity * light_color * N * L + 
	mat_spec * spec_intensity * ((N * H)^n)^m
)
```

<table>
 	<thead>
		<tr>
			<th>Name</th>
			<th>Detail</th>
			<th>Name</th>
			<th>Detail</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>attenuation</td>
			<td>控制灯光的衰减</td>
			<td>intensity</td>
			<td>控制光照的强度</td>
		</tr>
		<tr>
			<td>n</td>
			<td>灯光的亮度</td>
			<td>m</td>
			<td>材质的亮度</td>
		</tr>
	</tbody>
</table>

光照方程中，与灯光相关参数：

<table>
	<tbody>
		<tr>
			<td>N * L</td>
			<td>light.color</td>
			<td>(N * H) ^ n</td>
			<td>attenuation</td>
		</tr>
	</tbody>
</table>

我们将其存储到Light Buffer中，用于后续渲染中对光照方程的重建：

```
light_buffer.rgb 	= light_color.rgb * N * L * attenuation
light_buffer.a 		= (N * H)^n * N * L * attenuation
N * L * attenuation
```

若需要重建高光反射分量，则共需要两个Render Target。
在这里，我们可以转换 N * L * attenuation 到 luminance <sup>[[3]](#ref)</sup>，这两个值非常接近：

```
luminance 	= N * L * attenuation 
			= (0.2126 * R + 0.7152 * G + 0.0722 * B)
```

这样我们就可以重建高光反射分量，并且仅需要一个Render Target：

```
(N * H)^n 	= light_buffer.a / luminance
			= ((N * H)^n * N * L * attenuation) / (N * L * attenuation);
```

<span id="ref"></span>
## 附录

[0] Wolfgang Engel. [Light Pre-Pass Renderer](http://diaryofagraphicsprogrammer.blogspot.com/2008/03/light-pre-pass-renderer.html). Diary of a Graphics Programmer.

[1] Wikipedia. [Phong Reflection Model](https://en.wikipedia.org/wiki/Phong_reflection_model).

[2] Wikipedia. [Blinn Phong Shading Model](https://en.wikipedia.org/wiki/Blinn%E2%80%93Phong_shading_model).

[3] Wolfgang Engel. [Light Pre-Pass More Blood](http://diaryofagraphicsprogrammer.blogspot.com/2008/09/light-pre-pass-more-blood.html). Diary of a Graphics Programmer.
