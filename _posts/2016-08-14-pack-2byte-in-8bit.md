---
layout: post
title: Pack 2Byte In 8Bit
tags: [Unity, CSharp, Graphics]
---

## 前言

现在的手机游戏界面越来越复杂，在unity引擎下，通常通过打图集来减少drawcall，增加batch，移动平台对于内存／外存的要求都比较苛刻，所以如何减少图集的内存／外存是优化的重点之一。

最常用的方法就是使用贴图压缩算法对图集进行压缩，内外存都会有比较可观的减少。由于为了兼容性和更高的压缩比，大家普遍选择ETC1压缩算法，但其无法支持带Alpha通道的贴图压缩，而通常的做法是将RGB和Alpha打成两张图集。

这种方法目前为止看起来也没有太大的优化空间了，不过，看到kaimingyi同学的[分享](http://gameknife.github.io/tech/2016/04/25/UITexCompress/)，继续优化的可能性还是有的。其通过使用延迟渲染中压缩GBuffer的Albedo数据的方法，即将RGB颜色转换到YUV颜色空间（文中使用其变种[YCbCr](https://en.wikipedia.org/wiki/YCbCr)），因为在此表示方式下亮度通道的重要性比较高，而其他两个色度分量的重要性比较低，可通过牺牲色度分量的精度，将这两个分量Pack到8bit中：

```
亮度8色度8透明度8
```

即可实现通过一张图集，来支持Alpha通道的ETC1压缩解决方案。

## 实现

可在生成AssetBundle的时候，对资源图片进行处理，将其转换颜色空间：

```csharp
public static byte Pack(byte a, byte b)
{
    // scale from 0-255 -> 0-15
    int ia = (int)a / 17;
    int ib = (int)b / 17;

    // pack scaled value to c
    byte c = (byte)(ia << 4);
    c = (byte)((int)c | ib & 15);
    return c;
}

public static void UnPack(byte c, out byte a, out byte b)
{
    // unpack value from c
    int ic = c;
    a = (byte)(ic >> 4);
    b = (byte)(int)((byte)(ic << 4) >> 4);

    // scale from 0-15 -> 0-255
    a = (byte)((int)a * 17);
    b = (byte)((int)b * 17);
}
```

在渲染时的shader里[还原](http://www.cnblogs.com/crazybingo/archive/2012/06/08/2541700.html)颜色到RGB空间，即可使用。

## 截图

待续 … 

## 总结

通过此方法，仅使用一张图集就可支持带Alpha通道的ETC1贴图压缩。

优点

* 进一步减少了内外存的占用
* 减少了FragmentShader里贴图采样的带宽占用

缺点

* FragmentShader里增加了颜色空间转换的计算
* 图像质量的问题

综上所述，在图像质量可以接受的情况下，此方法还是值得一试的。最后，感谢kaimingyi同学的脑洞 XD