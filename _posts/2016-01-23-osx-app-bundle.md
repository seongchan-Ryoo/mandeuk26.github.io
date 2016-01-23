---
layout: post
title: Osx app bundle resource path
tags: [CPP, Notes]
---

## 简介

用于OSX的.app程序中得到Resources文件夹的绝对路径，来正确载入资源文件

只需链接CoreFoundation即可，代码来自SFML论坛<sup>[[0]](#ref)</sup>

## 代码

{% highlight cpp linenos %}
#if defined(__APPLE__)
#include <CoreFoundation/CoreFoundation.h>
#endif

std::string ResourcePath()
{
#if defined(__APPLE__)
	  CFBundleRef mainBundle = CFBundleGetMainBundle();
    CFURLRef resourcesURL = CFBundleCopyResourcesDirectoryURL(mainBundle);
    char path[512];
    if (!CFURLGetFileSystemRepresentation(resourcesURL, TRUE, (UInt8*)path, 512))
    {
        // error!
        std::cout << "Path not found!" << std::endl;
    }
    CFRelease(resourcesURL);
    return std::string(path);
#else 
    return std::string();
#endif
}
{% endhighlight %}

## 附录<span id="ref"></span>

[0] SFML forum post [Load resources in app bundle](http://en.sfml-dev.org/forums/index.php?topic=19015.msg137109#msg137109).

[1] Apple reference [Accessing a Bundle’s Contents](https://developer.apple.com/library/mac/documentation/CoreFoundation/Conceptual/CFBundles/AccessingaBundlesContents/AccessingaBundlesContents.html)
