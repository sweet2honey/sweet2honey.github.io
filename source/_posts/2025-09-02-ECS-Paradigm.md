---
title: ECS Paradigm
mathjax: false
toc: true
categories:
  - Game Developing
tags:
  - design
  - ecs
date: 2025-09-02 14:25:57
description:
---

# ECS

ECS 是一种软件模式，将程序分解为：
- Entity 实体
- Component 组件
- System 系统

**Entities** are unique "things" that are assigned groups of **Components**, which are then processed using **Systems**.

这种模式是很典型的“组合”的思想。实体和组件是一对多的关系，实体拥有的组件决定了它所拥有的能力、行为，而行为是通过系统来实现的。

> 角色英雄想飞？只要给他加上飞行速度组件就行了。处理飞行的系统会让所有拥有飞行速度组件的实体都能按照“飞”的逻辑来行动。
> 反过来想，如果角色被“禁足”了，只需要移除他身上的移动组件即可，移动系统就不会访问这个实体了，角色自然就无法移动了。

ECS模式鼓励通过强制将应用数据与逻辑分解为其核心组件来实现干净、解耦的设计。

## Entity
实体只是一个容器，用于管理其中包含的所有组件的生命周期。

## Component
组件就是纯数据组合，没有操作数据的方法。

## System
系统就是纯方法的组合，它没有状态和数据（临时计算的不算）。
对于每个系统来说，游戏框架只需要筛选出当前系统关心的对象（世界中所有对象的一个*子集*），提供对应的数据就可以了。

# Reference
[ECS](https://bevy.org/learn/quick-start/getting-started/ecs/)
[浅谈《守望先锋》中的 ECS 构架](https://blog.codingnow.com/2017/06/overwatch_ecs.html)