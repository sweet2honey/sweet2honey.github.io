---
title: 《大教堂与集市》读书笔记
mathjax: false
toc: true
categories:
  - Reading
tags:
  - book
  - opensource
  - management
date: 2025-10-09 17:35:38
description:
  开源运动的独立宣言？对现时的启发还是有的，大概是关于热爱驱动、共建，还是挺理想和美好的。可能也是开源吸引人们的地方吧。
---

2002 年作者公开的 3.0 版本：

ENG: [http://www.catb.org/~esr/writings/cathedral-bazaar/cathedral-bazaar/index.html](http://www.catb.org/~esr/writings/cathedral-bazaar/cathedral-bazaar/index.html)

CHS: [https://github.com/thuwyh/The-Cathedral-the-Bazaar-zh/tree/master](https://github.com/thuwyh/The-Cathedral-the-Bazaar-zh/tree/master)

CHT: [https://crazyangelo.github.io/Cathedral-and-Bazaar/](https://crazyangelo.github.io/Cathedral-and-Bazaar/)

2014 出版的版本，内容多了不少：

[https://www.chenshaowen.com/static/file/books/musings-on-linux-and-open-source.pdf](https://www.chenshaowen.com/static/file/books/musings-on-linux-and-open-source.pdf)

---

1. Every good work of software starts by scratching a developer's personal itch.

    每一个优秀的软件作品都是从解决开发者个人需求开始的。

有点尖锐了，which means 大多数人上班写的东西都不是“优秀”的，因为这些程序他们既不需要，也不热爱（还很讨厌铲屎）。

我会想到在开发分支上疯狂重构代码的事情，从函数名、入参、调用链、头文件都改。为啥要改呢？也说不出来，可能是强迫症？可能只是表示对好的东西的一种追求和偏执的喜欢？

‍

2. Good programmers know what to write. Great ones know what to rewrite (and reuse).

    优秀的程序员知道要写什么程序，伟大的程序员知道要改写（和重复利用）什么/哪些程序。

程序员的产出最终是结果导向的，所以从好的半成品开始总是更容易更有效率。

‍

3. “Plan to throw one away; you will, anyhow.”(*The Mythical Man-Month*, Chapter 11)

    做好抛弃（第）一个开发的系统的准备；不管如何，你都会这么做的。

往往在第一次实施解决方案之后才能真正地理解问题。如果想把事情做好，至少要准备好把东西推倒重做一次。这不是说预期着第一次做的事情就是错的，而是说按照正确的想法思路重开一把会比持续地挽救一场灾难更有效率，更经济。

‍

4. If you have the right attitude, interesting problems will find you.

    抱持正确的态度，就会发现有趣的问题。

5. When you lose interest in a program, your last duty to it is to hand it off to a competent successor.

    当你对一个问题不再感兴趣时，你最后的责任就是找位能胜任的接棒人。

‍

6. Treating your users as co-developers is your least-hassle route to rapid code improvement and effective debugging.

    将用户视为共同开发者是您实现代码快速改进和有效调试的最省事途径。

7. Release early. Release often. And listen to your customers.

    早发布。常发布。倾听客户的声音。

8. Given a large enough beta-tester and co-developer base, almost every problem will be characterized quickly and the fix obvious to someone.

    以足够多的 beta 版测试者和协同开发者做基础，几乎程序中的每一个问题都可以很快地找出来，并且对某些人而言，针对发现的问题的解决方法是显而易见的。

> **德尔菲法**（Delphi method）是一种结构化的[决策支持技术](https://zh.wikipedia.org/wiki/%E5%86%B3%E7%AD%96%E6%94%AF%E6%8C%81%E7%B3%BB%E7%BB%9F "决策支持系统")，它的目的是在讯息收集过程中，通过多位[专家](https://zh.wikipedia.org/wiki/%E5%B0%88%E5%AE%B6 "专家")的独立的反复主观判断，获得相对客观的讯息、意见和见解。

开源开发使得测试人员和开发人员更容易基于实际源代码开发共享认知上下文，并有效地进行沟通。

‍

9. Smart data structures and dumb code works a lot better than the other way around.

    聪明的数据结构配上笨拙的代码要比相反的组合好。

> 数据的表现形式是编程的根本。(*The Mythical Man-Month*, Chapter 9)

混乱的数据结构通常就是“缺乏思考”的产物，各种毫无关联的字段堆砌在一起，把这样的数据丢给某个函数执行，你完全不会知道会发生什么事情，函数职责的单一性也没法有效体现:

- 你必须逐行代码看，才能确定到底访问了哪些信息、资源；
- 可变性 `constness` 的控制颗粒度太大，不利于优化；

‍

10. If you treat your beta-testers as if they're your most valuable resource, they will respond by becoming your most valuable resource.

     如果你视 beta 版测试者如同你最珍贵的资源，那么他们就会成为最珍贵的资源。

‍

11. The next best thing to having good ideas is recognizing good ideas from your users. Sometimes the latter is better.

     仅次于拥有好点子的是从你的用户那里识别好点子，有时候后者反而更好。

12. Often, the most striking and innovative solutions come from realizing that your concept of the problem was wrong.

     通常，最有突破性和最有创意的方案法来自于发觉自己对问题原先的观念是错误的。

有足够的决断力丢弃老的东西，同时保持功能不回退。

‍

13. “Perfection (in design) is achieved not when there is nothing more to add, but rather when there is nothing more to take away.”

     设计上完美，不是「没有东西能再被加入」，而是「没有东西能再被移出」。

Less is more.

‍

14. Any tool should be useful in the expected way, but a truly great tool lends itself to uses you never expected.

     任何的工具以我们所知道的方法来使用都会有用，但一个真正了不起的工具会以你从未想过的使用方法来发挥它的功能。

这个通常是开发者很难提前想象出来的；也说明产品可以有很多使用方法，挖掘到细分领域的特别需求也许是个推广的触点。

‍

15. When writing gateway software of any kind, take pains to disturb the data stream as little as possible—and never throw away information unless the recipient forces you to!

     在编写任何类型的网关软件时，应尽力减少对数据流的干扰——并且除非接收方强迫你这么做，否则永远不要丢弃信息！

不要过早地，或者说激进地进行程序的抽象和分层，尤其是涉及到通信和字节流相关的。

考虑到之前设计协议的经历，报文头的兼容性（版本的检测，版本握手，前后兼容）也是个在早期就要考虑的。

‍

16. When your language is nowhere near Turing-complete, syntactic sugar can be your friend.

     当你设计的语言没有一处是 Turing-complete，你可以采用比较平易的语法。

DSL 的好处吧，可以考虑易用性和降低错误概率优先。

‍

17. A security system is only as secure as its secret. Beware of pseudo-secrets.

     安全系统的安全性仅取决于其机密性。警惕伪机密。

‍

**Necessary Preconditions for the Bazaar Style**

**市集模式必要的条件**

I think it is not critical that the coordinator be able to originate designs of exceptional brilliance, but it is absolutely critical that the coordinator be able to *recognize good design ideas from others*.

我想对于项目协调者是否能做出耀眼的设计并不重要，最重要的是协调者是否能认知别人在设计上的好点子。

A bazaar project coordinator or leader must have good people and communications skills.

市集模式项目中的协调者或领导人必须有人缘和好的沟通技巧。

‍

18. To solve an interesting problem, start by finding a problem that is interesting to you.

     要解有趣的问题，从寻找你感兴趣的问题开始！

其实是第 1 条的另一个说法。

‍

19. Provided the development coordinator has a communications medium at least as good as the Internet, and knows how to lead without coercion, many heads are inevitably better than one.

     假如项目开发协调者拥有至少跟因特网一样好的媒体，而他也不靠强制力来领导，那么一群人必定胜过一个人。

> 在 Weinberg 对「忘我编程」（egoless programming）的讨论中说到，在开发者不对其代码有领土意识，并鼓励其他人寻找其中的错误和潜在改进的地方，改进的速度比其他地方快得多。

‍

软件项目管理有五个主要功能︰

1. 定义目标，确保每一位成员方向一致。
2. 监督并且确定重要的细节没有被忽视。
3. 诱导成员去做乏味但必要的苦工。
4. 调整成员组织以期发挥最大生产力。
5. 安排足够的资源以支持整个项目。

但是对于开源项目来说：

1. 【资源】开发人都是志愿者，他们带来自己所拥有的资源；
2. 【生产力】开源的黑客们自行分工，以发挥最大的生产力；
3. 【动力】我们对开源社群能完成工作的信息，源自于「性感」的吸引力或者技术上的喜悦；

    因为问题本身的魅力，吸引了解决问题的人 ―― 不只在软件界，或者其他创造性的工作中，问题的魅力是一个比只提供金钱还更有效的推动因素。
4. 【细节】开源社群最强的一点就在于分散式的审校，这点胜过确认有无细节被遗漏的任何传统方法。
5. 【目标】开源项目领导者和社区长老在定义目标方面与管理层相比具有类似的作用，并可能更成功。

将来的事实会证明开源最大的成功之一就是它告诉了我们︰如玩游戏般地工作是从事创造性工作最经济、最有效率的模式。
