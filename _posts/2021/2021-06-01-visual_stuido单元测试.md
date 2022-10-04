---
layout: post
title:  "visual studio 2019 单元测试"
date:   2021-06-01 15:03:36 +0800
categories: c++
typora-root-url: ..
---

# visual studio 2019 单元测试

leetcode 上的大神为了更好的性能和评分，解题目多喜欢用c++。为了能更好的看懂题解，方便刷题目，重新学了一下c++。

## 为什么需要有单元测试？

visual studio确实很强大，可是在写小的东西时候太重了，尤其是不允许有多个main方法。况且，很多题目就是让你考虑各种边界条件的，在控制台手动验证太繁琐了，通过单元测试能很好的验证结果。

Visual Studio 包含这些 C++ 测试框架，如果用上boost和Google test就更复杂了，这里用自带的Microsoft 单元测试框架。

- 适用于 C++ 的 Microsoft 单元测试框架
- Google Test
- Boost.Test
- CTest



参考链接：

[编写适用于 C/C++ 的单元测试 - Visual Studio | Microsoft Docs](https://docs.microsoft.com/zh-cn/visualstudio/test/writing-unit-tests-for-c-cpp?view=vs-2019)

如果还是没看明白，跟着别人的操作走一遍就可以了：

[VS2019 C++ 单元测试_Fight It-CSDN博客_vs2019单元测试](https://blog.csdn.net/huahua520amy/article/details/111559997)

## 快速排序

随手写一个快排：

```c++
#include "Sorting.h"
#include "iostream"

using namespace std;

//快速排序
void Sorting :: quickSort(int nums[],int length) {
	//sizeof(nums)/sizeof(nums[0]); 如果是指针就不能用了，传入length
	quick_sort(nums, 0, length-1);
}

void Sorting :: quick_sort(int nums[], int low, int high) {

	if (low < high) {
		int mid = partition(nums, low, high);
		quick_sort(nums, low, mid - 1);
		quick_sort(nums,  mid + 1, high);
	
	}
}

int Sorting::partition(int nums[], int low, int high) {
	int temp = nums[low];
	while (low < high) {
		while (low < high) {
			if (nums[high] > temp) {
				high--;
			}
			else {
				nums[low] = nums[high];
				break;
			}
		}
		while (low < high) {
			if (nums[low] < temp) {
				low++;
			}
			else {
				nums[high] = nums[low];
				break;
			}
		}
	}
	nums[low] = temp;
	return low;
}
```

## SortTest.cpp

这里RandUtils是为了方便测试编写一个工具类，用了一下CppUnitTestFramework的api:

参考：

[Microsoft.VisualStudio.TestTools.CppUnitTestFramework API - Visual Studio | Microsoft Docs](https://docs.microsoft.com/zh-cn/visualstudio/test/microsoft-visualstudio-testtools-cppunittestframework-api-reference?view=vs-2019#cppunittestloggerh)

```c++
#include "pch.h"
#include "CppUnitTest.h"
#include "../Algorithms/Sorting.h"
#include "../Algorithms/RandUtils.h"
using namespace Microsoft::VisualStudio::CppUnitTestFramework;

namespace SortTest
{
	TEST_CLASS(SortTest)
	{
	public:
		TEST_METHOD(quickSort)//快速排序
		{
			Sorting sort;
			int length = 10;
			int*  nums = RandUtils::random(length,1,100);
			std::string numsBegin = "数据开始：" + RandUtils::intArrToStr(nums, length);
			Logger::WriteMessage(numsBegin.c_str());
			sort.quickSort(nums, length);
			std::string numsEnd = "排序结果："+ RandUtils::intArrToStr(nums, length);
			Logger::WriteMessage(numsEnd.c_str());
			Assert::IsTrue(RandUtils::isSorted(nums, length));
            delete nums;
		}
	};
}
```



## 单元测试结果

![unittest](/assets/images/unittest.png)
