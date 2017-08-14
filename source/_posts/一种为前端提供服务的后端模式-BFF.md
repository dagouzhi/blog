---
title: 一种为前端提供服务的后端模式(BFF)
date: 2016-12-19 10:23:25
tags: BFF
---

> 术语解释 BFF （ Backend for Frontend ）
  1. BFF ( Backend for Frontend ) 意为前端专设一个服务端
  2. 从底层服务抓取数据·组装·裁剪，然后提供给前端
  3. BFF专为前端服务，要能够快速响应前端的变化，提供各种各样的数据

![](/images/C6F003CB6AF935154F7AC9F21018273B.jpg)

## 摘要

> 由于移动设备屏幕更小、数据规划有限以及需要更少的请求，移动端的Web体验与桌面端Web浏览器的体验在很多方面都有差异。移动设备需要的数据更少且常常不同于桌面端的需求，而且可能会提供其他的交互方式，例如通过条形码读码器。这就意味着，为了支持移动设备，我们需要向我们的API后端添加额外的功能。Sam Newman在一篇博客中这样解释到。

## 正文

由于移动设备屏幕更小、数据规划有限以及需要更少的请求，移动端的Web体验与桌面端Web浏览器的体验在很多方面都有差异。移动设备需要的数据更少且常常不同于桌面端的需求，而且可能会提供其他的交互方式，例如通过条形码读码器。这就意味着，为了支持移动设备，我们需要向我们的API后端添加额外的功能。[Sam Newman](http://samnewman.io/patterns/architectural/bff/)在一篇博客中这样解释到。这篇博客描述了[一种API后端的模式](http://samnewman.io/patterns/architectural/bff/)，用来处理这种不同用户体验的设备之间的这种不协调。

正如这位Thoughtworks的开发者Newman所言，一种解决方案是构建一个为所有类型的用户界面服务的通用的API后端。然而正是由于需求不同，这种方案在实践中意味着后端功能和复杂度的增加。同时，为了支持所有设备而做的变动也可能导致该方案成为拖累部署流程的瓶颈。根据Newman的经验，使用这种通用后端有时会导致一个专门的团队出现，进而导致问题增加。这种情况下，前端团队会有一个单独的团队需要去沟通，而这个单独的团队需要确定来自其他团队的需求的优先级。

Newman见过的另外一种实际在用的方案是为每种用户体验提供一套API后端。
从概念上讲，一个面向用户的应用会有两个组件组成，客户端组件和服务端组件，每个前端一个后端。为此，Phil Calçado在SoundCloud工作的时候还发明了一个术语：BFF。

当使用BFF架构工作的时候，前后端由同一个团队来维护，后端很自然地会和某种特定的用户界面紧耦合。当处理同种类型的用户体验，但是不同的平台（例如Android和iOS），Newman描述了两种不同的处理方式：每个平台提供一套BFF和每种类型用户界面提供一套BFF。

Newman更倾向于采用一种严格的模型，为每个平台提供一套BFF，即Android和iOS各一套。这种方式有一个令人担忧的问题，就是需要冒着不同平台的BFF之间大量重复的风险，例如，相同类型的聚合或者与下游服务交互的相似代码。但是，他并不太担心这个，因为这种重复已经跨越了不同的流程。恰恰相反，合并成一种通用的聚合Edge API服务才是他想警告的。因为事实已经反复证明，这种模式会导致高度膨胀的代码。

让所有类型的客户端共用一套BFF，即Android和iOS共用一套BFF，是他在SoundCloud看到的实际使用中的方式。他对这种方式的担心就是，更多类型的客户端会导致整个BFF膨胀的风险增加。

同样在Thoughtworks工作的[Lukasz Plotnicki](https://www.thoughtworks.com/profiles/lukasz-plotnicki)，最近写了一篇博客，专门论述了[SoundCloud在从单体Rails应用转向微服务的过程中对BFF的使用](https://www.thoughtworks.com/insights/blog/bff-soundcloud)。

在Thoughtworks最近的Technology Radar中，BFF作为值得推行的技术而被提及。

查看英文原文：[A Pattern for API Backends Serving Frontends](http://www.infoq.com/news/2015/12/bff-backend-frontend-pattern)
