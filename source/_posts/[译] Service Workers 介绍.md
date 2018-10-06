---
title: Service Workers 介绍 [译]
date: 2018-09-30 16:31:58
categories: translation
tags: [translation]
---

> [原文链接](https://developers.google.com/web/ilt/pwa/introduction-to-service-worker)

# 什么是 service worker

service worker 是 web worker 的一种。它本质上是一个 javaScript 文件独立运行于浏览器主线程之外，可以拦截网络请求，可以在缓存中缓存或取回资源，还可以分发推送消息。

因为 workers 独立运行于主线程之外，所以 service workers 和与它相关联的应用是互相独立的。因此有如下结果：
- 因为 service worker 不是同步的（它被设计成异步的），所以 `localStorage` 在 service worker 中不能被使用。
- service worker 可以在 app 不是活跃状态下接收来自服务器的推送消息。这个特性甚至可以让你的 app 在没有被浏览器打开的情况下，向用户展示推送通知。
- service worker 不能直接接触 DOM。如果需要和页面进行通信，那么 service worker 可以使用 `postMessage()` 来发送数据，另外通过监听 "message" 事件来接收数据。
<!-- more -->
> 注意：当浏览器没有运行的时候，通知消息是否被接收取决于浏览器和操作系统的交互方式。比如在桌面操作系统中，Chrome 和 Firefox 只有在浏览器打开的时候才能接收通知消息。然而，Android 是被设计成在接收推送消息时可以唤醒浏览器，并且总是接收推送消息，无论浏览器是什么状态。在[这本书](https://web-push-book.gauntface.com/)的 [FAQ](https://web-push-book.gauntface.com/chapter-07/01-faq/#why-doesnt-push-work-when-the-browser-is-closed) 中可以查看更多信息。

使用 service worker 需要注意以下内容：
- service worker 是一个可编程的网络代理，能够控制网页中的网络请求是如何被处理的。
- service worker 只运行在 https 中。因为 service worker 可以拦截网络请求和修改应答，所以中间人攻击很难实施。
- service worker 在不工作的时候会停下来，然后在需要的时候重新开始工作。在此过程中，你不能依靠全局变量来保存状态信息。如果你有这个需求，推荐使用 IndexedDB 数据库。
- service worker 广泛使用了 promises，如果你对 promises 不熟悉的话，推荐阅读 [Promises, an introduction.](https://developers.google.com/web/fundamentals/primers/promises)

# service worker 能做什么

service worker 使得 apps 可以控制网络请求，缓存请求结果来改进性能并且提供了缓存资源的离线访问方式。

service worker 利用两个 API 使得一个应用可以被离线使用：Fetch （网络请求资源的标准实现）和 Cache （应用数据的持久化存储）。Cache 是持久化的，并且与浏览器缓存和网络状态不相关。

## 为应用/站点改进性能

缓存资源将使得内容在大多数网络条件下加载更快。在[Caching files with the service](https://developers.google.com/web/ilt/pwa/lab-caching-files-with-service-worker) 和 [The offline cookbook](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/) 查看更多内容。

## 离线可用优先

在 service worker 中使用 Fetch API，我们可以拦截网络请求，也可以使用请求资源之外的内容来修改响应。我们使用这项技术可以保证用户离线的情况下依然可以使用服务。

## 高级功能的基础

service worker 是其他功能的基础，由此才得以让 web app 像原生应用一样工作。部分高级功能如下：
- Notifications API： 使用操作系统本地的通知系统向用户展示通知并进行交互。
- Push API：这个 API 使得你的应用可以订阅推送服务和接收推送消息。推送消息会由 Service Worker 处理，可以用来更新本地状态，也可以用来展示系统通知。由于 Service Worker 的运行不依赖于应用本身，所以它可以在浏览器没有运行的时候接收消息通知。
- Background Sync API：这个 API 可以在用户网络条件不好的时候收集用户需要发送的内容，然后等到网络条件良好的时候再发送。这个 API 还可以接收服务器的定时更新，从而使得应用可以在下次打开时进行更新操作。
- Channel Messaging API：这个 API 让 Web Worker、Service Worker 和主应用可以互相传递消息。这个 API 可以用于新内容通知和软件更新等与用户有交互的操作。

# service worker 生命周期

service worker 在它的生命周期内会经历以下3步：
- 注册
- 安装
- 激活

## 注册和作用域

在 __安装__ 一个 service worker 之前，你需要在你的代码中进行 __注册__ 。注册的生命周期是告诉浏览器 Service Worker 位于何处，然后在后台开始安装。

```javascript
// main.js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/service-worker.js')
  .then(function(registration) {
    console.log('Registration successful, scope is:', registration.scope);
  })
  .catch(function(error) {
    console.log('Service worker registration failed, error:', error);
  });
}
```

上述代码开始处对 `navigator.serviceWorker` 进行了浏览器支持性检测。然后使用 `navigator.serviceWorker.register` 进行了注册，如果注册成功，那么将返回一个 resolve 状态的 promise。service worker 的 `scope` 记录在 `registration.scope`。

`scope` 代表 service worker 可以控制哪些文件，也就是说，service worker 可以控制哪些路径下的网络请求。默认路径是 service worker 文件所在的位置，以及其中所包含的所有路径。所以 __service-worker.js__ 如果在根目录下，那么 service worker 可以控制这个域下的所有请求。

另外，还可以通过传递一个参数来指定，例如：
```javascript
// main.js
navigator.serviceWorker.register('/service-worker.js', {
  scope: '/app/'
});
```

在这个例子中，service worker 的作用域被设置为 `/app/`，所以这就意味着 service worker 可以控制请求来自 `/app/`，`/app/lower/` 或 `/app/lower/lower`，但是来自更高级别的目录则不行，如`/app` 或 `/`。

如果 service worker 已经被注册了，那么 `navigator.serviceWorker.register` 返回当前活跃的 service worker 注册对象。

## 安装

一旦浏览器注册了一个 service worker，那么 __安装__ 工作将会开始，只要 service worker 被浏览器认为是新的，即当前站点没有安装过 service worker，或新旧 service worker 之间有任何一字节的不同。

安装一个 service worker 会触发 `install` 事件。在 service worker 安装过程中可以通过 `install` 的事件监听器来开展一些额外的工作。例如，在安装过程中，service worker 可以预缓存 web app 的一部分内容，从而在用户下次打开的时候可以快速响应。所以在首次加载后，重复加载的资源的响应速度会很快，用户体验会变得更好。示例代码如下：
```javascript
// service-worker.js
// Listen for install event, set callback
self.addEventListener('install', function(event) {
    // Perform some task
});
```

## 激活

一旦 service worker 成功安装，那么将进入 __激活__ 阶段。如果有些页面是被旧的 service worker 控制，那么新的 service worker 会进入 `waiting` 状态。新的 service worker 只有等到没有页面被旧的 service worker 控制的时候才会被激活。这保证了任何时候只有一个版本的 service worker 在运行。

> 注意：简单地刷新页面是不足以将控制权转交给新的 service worker，因为在卸载当前页面之前，新页面会被请求，所以任何时间内旧的 service worker 都在被使用中。

当新 service worker 激活的时候，`activate` 事件会被触发。可以利用这个事件来清理过期的缓存。
```javascript
// service-worker.js
self.addEventListener('activate', function(event) {
  // Perform some task
});
```

一旦激活，service worker 会控制其作用域下的所有页面，并且开始监听这些页面的事件。然而，在激活之前加载的页面将不会受新的 service worker 的控制。新的 service worker 只有在关闭或重启应用，或调用 `clients.claim()` 才能掌握控制权。在此之前，这些页面的请求都将不会被新的 service worker 拦截。这其实是一种方法来保证站点前后统一。

# service worker 事件

service worker 是事件驱动的。安装和激活的时候会分别触发 `install` 和 `activate` 事件，此外还有其他事件 Service Worker 也能进行响应，比如 `fetch`、`push`、`sync` 和 `message` 等，其中 `message` 是 service worker 用来和其他脚本通信用的。

要想检查 service worker，可以在浏览器的开发者工具中查看。这个过程由于各浏览器的实现不同而可能有所差异。更多信息可查看[工具使用链接](https://developers.google.com/web/ilt/pwa/tools-for-pwa-developers/)

# 深入阅读
- service worker [生命周期的详细介绍](https://developers.google.com/web/fundamentals/instant-and-offline/service-worker/lifecycle)
- service worker [注册](https://developers.google.com/web/fundamentals/instant-and-offline/service-worker/registration)
- [Create your own service worker](https://developers.google.com/web/ilt/pwa/lab-scripting-the-service-worker)
- [Take a blog site offline](https://developers.google.com/web/ilt/pwa/lab-offline-quickstart)
- [Cache files with Service Worker](https://developers.google.com/web/ilt/pwa/lab-caching-files-with-service-worker)
