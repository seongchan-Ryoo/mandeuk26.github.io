---
layout: post
title: Simple Gradiant Script For Unity
tags: [Unity, CSharp]
---

## 简介

用于Unity的UI的简易渐变效果脚本

支持渐变方向选择，支持单独设置Alpha

{% highlight csharp linenos %}
using UnityEngine;
using UnityEngine.UI;
using System.Collections;
using System.Collections.Generic;

public class Gradiant : BaseVertexEffect
{
    public Color32 from = Color.black;

    public Color32 to = Color.white;

    public bool linearAlpha = true;

    public byte alpha = 255;

    public enum Type { Horizontal, Vertical };

    public Type type = Type.Vertical;

    public override void ModifyVertices(List<UIVertex> vertexList)
    {
        if (!IsActive() && vertexList.Count < 1)
            return;

        bool vertical = type == Type.Vertical;

        float start = vertical ? vertexList[0].position.y : vertexList[0].position.x;
        float end = start;

        foreach (var vertex in vertexList)
        {
            float p = vertical ? vertex.position.y : vertex.position.x;
            if (p > start)
                start = p;
            else if (p < end)
                end = p;
        }

        float size = start - end;

        for(int i = 0; i < vertexList.Count; i++)
        {
            var vertex = vertexList[i];

            float p = vertical ? vertex.position.y : vertex.position.x;
            vertex.color = Color32.Lerp(from, to, (p - end) / size);

            if (!linearAlpha)
                vertex.color.a = alpha;

            vertexList[i] = vertex;
        }
    }
}
{% endhighlight %}
