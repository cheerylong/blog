---
layout: blog
title: 【译】React Native布局原理（以及Fabric将做出的改变）
date: 2019-01-24 22:56:45
categories: React Native
tags: React Native
toc: true
---


![](https://user-gold-cdn.xitu.io/2019/1/20/1686b38c126680b5?w=1920&h=1080&f=jpeg&s=440457)


> - 原文作者：[Mehul Mohan
](https://medium.freecodecamp.org/@mehulmpt)
> - 原文链接: [How React Native constructs app layouts (and how Fabric is about to change it)](https://medium.freecodecamp.org/how-react-native-constructs-app-layouts-and-how-fabric-is-about-to-change-it-dd4cb510d055)
> - 作者 React Native 入门教程：[React Native - First Steps
](https://www.udemy.com/react-native-first-steps/?couponCode=MEDIUM_SPECIAL)
> - 作者 Twitter: [Mehul Mohan](https://twitter.com/mehulmpt)

`React Native` 团队一直致力研究一些可以从根本上改变 `React Native`内部与操作系统协同工作的方式。
在官方没发布前，暂且称这个项目为 “Project Fabric”

让我们来讨论下它是什么以及它会为开发人员带来什么变化。

## 目前React Native是如何工作的

`React Native` 目前使用3个线程：

![](https://user-gold-cdn.xitu.io/2019/1/20/1686b44a5766b0ce?w=965&h=507&f=jpeg&s=40227)

1. UI线程 - 这是运行 `Android / iOS` 应用程序的主要应用程序线程。
它可以访问UI，UI也只能由此线程更改。

2. 影子线程 - 此线程是 `React Native` 用于计算 `React` 创建的布局的后台线程。

3. JavaScript线程 - 此线程是 `JavaScript` 代码（实际上是您的 `React` 代码）的执行环境。

## 内部通信

让我们从头开始分析构建过程，假设你要在屏幕中央绘制一个红色框。
那么，你的JS线程包含用于创建这个布局的代码。
这是一段典型的 React Native（RN） 布局代码：

````javascript
    <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
        <View style={{width: 100, height: 100, backgroundColor: "red"}}></View>
    </View>
````

系统有自己的布局实现方式，它并不理解你刚刚编写的Flexbox代码。因此，`RN` 首先必须将您的 `flexbox` 编码布局转换为系统可以理解的布局系统。

继续来看，在转换之前，我们需要将此布局计算部分转交到另一个线程，以便我们可以保持 `JavaScript` 线程继续执行。因此，RN使用了影子线程，它本质上是在构建了JS线程中编码的布局树。在这个线程中，RN 使用了一个名为 [Yoga](https://github.com/facebook/yoga) 的布局引擎，它将基于 `flexbox` 的布局转换为系统可以理解的布局。

`React Native` 使用 `React Native bridge` 将这些信息从JS线程传递给Shadow线程。
简而言之，这只是以JSON格式序列化数据并将其作为字符串传输到 `React Native bridge` 上。

这时，我们正处于Shadow线程中。JS线程正在执行，屏幕上没有任何内容。
一旦我们从 `yoga` 获得了渲染标记，这些信息将再次通过 `React Native bridge ` 传输到UI线程。
同样，这会在Shadow线程上执行一些序列化并在主线程上执行反序列化。然后主线程会渲染出 UI。


![](https://user-gold-cdn.xitu.io/2019/1/20/1686b4bcdd6c93a7?w=922&h=475&f=jpeg&s=43424)

## 存在的问题

你可以看到，线程之间的所有通信都发生在 `bridge` 上，但是同时存在很多限制。
包括：
- 传输较大数据（比如将图像转换为base64字符串信息）会很慢，
如果只需指向内存中的数据就可以实现相同的任务，这些就是不必要的数据拷贝

另外，所有的通信都是异步的，在大多数情况下这是没问题的。
但是，目前没有办法同步从JS线程更新UI线程。当您使用具有大量数据的 `FlatList` 时，这就会产生问题。（你可以将 `FlatList` 视为 `RecyclerView` 的较差实现。）
最后，由于JS线程和UI线程之间通信是这种异步的方式，严格要求同步数据访问的原生模块并不能完全使用。
例如，`android` 上的 `RecyclerView` 为了屏幕上没有闪烁，适配器需要同步访问它所呈现的数据。
由于目前 `React Native` 的多线程架构，无法做到这一点。

## 介绍Fabric

我们回头来看看浏览器的布局。我们仔细想下，浏览器渲染出的输入框，按钮等实际上是依赖于操作系统的。因此，在渲染时，你的浏览器会询问您的操作系统（Windows，Mac，Linux或其他的系统），例如，应该在网页上的哪个位置绘制输入框？那让我们来看下浏览器和`React Native` 的线程映射关系。

- UI线程 → UI线程

- 浏览器渲染引擎 → React Native渲染引擎（Yoga / Shadow thread）

- JavaScript线程 → JavaScript线程

我们知道现代浏览器非常成熟，可以有效地处理掉所有这些任务。
那为什么 `React Native`不能这么做呢？是什么限制了它不能像浏览器这样来工作呢？

## 将Native API直接暴露给JavaScript


![](https://user-gold-cdn.xitu.io/2019/1/20/1686b4ec682c467f?w=547&h=269&f=jpeg&s=32764)

你曾经有没有在控制台中写过像 `document.getElementById` 这样的命令以及 `setTimeout` 和 `setInterval` 之类的代码并看到打印出的结果？他们的实现结果是 `[native code]` , 这是什么意思呢？可以发现，当你执行这些方法时，它们不会调用任何 `JavaScript` 代码。反而，这些方法会直接调用到本机的C ++代码。所以其实浏览器不允许JS使用桥接与主机操作系统进行通信，而是使用本机代码直接向系统暴露JS接口！简而言之，这就是 `React Native Fabric` 将要做的事情：消除桥接通信并且使用本机代码直接通过JS线程控制UI。

## 小贴士

- RN Fabric允许UI线程（绘制UI的位置）与JS线程（UI编程的位置）同步
- Fabric仍处于开发阶段，React Native团队确实没有提到截至目前的公开发布日期。
但我很确定今年我们会看到一些很棒的东西。
- 像这样的应用程序开发框架（RN，NativeScript，Flutter）将会发展的越来越好！

*图片来源：https：//www.slideshare.net/axemclion/react-native-fabric-review20180725*
