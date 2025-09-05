---
title: "abseil tips #1 string_view #231 algo #232 auto"
mathjax: false
toc: true
categories:
  - Programming
tags:
  - cpp
  - design
date: 2025-09-04 10:43:49
description:
---

# #1 string_view

`string_view` 仅由一个指针和长度组成，标识一个**不属于** `string_view` 所有且**无法被视图修改**的字符**数据段**。

注意：
- 生命周期
- 不一定是 `NULL` 结尾

# #231 Overlooked Algorithms

`std::clamp`：
- 返回的是引用
  ```cpp
  const int& dangling = std::clamp(1, 3, 4);
  ```
- 用 `<` 或者 `cmp` 比较

`std::midpoint` `std::lerp`：
线性插值用。

# #232 When to use auto

> Use type deduction only if it makes the code clearer to readers who aren’t familiar with the project, or if it makes the code safer. Do not use it merely to avoid the inconvenience of writing an explicit type.
> 仅在以下情况下使用类型推导：对于不熟悉项目的读者来说，它能使代码更清晰；或者它能使代码更安全。不要仅仅为了避免编写显式类型的麻烦而使用它。

## 难以正确、高效描述类型

```cpp
for (const std::pair<std::string, DogBreed>& name_and_breed :
    dog_breeds_by_name) {
  ...
}
```

这里的 Key 类型其实是 `const std::string`，这就导致了拷贝。

## 冗长类型影响阅读

当容器类型在附近可见时，直接用 `auto` 是不错的选择：

```cpp
std::vector<std::string>::iterator name_it = names_.begin();
while (name_it != names_.end()) {
  ...
}
```

如果容器类型在附近不可见，建议把迭代器或者元素类型体现出来：
```cpp
auto name_it = names_.begin(); // 不知道 names_ 装的是啥
while (name_it != names_.end()) {
  const std::string& name = *name_it;
  ...
}
```

---

`std::make_unique<T>` 就返回 `std::unique_ptr<T>`，没必要再重复写了。

```cpp
auto my_type = std::make_unique<MyFavoriteType>(...);
```

## 总结
考虑性能，小心意外的拷贝；可读性是依赖上下文的。