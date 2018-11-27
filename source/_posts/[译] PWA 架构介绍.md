---
title: PWA 架构介绍 [译]
date: 2018-10-09 17:21:58
categories: translation
tags: [translation]
---

> [原文链接](https://developers.google.com/web/ilt/pwa/introduction-to-progressive-web-app-architectures)

渐进式 Web Apps （ PWA ）使用现代 Web 功能提供快速、具有用户粘性和可靠的移动网络体验，这对于用户和企业来说都是十分有益的。

这份文档描述了一些架构和技术，这些内容可以让你的应用支持离线可用、后台同步和推送通知。这些功能原本都是本地应用才能有的体验。

# service worker 和 application shell 造就的瞬间加载

PWA 融合了很多本地应用和 Web 的优点。PWA 从浏览器标签下的页面发展为沉浸式应用，仅仅是用了普通的 HTML 和 Javascript，并且进一步增强，为用户提供了一流的类似本地应用的体验。

即使用户离线或者网络不可靠的情况下，PWA 也能提供快速体验。另外，PWA 有很大潜力去融合之前只属于本地应用的功能，比如推送消息。开发一个离线可用且性能优异的 Web 应用主要依靠 service worker 和客户端存储 API （例如，Cache Storage API 或 IndexedDB ） 的混合使用。

__service woker__ ：得益于 service worker 可以使用缓存和存储 API ，所以 PWA 可以预缓存一部分 Web 应用的数据，从而在用户下次开启时可以快速加载。使用 service worker 可以让你的 Web 应用拦截或处理网络请求，包括管理各种缓存，最小化数据传输，和保存用户离线操作的数据直至重新上线。这样的缓存机制可以让开发者集中关注速度，使得本地应用上常见的瞬时加载和定期更新同样出现在 PWA 上。如果你不熟悉 service worker，请阅读之前的文章。

service worker 不需要用户操作，不需要在打开的页面中也能执行自身的功能。这就催生出了新服务，比如推送消息或离线采集用户操作然后上线再发送。（这不太可能使你的应用程序膨胀，因为浏览器会根据需要启动和停止 service worker 来管理内存）

service worker 提供的服务如下：
- 拦截 HTTP/HTTPS 请求，因此你的应用可以从缓存、本地存储或网络中做出决定来提供什么样的服务。

  service worker 不能接触 DOM 但是它可以使用 Cache Storage API，使用 Fetch API 来发送网络请求，使用 IndexedDB API 来做持久化存储。除去拦截网络请求，service worker 还可以使用 `postMessage()` 来和其他 service worker 和它控制的页面进行通信。（例如要求更新 DOM ）  

- 从服务器接受推送消息

  service worker 独立运行于其他 Web 应用之外，并提供底层操作系统的钩子。它能响应来自操作系统的事件，包括推送消息。

- 让用户在离线情况下依然可以使用 PWA 应用，直到浏览器恢复网络才执行一系列用户的操作。（这也就是后台同步）

  将 service worker 想象成应用的管家，被需要的时候就会醒过来，然后执行任务。实际上，service worker 是浏览器的后台事件处理程序。service worker 的生命周期比较短。它只会在接收到事件的时候醒来，然后只运行必要的时间去处理这个事件。

缓存的概念是令人兴奋的，因为这可以让用户体验离线操作，而且给了开发者足够的控制权去控制什么是真正的离线体验。但是，为了充分发挥 service worker 的优点并融入越来越多的 PWA 功能，是需要一个新思路，即利用 application shell 的架构来创建 Web 站点。

<!-- more -->

Application Shell (app shell)：PWA 倾向于围绕 app shell 进行架构的。这其中包含了一些本地资源，用于 Web 应用加载用户界面框架，以此来使得 PWA 离线可用并且使用 JavaScript 填充其内容。如果 app shell 已经被 service worker 缓存，那么重复访问 app shell 将可以在没有网络的情况下也能在屏幕上展示丰富的内容。在构建 PWA 应用时，使用 app shell 不是硬性要求，但是如果缓存正确，提供的服务恰当，那么它可以带来非常显著的性能提升。

__app shell 是什么__ 在下文将详细介绍如何使用 app shell，同样非常建议使用 app shell 来升级现有的单页应用（SPA)和构建 PWA。这个架构提供了连接弹性，它使得 PWA 的使用体验和原生应用类似，并提供了类似原生应用的交互和导航，还有可靠的性能。

> 注意：如果你的站点是一个模板站点（比如站点内容是使用各种模板混合实际文字、图片和其他资源），那么建议阅读 Jeffrey Posnick 的 [Offline-first for Your Templated Site](https://jeffy.info/2016/11/02/offline-first-for-your-templated-site-part-1.html) ，以此来了解为模板站点提供服务和缓存的不同策略。

service worker 的缓存应当被视为渐进增强的。如果你的 Web 应用遵守 service worker 只在浏览器支持的情况下再进行注册，那么你可以使用 service worker 体验离线可用的用户体验。如果浏览器不支持 service worker，那么离线可用的相关代码将永远不会被调用，因此也不会有任何开销或破坏。

## 核心概念

使用 app shell 依赖于利用 service worker 来缓存 Web 站点的 ‘shell’ 。使用 __app shell__ 和 __dynamic content model__ 大幅提升了应用的性能，并且配合使用 service worker 来作为一个渐进增强的功能点，体验效果非常好。渐进增强 Web 站点意味着站点可以添加功能，如离线缓存，推送通知和安装到桌面等功能。

下面简要概括下它是如何工作的：  
1. 当用户进入你的站点，基础的 HTML，JavaScript 和 CSS 开始展示。

  第一次进入页面的时候，页面注册 service worker 从而控制这个站点未来的导航。注册创建了一个新的 service worker 实例，然后触发了 service worker 会响应的事件 `install` 。当 service worker 注册完，app shell 的内容会被添加到缓存中。一旦注册，service worker 会控制站点上未来的导航。注意到你可以通过浏览器开发者工具或 service worker API 来手动激活新的 service worker。

2. 在 shell 内容加载完毕后，应用会请求内容来填充视口。app shell 加上动态内容就是完整呈现的页面。

  然后，SPA 会发起请求（比如通过 `XMLHttpRequest` 或 Fetch API），收到内容后会填充视口。每个请求都会触发 service worker内部的 `fetch` 事件，你可以选择任何方式处理它。一旦这些处理都完成了，那么 service worker 将会进入一个空闲状态。所以，service worker 会一直空闲直到一个新的网络请求触发一个新事件。作为网络请求的回应，`fetch` 事件处理函数可以根据你的设置随意拦截请求和响应。在 service worker 空闲一段时间之后，它就会自动停止，然后当页面再次加载并发生了下一个请求时，service worker 会重新启动并快速响应 `fetch` 事件。

![life circle](https://developers.google.com/web/ilt/pwa/img/c9f7d527c81ed1a1.png)

> 注意：对于给定版本的 service worker ，注册激活之后，如有必要，过往不再需要的过时缓存资源将会被清除。这对于给定版本的 service worker js 文件来说，只会发生一次。

__浏览器不支持 service worker 怎么办？__

app shell 模型是非常棒的，但是如果浏览器不支持 service worker 的话，那该怎么办呢？不用慌！Web 应用仍然可以被加载即使浏览器不支持 service worker。加载 UI 界面必须的东西是一样的（例如，HTML，CSS，JavaScript），不管 service worker 是否有被使用。service worker 只是将类似原生应用的特性加到了 Web 应用上而已。没有 service worker，Web 应用依然可以运行，只是将缓存资源变成了 HTTPS 请求。换句话说，service worker 不支持的时候，静态资源不是离线缓存的，而是通过网络获取，因此用户依然能有一个基本可用的版本。

## 组件

| 组件 | 说明 |
| ---- | ---- |
| app shell | 除去页面中的特定内容之外，页面结构需要的最少的 HTML，CSS，JavaScript 和任意其他静态资源。 |
| cache | 浏览器中有两种类型的缓存：浏览器自身管理的缓存和应用管理的缓存（service worker）。<br> <br> __浏览器管理的缓存__ 是电脑临时存储文件的地方，这些文件是浏览器下载下来用于展示站点用的，其中包括任何可以填充站点的内容，比如 HTML 文件，CSS 样式表，JavaScript 脚本，图片和其他多媒体文件。这种缓存是浏览器自动管理的，并且不是离线可用的。<br> <br> __应用管理的缓存__ 是使用 Cache API 创建的，独立于浏览器管理的缓存之外。这个 API 可以被应用（通过 window.caches）和 service worker 使用。应用管理的缓存可缓存的文件类型和浏览器一样，但它是可以离线使用的（通过 service worker 启用离线可用功能）。这种缓存是开发者管理的，在脚本中使用 Cache API 精确更新已命名的缓存对象中的某些内容。<br> <br> 缓存是你构建应用的好帮手，只要你的缓存对这些资源来说是合适的就行。在 PWA 缓存机制教程中有一些缓存策略可供查看。
| 客户端渲染 | 客户端渲染意味着浏览器中运行的 JavaScript 负责产出 HTML（大概率是通过模板）。这就可以让你在用户点击的时候瞬间更新界面，而不是等几百毫秒去询问服务器再更新内容。多数站点里的内容和静态资源都是可以不用服务器端渲染。只要页面中任何部分是用到动画或高交互行为的（可拖动的滑块，可排序的表格，可下拉的菜单），基本都是使用客户端渲染完成的。 |
| 动态内容 | 动态内容是指 Web 应用正常工作所需要的所有数据，图片和其他资源等，这些是独立于 app shell 之外的。尽管 app shell 倾向于快速填充站点的内容，但是用户可能也会期待动态内容的出现，因此你的应用必须去请求相应的资源来满足用户的需求。有时候，动态内容会取自另外的第三方 API ，当然也有一些自身的数据会动态产生或频繁更新。 |
| Fetch API | 你可以自己选择性实现 Fetch API 来帮助 service worker 获取数据。举个例子来说，如果你的 Web 应用主体是新闻，那么你就需要充分利用自己的 API 去获取最新的文章，然后用第三方 API 去获取当前天气。这两种请求类型的内容都是归属于动态内容下。 |
| 渐进增强 | 这是一种 Web 开发方式，首先针对普通浏览器特性进行开发，然后针对支持现代技术的浏览器会添加功能或增强功能。 |
| PWA 架构风格 | 指的是任意一种基于可用后端技术和性能需求来构建 PWA 的方法。这些模式包括使用 app shell，服务器端渲染，客户端渲染和其他等等。这些内容在 [PWA Architectural Patterns](https://developers.google.com/web/ilt/pwa/introduction-to-progressive-web-app-architectures#patterns) 有详细说明。 |
| 服务器端渲染（SSR） | 服务器端渲染指的是当浏览器导航至一个 URL 然后获取页面，浏览器立马取回描述页面的 HTML 。SSR非常好，因为页面加载更快了（这可以是整个页面的服务器渲染版本，app shell 或内容）。因此也就可以在下载渲染代码和解析代码的时候避免白屏的出现。如果渲染内容在服务器端，那么即使在不好的网络条件下，JavaScript文件资源还没有完全获取并解析，用户也可以在界面上得到有意义的文字。SSR 很好地维持了页面就是文档的概念，如果你使用 URL 向服务器询问一份文档，然后文档内容就会返回，而不是返回一个使用复杂 API 产生文字的程序。 |
| service worker | Web worker 的一种，和 Web 应用一起工作，它的生命周期是和 app 的事件执行是绑定的关系。有些 service 包含 JavaScript 写的网络代理，用来拦截网页发出的 HTTP/HTTPs 请求。它同样可以接收推送消息。另外的特性已经在计划中。 |
| sw-precache | 这个模块集成在构建过程中，并产生代码用来缓存和维持 app shell 中的所有资源。 |
| sw-toolbox | 这个工具库是 service worker 运行时加载的，并提供事先写好的工具，这些工具主要用于将常见的缓存策略应用于不同的 URL 模式。 |
| 通用 JavaScript 渲染 | 通用或同构 JavaScript 应用拥有客户端和服务器都能运行的代码。这就意味着一些代码逻辑可以同时跑在客户端和服务器。这也同样意味着拥有更好的性能去展现第一屏和更多状态信息。同构应用在路由（理想情况下，有一组唯一的路由将 URI 模式映射到路由处理程序），数据获取（这是指组件资源，不管获取机制是怎样的，它们都可以在服务器和客户端上呈现）和视图渲染（视图必须依照应用的需要既可以在服务器端渲染，也可以在客户端渲染）。 |
| Web app manifest | app shell 是通过 Web app manifest 部署的，这是个简单的 JSON 文件用来控制应用是如何启动并展现给用户的（这个一般被命名为 __manifest.json__ )。当 PWA 应用第一次连接网络的时候，浏览器就会读取 manifest 文件，下载指定的资源并保存到本地。然后如果离线没有网络连接，浏览器可以使用本地缓存来展示应用。<br><br> __注意：__ 不要和 appcache 使用的 __.manifest__ 文件混淆了。PWA 应该使用 service worker 来实现缓存，使用 Web app manifest 来启用 “添加至桌面” 和消息推送。 |














