---

layout: default
title: Chromium Multi-process Architecture

---


# Table of Contents

1.  [问题引入](#orgafd018c)
2.  [架构总揽](#org312ba0b)
    1.  [管理渲染进程](#org222efab)
    2.  [视图管理](#org39db9af)
3.  [组件和接口](#org50c25bb)

简单翻译下Chromium的多进程架构文档，原文档猛击[Mulite-process Architecture](https://sites.google.com/a/chromium.org/dev/developers/design-documents/multi-process-architecture)


<a id="orgafd018c"></a>

# 问题引入

几乎不可能建立一个不会崩溃或挂起的渲染引擎，同样也几乎不可能达到几乎完美的安全性能。
从某方面来讲，2006年的浏览器是一个类似单用户-多任务协调合作的操作系统。一个行为不端的网页就像一个行为不端的应用会毁掉整个操作系统，它也会毁掉浏览器。只要一个小小的浏览器或者插件的bug就可以搞跨整个浏览器以及所有已经打开的标签页面。
现代操作系统把每个应用程序运行在一个独立的进程里，每个进程都相互隔离，所以很健壮。一个程序崩溃了，不会妨害其他的程序，也不会影响操作系统的健壮。用户之间的数据访问是受限制的。


<a id="org312ba0b"></a>

# 架构总揽

我们给每个标签页建立独立的进程，防止渲染引擎的bug和故障导致整个应用崩溃。同样限制渲染引擎之间，以及渲染引擎和其他部分的访问。也就是给网页浏览带来了内存保护以及操作系统的访问控制。
运行UI，管理标签和插件的进程成为"browser process"或"browser"。同样地，标签页进程称为"render process"或者"renders"。渲染引擎使用开源的Blink来解析布局HTML页面。
![img]({{"/assets/arch.png" | absolute_url}})


<a id="org222efab"></a>

## 管理渲染进程

每个渲染进程都有一个全局的RenderProcess对象，用来跟父级浏览器进程进行通讯，并维护全局的状态。对应于每个渲染进程，browser维护一个RenderProcessHost对象，同时也负责管理browser状态以及与通信。渲染进程与browser使用[Chromium's IPC system](https://sites.google.com/a/chromium.org/dev/developers/design-documents/inter-process-communication).


<a id="org39db9af"></a>

## 视图管理

每个渲染进程都有一个或多个RenderView对象。这些对象由RenderProcess管理，对应于标签的内容。与
RenderProcess对应的RenderProcessHost维护一个RenderViewHost，RenderViewHost对应于渲染器里的一个视图。
每个视图都有一个ID，用于区分同一个渲染器里的多个视图。这个ID是只在渲染进程里唯一，browser内确定一个
视图需要RenderProcessHost和view ID才行。RenderViewHost可以使得browser和标签内容进行通信，因为它知道
如何通过RenderProcessHost向RenderProcess以及Renderview发送消息。


<a id="org50c25bb"></a>

# 组件和接口

渲染进程:

-   RenderProcess处理与Browser里相关的RenderProcessHost的IPC。一个RenderProcess对象就是一个渲染进程。
    这就是渲染器和browser如何通信的。
-   通过RenderProcess，RenderView对象与browser进程里对应的RenderViewHost相互通信，包括Webkit内嵌层。这
    个对象表示一个页面或弹出窗口的内容。

浏览器进程:

-   Browser对象代表了最上层的浏览器窗口。
-   RenderProcessHost代表了浏览器侧的一个单一的browser与渲染器的IPC链接，并且RenderProcessHost与渲染器
    进程是一对一的关系。
-   RenderViewHost封装了与远端RenderView的通信。RenderWidgetHost为browser内的RenderWidget处理输入和绘
    画。

For more detailed information on how this embedding works, see the [How Chromium displays web pages](https://sites.google.com/a/chromium.org/dev/developers/design-documents/displaying-a-web-page-in-chrome).
