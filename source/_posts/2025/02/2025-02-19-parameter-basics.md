---
title: 传参也要讲究基本法
mathjax: false
categories:
    - Design
tags:
    - parameter
date: 2025-02-19 11:13:18
---

## AI Summary(DeepSeek R1)
在代码设计中，避免将庞大的结构体透传至底层函数，以减少认知负荷与依赖耦合。传递完整结构体会迫使开发者逐层理解无关字段，增加维护难度，同时引发编译效率问题。

优化方案是解构参数，仅传递必要字段（如 `bool` 和 `vector`），而非透传整个对象。这能明确函数职责，降低复杂度，提升可维护性和编译效率。原则是“按需传递”，避免引入无关信息。

## Background

最近在阅读存量代码，其中有一个很痛苦的点是，上游巨大的结构体，被原封不动地透传到不同的 callee 中。

这种类型的形参，从数据最原始进来的地方，一直被传到底下最小的、仅有十多行长度的 helper function 中。

```cpp
#include <vector>

//* Data
struct BasePoint {
    int x;
    int y;
};

namespace UpperStream {
struct RawData {
    bool is_out_of_bound{false};
    std::vector<BasePoint> neighbors{};
};
}  // namespace UpperStream
```

```cpp
//* Bad Practice
void SubProcess(const UpperStream::RawData& raw_data) {
    if (raw_data.is_out_of_bound) {
        for (const auto& neighbor : raw_data.neighbors) {}
    } else {
    }
}

void ProcessRaw(const UpperStream::RawData& raw_data) {
    SubProcess(raw_data);
    // Do something else
}
```

## Bad Smell

我认为这样的实践有这么些坏处：

1. 对人来说：引入了严重的认知负荷。

    对于阅读代码的人来说，这个巨大的 struct 伴随着他对代码的每一层深入阅读，这个过程中往往伴随着这么一些心理活动：

    - 这个结构体的内容我都不了解，也没有用注释，是个啥意思啊？
    - 这个字段跟那个字段有什么关系，怎么一会访问 A 字段和 B 字段，一会访问 A 字段和 C 字段？
    - 我发现还有个字段 D，怎么我看了 5 层了，都没看到访问它的？

2. 对代码本身来说：提供了多余的信息，扩散并引入不必要的依赖关系。

    举个栗子，对于一个底层的函数、算法来说，仅仅需要告诉 `bool is_truncate` 和 `span<T> data` 就可以了，作为这个函数的提供者/实现者，并不关心的 `is_truncate` 是通过一个什么计算、一个什么配置项、一个对于 raw data 的什么加工获得的结果，所以我不需要也不应该把我的函数形参定义成处理 raw data 的类型的样子。

    这样才能做到真正的解耦和复用。

    > 想到 Cpp Core Guidelines 里面的，不要使用智能指针作为函数形参，这样对使用者的约束是很大的。这里讲究的就是一个“意”，不需要的东西和概念，一概不要引入。
    >
    > [F.7: For general use, take `T*` or `T&` arguments rather than smart pointers](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f7-for-general-use-take-t-or-t-arguments-rather-than-smart-pointers)

3. 潜在的编译性能问题。在 C++ 里面类型的定义往往是通过引入头文件引入的，过多的不必要的依赖会导致编译和链接时间过长，也不利于不同团队之间的交付配合：因为对方修改大结构体之后，你需要更新头文件重新编译你的程序啊。


## Good Practice

如果把 struct 的字段的组合关系组成看成一棵树，那么一个理想的情况是，我们不“跨层访问”里面的节点，这就要求我们在设计之初就构想好结构体的含义和定位。~~（当然需求来了一定会变的啊）~~

此外，函数调用的逻辑每往里走一层，对字段对信息的需求应该是越来越少，函数的参数和逻辑应该是越来越集中。注意这里说的是业务处理、函数调用的逻辑，并不是指每经过一个函数调用就必须变得简单，因为会存在一些为提升可读性复用性提取的代码。

例如：

```cpp
//* Good Practice
void BetterFunc(bool is_out_of_bound, const std::vector<BasePoint>& neighbors) {
    if (is_out_of_bound) {
        for (const auto& neighbor : neighbors) {}
    }
}

void BetterProcessRaw(const UpperStream::RawData& raw_data) {
    BetterFunc(raw_data.is_out_of_bound, raw_data.neighbors);
    // Do something else
}
```

仅仅是从 `BetterFunc` 的签名来看，避免了透传整个RawData结构体，`bool` + `std::vector` 的可用性、简洁性就一定会比 `const UpperStream::RawData&` 好得多。这种设计不仅简化了函数签名，还明确了函数的职责范围，降低了复杂度。

在 `BetterProcessRaw` 中，我们实现了对原始大结构体的“解构”，也实现了对思维复杂度的降维：“徒儿啊：从这往下走啊，你只有一个 bool 和 data 了，外面的风风雨雨都与你无关”。

## Wrap Up

传参也是有讲究的：“头是头，尾是尾，不需要的东西，一概不给。”
