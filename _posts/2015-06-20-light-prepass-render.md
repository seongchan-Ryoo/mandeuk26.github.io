---
layout: post
title: Light Pre-pass Render
tags: [Note, Graphics]
---

## 光照方程

常用的就是Phong与Blinn-Phong了，先复习一下其原理：

### Phong模型

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

- N与L的点积，控制漫反射分量
- R与V的点积的n次方，控制镜面反射分量，n代表shininess

![phong model](/public/content/2015-06-20/phong_model.png)

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
