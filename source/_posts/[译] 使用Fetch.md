---
title: 使用Fetch [译]
date: 2018-10-06 18:01:02
categories: translation
tags: [translation]
---

> [原文链接](https://developers.google.com/web/ilt/pwa/working-with-the-fetch-api)

# fetch 是什么

Fetch API 是一个简单的获取资源的接口。比起以前的 XMLHttpRequest 经常需要额外的逻辑处理（例如处理重定向），Fetch 使得发起网络请求和处理响应变得更简单。
<!-- more -->

> 注意：Fetch 支持 CORS。测试工作通常需要跑在本地服务器上。注意到 fetch 并不需要 https，但是 service worker 是需要的，所以在 service worker 中使用 fetch 是需要 https 的。而本地服务器测试的时候是可以免除这个的。

可以在 window 接口中检查浏览器是否支持 fetch。例如：
```javascript
if (!('fetch' in window)) {
  console.log('Fetch API not found, try including the polyfill');
  return;
}
// We can safely use fetch from now on
```

这有个 [ployfill](https://github.com/github/fetch) 适用于还不支持 fetch 的浏览器（注意 readme 中的注意事项）。

# 发出请求

下面是一个简单的例子：
```javascript
// main.js
fetch('examples/example.json')
.then(function(response) {
  // Do stuff with the response
})
.catch(function(error) {
  console.log('Looks like there was a problem: \n', error);
});
```

我们将想要获取的资源路径作为一个参数传递给了 fetch。在这个例子里是 __examples/example.json__ 。fetch 调用返回一个解析为响应对象的 promise。

但 promise 成功返回时，响应值会传递给 `.then`。这正是响应值会被使用的地方。如果请求没有完成，promise 会被 `.catch` 接管并传递相应的错误。

响应对象代表了每个请求对应的响应内容。它包含了请求的资源、丰富的属性和方法等。例如，`response.ok`，`response.status`，和 `response.statusText` 都可以被用来评定响应的状态。

评定响应的成功对于使用 fetch 来说是十分重要的，因为失败的响应（如404）也会被当做成功返回。fetch 只有在请求无法完成时才会返回一个失败的 promise。前述的代码片段只会在没有网络连接的情况下才会进入 `.catch`，失败的响应（如404）则不会进入。如果为了校验响应更新代码如下：

```javascript
// main.js
fetch('examples/example.json')
.then(function(response) {
  if (!response.ok) {
    throw Error(response.statusText);
  }
  // Do stuff with the response
})
.catch(function(error) {
  console.log('Looks like there was a problem: \n', error);
});
```

现在如果响应对象的 `ok` 属性是 false （状态码不是 200 - 299），这里的处理函数会抛出一个带有 `response.statusText` 的错误，从而触发代码块 `.catch` 。这可以防止失败的响应沿着 fetch 链继续传播。

# 读取响应对象

一定要读取响应对象来获取响应内容。响应对象正好有对应的方法。例如，`Response.json()` 读取响应并返回一个成功解析为 JSON 的 promise。将这一步的代码加入上述例子：

```javascript
// main.js
fetch('examples/example.json')
.then(function(response) {
  if (!response.ok) {
    throw Error(response.statusText);
  }
  // Read the response as json.
  return response.json();
})
.then(function(responseAsJson) {
  // Do stuff with the JSON
  console.log(responseAsJson);
})
.catch(function(error) {
  console.log('Looks like there was a problem: \n', error);
});
```

如果将功能抽象成函数，那将更加简洁，便于理解。

```javascript
// main.js
function logResult(result) {
  console.log(result);
}

function logError(error) {
  console.log('Looks like there was a problem: \n', error);
}

function validateResponse(response) {
  if (!response.ok) {
    throw Error(response.statusText);
  }
  return response;
}

function readResponseAsJSON(response) {
  return response.json();
}

function fetchJSON(pathToResource) {
  fetch(pathToResource) // 1
  .then(validateResponse) // 2
  .then(readResponseAsJSON) // 3
  .then(logResult) // 4
  .catch(logError);
}

fetchJSON('examples/example.json');
```

这段代码的意义如下：
1. Fetch 获取的资源是 __examples/example.json__ 。Fetch 返回一个成功解析为响应对象的 promise。响应结果成功返回时会被传递给 `validateResponse`。
2. `validateResponse` 检查响应是否有效。如果不是，则会抛出一个错误，这将跳过剩余的 `then` ，直接触发 `catch`。这是非常重要的。没有这个校验的话，失败的响应会被传递下去，这可能导致后续依赖于正确响应结果的代码出错。如果响应结果正确，那么它将会传递给 `readResponseAsJSON`。
3. `readResponseAsJSON` 使用 `Response.json()` 获取响应内容。这个方法返回一个成功解析为 JSON 的 promise。一旦这个 promise 状态变为成功，那么 JSON 数据就会被传递给 `logResult`。（如果 promise 失败了会怎样呢？）
4. 最后，来自于 __examples/example.json__ 的 JSON 数据被 `logResult` 记录下来。

> 注意：你也可以使用响应对象的 `status` 属性来处理任意一种网络状态码。这可以让你对不同的错误类型定制不同的页面，或者处理一些不成功（200 - 299）但是仍然可用（300 范围内的状态）的响应。查看 [使用 service worker 缓存文件](https://developers.google.com/web/ilt/pwa/caching-files-with-service-worker#generic-fallback) 获取使用 404 自定义页面的内容。

更多信息请查看：
- [Response interface](https://developer.mozilla.org/en-US/docs/Web/API/Response)
- [Response.json()](https://developer.mozilla.org/en-US/docs/Web/API/Body/json)
- [Promise chaining](https://developers.google.com/web/ilt/pwa/working-with-promises#chaining)

## 例子：获取图片

```javascript
function readResponseAsBlob(response) {
  return response.blob();
}

function showImage(responseAsBlob) {
  // Assuming the DOM has a div with id 'container'
  var container = document.getElementById('container');
  var imgElem = document.createElement('img');
  container.appendChild(imgElem);
  var imgUrl = URL.createObjectURL(responseAsBlob);
  imgElem.src = imgUrl;
}

function fetchImage(pathToResource) {
  fetch(pathToResource)
  .then(validateResponse)
  .then(readResponseAsBlob)
  .then(showImage)
  .catch(logError);
}

fetchImage('examples/kitten.jpg');
```

在这个例子中，获取的图片是 __examples/kitten.jpg__ 。在上述例子中，`validateResponse` 是用来校验响应的。响应被读取为 Blob 对象（而不是 JSON ）。然后创建了一个图片元素并添加到了页面中，图片的 `src` 属性被设置为一个 URL 来代表这个 Blob。

> 注意：URL 对象的方法 `createObjectURL()` 是用来产生一个 URL 以替代 Blob。这是非常重要的，因为你不能直接将图片的源数据设置为 Blob。Blob 对象必须转化成 URL。

更多信息请查看：
- [Blobs](https://developer.mozilla.org/en-US/docs/Web/API/Blob)
- [Response.blob()](https://developer.mozilla.org/en-US/docs/Web/API/Body/blob)
- [URL object](https://developer.mozilla.org/en-US/docs/Web/API/URL)

## 例子：获取文本

```javascript
function readResponseAsText(response) {
  return response.text();
}

function showText(responseAsText) {
  // Assuming the DOM has a div with id 'message'
  var message = document.getElementById('message');
  message.textContent = responseAsText;
}

function fetchText(pathToResource) {
  fetch(pathToResource)
  .then(validateResponse)
  .then(readResponseAsText)
  .then(showText)
  .catch(logError);
}

fetchText('examples/words.txt');
```

在这个例子中，获取的文本是 __examples/words.txt__ 。在上述例子中，`validateResponse` 是用来校验响应的。响应内容被处理成文本，然后加入到网页中。

> 注意：或许可以传输 HTML 代码，然后使用 `innerHTML` 属性将代码插入文档中，不过这种处理方式需要格外小心，因为这可能导致你的网站陷入跨站脚本攻击的危险中。

更多信息请查看：
- [Response.text()](https://developer.mozilla.org/en-US/docs/Web/API/Body/text)

> 注意：考虑到完整性，上面使用到的方法其实是 Body（ Fetch API mixin，在响应对象中实现了）的方法。

# 自定义请求

`fetch()` 也可以接受第二个可选的参数，`init`，这个参数可以让你创建一个自定义的请求，例如请求方法，缓存模式和证书等等。

## 例子：HEAD 请求

fetch 默认使用 get 请求来获取特定资源，但是其他 http 请求方法也是可以被使用。

HEAD 请求和 get 类似，不过响应体的内容是空的。当你想要文件的元数据时，你可以使用此类请求，并且你希望或需要传输文件的数据。

如果想要在 fetch 使用 HEAD 请求，例子如下：
```javascript
fetch('examples/words.txt', {
  method: 'HEAD'
})
```
上述代码会对 __examples/words.txt__ 发出一个 HEAD 请求。你可以使用 HEAD 请求来检查资源大小。例如：
```javascript
function checkSize(response) {
  var size = response.headers.get('content-length');
  // Do stuff based on response size
}

function headRequest(pathToResource) {
  fetch(pathToResource, {
    method: 'HEAD'
  })
  .then(validateResponse)
  .then(checkSize)
  // ...
  .catch(logError);
}

headRequest('examples/words.txt');
```

HEAD 请求在这里被用于请求一个资源的大小（ header 中的 __content-length__ )，在不用加载资源的情况下完成这个操作。在实践中这经常用来决定是否加载全部的资源（甚至是如何去发起请求）。

## 例子：POST 请求

Fetch 同样可以使用 POST 请求。下面的代码将给 __someurl/comment__ 发送字段 “title” 和 “message” （都是字符串）：
```javascript
fetch('someurl/comment', {
  method: 'POST',
  body: 'title=hello&message=world'
})
```
> 注意：在生产环境中，总是记得去加密敏感的用户信息。

这个方法再次使用了 `init` 参数。这也是设置请求体的地方，代表了需要发送的内容（上例中是 title 和 message ）。

请求体的内容也可以从表单中使用 FormData 接口提取出来。例如，上述代码可以更新为：
```javascript
// Assuming an HTML <form> with id of 'myForm'
fetch('someurl/comment', {
  method: 'POST',
  body: new FormData(document.getElementById('myForm')
})
```

## 自定义 header

`init` 参数可以使用 Headers 接口来对 HTTP 请求和响应头执行各种操作，包括检索，设置，添加和删除它们。在前一小节中已经展示了如何读取响应头。下述代码将描述如何创建一个自定义的 Headers 并在 fetch 中使用：
```javascript
var myHeaders = new Headers({
  'Content-Type': 'text/plain',
  'X-Custom-Header': 'hello world'
});

fetch('/someurl', {
  headers: myHeaders
});
```

在这里，我们创建了一个 Headers 对象，其中 `Content-Type` 的值为 `text/plain`，自定义 `X-Custom-Header` 值为 `hello world`。

> 注意：只有部分头部，类似 __Content-Type__ 是可以被修改的。而其他的，比如 __Content-Length__ 和 __Origin__ 是被保护的，不能被修改。

在跨域请求上的自定义头部是需要资源服务器支持的。这个例子中服务器需要配置成接受 `X-Custom-Header` 来使得 fetch 成功。当一个自定义的头部被设置的时候，浏览器会执行一个预检操作。这也就是说，浏览器一开始会给服务器发送一个 `OPTIONS` 请求，以查看浏览器允许哪些 HTTP 方法和头部。如果服务器被配置成接受原始请求的方法和头部，那么请求可以被发送。否则会报错。

更多信息请查看：
- [Headers](https://developer.mozilla.org/en-US/docs/Web/API/Headers)
- [Preflight checks](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request)

# 跨域请求

Fetch 和 XMLHttpRequest 都只是支持同源策略。这就意味着浏览器限制了脚本中的跨域请求。当一个域名（例如 __http://foo.com/__ )向另一个不同的域名（例如： __http://bar.com/__ ）发起资源请求时，跨域就发生了。这是一个简单的跨域例子：
```javascript
// From http://foo.com/
fetch('http://bar.com/data.json')
.then(function(response) {
  // Do something with response
});
```
> 注意：跨域请求限制总是让人困惑。很多资源比如图片、样式表和 js 脚本都是通过跨域获取的。然而，这些是同源策略的例外。跨域请求在脚本内部还是被限制的。

一直以来都有突破同源策略的方法（如 JSONP ）。跨域资源共享（ CORS ）机制是获取跨域资源的标准方法。CORS 机制可以让你在请求中说明你想要的是一个跨域的资源（ fetch 是默认开启的）。浏览器会给请求加上一个 header `Origin` ，然后去请求合适的资源。只有当服务器返回 `Access-Control-Allow-Origin` header 时，浏览器才能做出响应，即此时浏览器有权访问这个资源。在实践中，期望大量第三方（如第三方 APIs ）使用他们资源的服务器会对 `Access-Control-Allow-Origin` 设置一个通配符，从而让任何人都可以请求那个资源。


如果你请求的服务器不支持 CORS ，那么在浏览器的控制台应该可以看到一个错误。这个错误提示说跨域请求被阻止，因为 CORS 的 `Access-Control-Allow-Origin` header 缺失。

你可以使用 `no-cors` 模式来请求不透明的资源。不透明响应不能被 JavaScript 利用，但是依然可以被 service worker 处理和缓存。在 fetch 中使用 `no-cors` 模式是相对简单的。上述例子更新一下 `no-cors` 的用法就是如下所示：
```javascript
// From http://foo.com/
fetch('http://bar.com/data.json', {
  mode: 'no-cors' // 'cors' by default
})
.then(function(response) {
  // Do something with response
});
```

更多信息请查看：
- [Cross Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)

















