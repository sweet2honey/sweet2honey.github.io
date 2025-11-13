---
title: "[Video] Refactoring C++ Code for Unit testing with Dependency Injection - Peter Muldoon - CppCon 2024"
mathjax: false
toc: true
categories:
  - Reading
tags:
  - cpp
date: 2025-04-18 11:21:14
description:
---
# Conclusion

[Refactoring C++ Code for Unit testing with Dependency Injection - Peter Muldoon - CppCon 2024](https://www.youtube.com/watch?v=as5Z45G59Ws)



评价：⭐⭐⭐⭐

- 依赖注入就是约定接口，通常就是几派：继承、模板、类型擦除，总结得很好。

- Refactoring for DI 那一节很不错，是一个从屎山开始拆解业务、理解业务和抽象业务的好例子。

  -  归类、分解代码是为了单独测试，那一但有了明确的分类，那么就有替换、注入的可能性了。

- 相比起把所有的东西全部塞到一个 struct 里面，把意义关联的、对称的概念用 template struct 标明出来，能获得更好的内聚性、更好的类型安全（没有隐式转换！！！）。

  ```cpp
  template<typename T>
  struct OptionalPairT {
      std::optional<T> bid_;
      std::optional<T> ask_;
  };
  using SidePair = OptionalPairT<Side>;
  using BrokerPair = OptionalPairT<Broker>;
  using YieldPair = OptionalPairT<Yield>;
  ```



# Method of injection

## Linking

Pro

- No code change

Con

- UB/ODR violation
- Brittle and confusing

> 链接是个奇妙的东西，小心玩火。

## Inheritance

Pro

- Can handle a lot of methods
- Well understood mechanism
- Easier to add to old code

Con

- Numerous functions with testing functions inside
- Data mixed in
- V-table extra hop

> 经典，但是什么都往 interface 里面加，还不如 CV。

## Template

Pro

- Only define methods that actually need
- No running overhead
- Use concepts(C++20) as an "interface"

Con

- Hard to add to old code

- Compile time

> 没有 Concept 就是难写一些；而且类型不同的话没法方便用容器。

## Type erasure

Calling any thing satisfying a function signature via `std::function` `std::invoke`

Pro

- Invokable on any callable target
- Versatile

Con

- Can handle only one method
- Runtime overhead

> 80% 情况下的通解了。

## Null valued objects/stubs

A stub with no functionality - only satisfy type requirements.

- Disable part of the system
- No actual logic: discard arguments, return fixed values



# Types of Dependency Injection

## Setter DI

```cpp
void SetSender(std::unique_ptr<Sender> sender)
```

**Do not use this !!!**

Class could be unusable for some time: sender not set.

## Method DI

```cpp
Resp Send(Com& com, ...) {
    com.Send(...);
}
```

**The function signature is changed.**

e.g. may not work for an external library.

## Constructor DI

```cpp
class Processor{
	Com& com_;
public:
    Processor(Com& com): com_(com){}
};
```

**The constructor is changed.**



# Roadblocks of DI

## Object creation hidden

- No handle to inject
- Constructors via Singletons or globals

### Reaching through multiple objects

- Long chain of process breaks the principle of least knowledge

### Disentangle getting from setting

- Dig out pure functions

  ```cpp
  if (handle.getBidPrice(&decimalPrice))
  	bid_.setPrice(Tickers::PriceVariant(decimalPrice));
  else if (handle.getBidPrice(&doublePrice))
  	bid_.setPrice(Tickers::PriceVariant(doublePrice));
  else if (handle.getBidPrice(&floatPrice))
  	bid_.setPrice(Tickers::PriceVariant(floatPrice));
  
  const boost::optional<Tickers::PriceVariant> &bid_price = getBidPrice(handle);
  if(bid_price)
  	bid_.setPrice(*bid_price);
  ```

### Having too many dependencies in one place

- Impractical to pass

### Class packed with huge chunks of data/functionality

- God class
- Too many dependencies

### Functionality splintered and spread around

- Fragmented inheritance chain
- Duplicated code
- **Blended into many utility classes**

### Lack of data structure

- Ungrouped data

  ```cpp
  Data getData(const Tick&,
               const std::optional<Side>& bid, 
               const std::optional<Side>& ask, 
               const std::optional<Side>& localBid,
               const std::optional<Side>& LocalAsk, 
               const std::optional<Broker>& bidBroker,
               const std::optional<Broker>& askBroker, 
               const std::optional<Yield>& bidYield,
               const std::optional<Yield>& askYield) const;
  ```

  Gather data into **coherent** data structures



# Highway Express of DI

- Object creation done outside the logic of functions

  Pass in Dependencies directly

  Pass in Dependency suppliers

  > 让信息从外部输入

- Invoke methods on immediate objects

  Avoid invoking methods on an object returned by other methods

- Disentangle information retrieval/calculation from state changing

  Find the const or pure functions

  > 寻找无状态的函数

- **Refactor** God classes

  Functionality clustered and pushed into tiered abstraction layers

  Lessen unnecessary dependencies

  > 分解大坨的屎

- **Refactor** fragmented functionality

  Cluster splintered functionality together

  Lessen dependencies

  > 内聚耦合分明

- **Refactor** data/state

  Gather into coherent data structures

  > 抽取有意义、耦合的数据结构



# Immutable APIs

- API is used wide so interface cannot change

  Transparent DI using:

  - Default arguments
  - Delegating functions
  - Delegating constructors

- Lazy initialization

  Use a dependency provider/creator.

- DI unexpected snags

  > 这个没看懂

  - Turn into regular function
  - Add type erasure at call point
  - Use template DI



# Conclusion

## DI Myths

- It's simple? Only for simple systems parts
- Overkill on small projects
- Easy to add later

## DI Truths

- It's hard for real production systems
- Properly factored code is the **KEY** to DI
- Give weight to **local refactoring** prior to DI
- Poor code needs more work for DI
- Improves the flexibility/re-usability/testability of a system
- Better long term maintainability of code

## DI Finally

 **Lessening number of dependencies** needing injection into an interface

- Horizontal abstraction : Refactoring code into decoupled functional chunks
- Vertical abstraction : Refactoring code into tiered layers

> 横向解耦，纵向分层



# Extras

Retiring The Singleton Pattern : Concrete Suggestions on What to Use Instead

Redesigning Legacy Systems : Keys to success

Managing External APIs in Enterprise Systems

Exceptionally Bad : The Story on the Misuse of Exceptions and How to Do Better (Exceptions in C++ : Better Design Through Analysis of Real-World Usage)

Software Development Completeness : Knowing when you are done and why it matters