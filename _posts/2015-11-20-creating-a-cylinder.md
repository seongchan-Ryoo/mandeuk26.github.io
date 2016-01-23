---
layout: post
title: Creating a Cylinder
tags: [CPP, Graphics]
---

## 简介

使用代码生成圆柱体，可用来表示Spot Light的凸包围体

当圆柱体的某一端闭合时，会产生重复顶点。
可参考Assimp<sup>[[0]](#ref)</sup>的做法，来剔除潜在重复顶点。

## 实现

{% highlight cpp linenos %}
void CreateCylinder(const std::string &name, float topR, float bottomR, float height, int segH, int segV)
{
    if (segH < 2 || segV < 3)
    {
        LOGW << "SegH or SegV tooooo small!";
        return;
    }
    
    // output result
    std::vector<float> vertices;
    std::vector<unsigned int> indices;

    // allocate some space.
    vertices.reserve((segV * segH + 2) * 3);
    indices.reserve((segV * segH * 2) * 3);

    float avgRadian = PI * 2.0f / segV;

    float currentHeight = height / 2.0f;
    float currentRadius = topR;
    
    float heightStep = height / (segH - 1);
    float radiusSetp = (topR - bottomR) / (segH - 1);

    std::vector<unsigned int> previousRing;
    previousRing.reserve(segV * 3);

    std::vector<unsigned int> currentRing;
    currentRing.reserve(segV * 3);

    unsigned int index = 0;
    auto AddVertex = [&currentRing, &vertices, &index](float x, float y, float z) -> unsigned int
    {
        vertices.insert(vertices.end(), { x, y, z });
        return index++;
    };

    for (int h = 0; h < segH; h++)
    {
        // fill current ring
        for (int v = 0; v < segV; v++)
        {
            float radian = avgRadian * v;
            currentRing.push_back(AddVertex(
                cos(radian) * currentRadius, currentHeight, sin(radian) * currentRadius));
        }

        if (previousRing.size() > 0)
        {
            // has previous ring, we connect them to triangles.
            for (unsigned int i = 0; i < currentRing.size() - 1; i++)
            {
                indices.insert(indices.end(), {
                    previousRing[i], previousRing[i + 1], currentRing[i + 1], 
                    currentRing[i + 1], currentRing[i], previousRing[i]
                });
            }
            indices.insert(indices.end(), {
                previousRing.back(), previousRing.front(), currentRing.front(), 
                currentRing.front(), currentRing.back(), previousRing.back()
            });
        }
        else
        {
            // don't have previous ring, then we're on top.
            // close this ring.
            unsigned int center = AddVertex(0, currentHeight, 0);
            for (unsigned int i = 0; i < currentRing.size() - 1; i++)
            {
                indices.insert(indices.end(), { center, currentRing[i + 1], currentRing[i] });
            }
            indices.insert(indices.end(), { center, currentRing.front(), currentRing.back() });
        }

        previousRing = currentRing;
        // this way the memory allocated earlier won't be destoried.
        currentRing.erase(currentRing.begin(), currentRing.end());
        currentHeight -= heightStep;
        currentRadius -= radiusSetp;
    }

    // close bottom ring
    unsigned int center = AddVertex(0, -height / 2.0f, 0);
    for (unsigned int i = 0; i < previousRing.size() - 1; i++)
    {
        indices.insert(indices.end(), { center, previousRing[i], previousRing[i + 1] });
    }
    indices.insert(indices.end(), { center, previousRing.back(), previousRing.front() });
}
{% endhighlight %}

## 附录<span id="ref"></span>

[0] Assimp. [JoinVerticesProcess.cpp](https://github.com/assimp/assimp/blob/master/code/JoinVerticesProcess.cpp).

[1] Andreaskahler. [Creating a icosphere](http://blog.andreaskahler.com/2009/06/creating-icosphere-mesh-in-code.html).
