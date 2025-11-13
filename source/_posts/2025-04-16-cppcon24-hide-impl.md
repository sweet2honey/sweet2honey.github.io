---
title: "[Video] How to Hide C++ Implementation Details - Amir Kirsh - CppCon 2024"
mathjax: false
toc: true
categories:
  - Reading
tags:
  - cpp
date: 2025-04-16 11:18:13
description:
---
# 评价

前面的概念听听还是可以，后面上了代码都是些啥啊？

# Why hide?

Encapsulation

不应公开的数据就不会被意外修改；；用户只需要了解最少的信息；内部改动不影响外部代码；数据变化在内部的特定地方发生，方便调试。

Decoupling

减少不必要的依赖；修改不会影响其他部分；组件更加通用；更方便隔离测试。



# Why not simple?

- We're lazy and don't think it's important

  > `std::pair<First, Second> p;`

- We do not realize we have exposed details

  > ```c++
  > const std::map<string, int>& getProp() const;
  > ```

- Technical constraints

  > ```c++
  > class Foo {
  >  int value;
  > public:
  >  // we prefer the ctor below to be private but it doesn't compile :(
  >  Foo(int value) : value(value) {}
  >  static unique_ptr<Foo> create(int value) {
  >      return std::make_unique<Foo>(value);
  >  }
  > };
  > ```

- Trade-offs

  不希望使用仅依赖于 interface 的解决方案（不希望用 virtual call）；

  把数据暴露给用户或许有助于提高性能（vector 意味着可以按索引访问）。

  但：

  - 有时候目的不足以在设计上妥协
  - 或许有两全其美的方案



# API

## Hyrum's Law

> Developers come to depend on all observable traits and behaviors
> of an interface, even if they are not defined in the contract.

- sers use it => Users abuse it.

  用户按照观测现象来使用 API，而非接口契约。

- Too many options/overloads => wrong one would be picked at sometime.

  更少的选项意味着更少的测试

## Be lean and mean

1. Keep your API short and concise.
2. Limit your API to what should actually be used
3. Different usages may get different view (or copy) of the same data,
   exposing only what is relevant.

就是严格，信息最小化。

**The interface that we provide shall be as narrow as possible.**

> Because:
>
> If they don’t need it, don’t give it.
>
> If they get it, they’ll use it.
>
> If they use it, they’ll abuse it.
>
> Trust no one

## Passing args

- value sematics
- interface
- pimpl
- wrapper class
- C++ 20 concepts

## What’s a Concept?

A constraint on template arguments

Evaluated to boolean value at compile time

May be used for overload resolution