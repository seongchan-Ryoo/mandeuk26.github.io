---
layout: post
title: Perfect Forwarding And Piecewise Construct
tags: [Notes, CPP]
---

## 简介

完美转发与分段构造用例

```cpp
#include <map>
#include <functional>
#include <typeinfo>
#include <typeindex>
#include <iostream>

class MyClass
{
public:
    int a;
	int b;

	MyClass(int a, int b) : a(a), b(b) {}
};

std::map<std::type_index, MyClass> myMap;

template<class T, typename... Args>
T &Test(Args&&... args)
{
	auto it = myMap.emplace(std::forward<Args>(args)...);
	return it.first->second;
}

int main()
{
	// g++ 4.8+
	auto ref = Test<MyClass>(
		std::piecewise_construct, 
		std::forward_as_tuple<std::type_index>(typeid(MyClass)), 
		std::forward_as_tuple<int, int>(10, 100)
	);

	std::cout<<ref.a<<", "<<ref.b<<std::endl;

	return 0;
}
```

[运行](http://coliru.stacked-crooked.com/a/751a932dd2b5cb1a)

## 参考

- <http://stackoverflow.com/questions/14075128/mapemplace-with-a-custom-value-type>
- <http://cpptruths.blogspot.com/2012/06/perfect-forwarding-of-parameter-groups.html>
- <http://www.cplusplus.com/reference/map/map/emplace/>
- <http://www.cplusplus.com/reference/utility/forward/>
