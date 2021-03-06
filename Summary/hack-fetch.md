# 「震惊」你可能需要一个假的 Fetch API

Fetch API 已经出现很久了，很多公司和个人都在鼓吹 Fetch 多么牛逼，这点必须要同意。

Fetch 使用来替代老掉牙的 XMLHttpRequest，XMLHttpRequest 在设计上有着很多缺陷，比如调用方式混乱，不注重分离设计的原则等等，所以后来才会有了类似 JQuery Ajax 之类的库出现。

首先先给出一个明确的观点，我不否认 Fetch 相反我认为是很优秀的，但是 Fetch API 整体用起来还是有一些不爽的，虽然得益于 Promise 的助攻，但是更多的缺陷也来自 Promise，所以本文就针对基于标准 Promise 实现的 Fetch 吐槽一下用起来的不爽。

## 简单回顾一下 Promise/A+ 规范

Promise 中文翻译「承若」，在异步世界里真的没有什么比承若更加重要了，因为真的不知道下一个出现的会是谁。

Promise 中值分成现在值和将来值两个部分，将来值正是我们所关心的，所以给 Promise 下一个简单定义就是：获取意料当中值。

在 Promise 中分成三个状态：

- *pending*：初始状态，未被 fulfilled 或者未被 rejected。
- *fulfilled*：处理成功
- *rejected*：处理失败

是的就是只有三种状态（坑点就在这里）

只提供了最简单的 API：

- [`Promise.all(iterable)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)
- [`Promise.race(iterable)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)
- [`Promise.reject(reason)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/reject)
- [`Promise.resolve(value)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve)

在 Promise 原型链上有两种方法：

- [`Promise.prototype.catch(onRejected)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch)
- [`Promise.prototype.then(onFulfilled, onRejected)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)

所以 Promise 总的来说只有三种状态，四个方法、两个原型方法，多么简单。

![](https://static.zhengxiaowai.cc/ipic/2017-03-26-promise-process.png)

> 以上内容来自 [MDN Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

## 没有 Timeout 机制的 Fetch

没有这个功能确实很蛋疼，当遇到网络不顺畅的时候，不能老是等待吧，这样太恶心了。

这个槽点还是在 Promise 本身上，由于只有三种状态，成功、挂起、失败，并没有取消啊，WTF？？？黑人问号？？

怎么办？弹药不够敌人来造，最大的敌人就是 Promise 本身。Promise 中有一个方法叫做 race，该方法一组 Promise 中只要有一个promise对象进入 FulFilled 或者 Rejected 状态的话，就会继续进行后面的处理。

So~，有了这种机制就可以造一个假的 Timeout 出来了。

```javascript
function hackFetch(url, timeout=10, params={}) {
  // 用 Promise 包装一个 timeout 的 reject
  var _abort = new Promise((resolve, reject) => {
    setTimeout(() => reject('abort promise'), timeout);
  })            
  var _fetch =  fetch(url, params);

  return Promise.race([_fetch, _abort])
}
```

实现的代码很简单，两个 Promise，一个是 timeout 、一个是 Fetch，对这样就完成了。

然后再来一个工厂方法，多创建几个，来尝试一下。

```javascript
// hackFetch 的工厂方法
function createHackFetch(url, timeout=10, params={}) {
	return () => {
      return hackFetch(url, timeout, params)
              .then(res => res.json())
              .then(json => textDOM.value += json.message + '\n')
              .catch(err => alert('fetch 超时'))
    }
}
```

> 实验使用的是 Express，实现了 4 个接口，分别 0，5，10，15 秒返回数据。
>
> 完整例子可以转到该项目的 [Repo](https://github.com/zhengxiaowai/hack-fetch)

![](https://static.zhengxiaowai.cc/ipic/2017-03-28-143345.jpg)

当我点击「测试15秒 timeout 的 fetch」过后的 10 秒，出现了 alert，中断了这次 hackPromise，没有在下面的 textarea 中添加获取到字符串。

事情并不会那么美好，确认完成这个 alert 以后。观察那个 `fifteen-delay` 的请求，它依然返回数据。

![](https://static.zhengxiaowai.cc/ipic/2017-03-28-143623.jpg)

此坑开始在于 Promise 本身没有 cancel 机制。通过 hack 出来的带有 Timeout 机制的 Fetch，只不过的骗过了自己，但是没有骗过了浏览器。

这种方法是很危险的行为。轻的来看结果是显示和实际情况不一致罢了，但是严重的来看，本不应该出现东西却出现了，确确实实是一个漏洞。

此法有解吗？目前来看前端无解，后端可以通过设置连接的 Timeout 时间来解决这个问题，Nginx 可以通过设置 `send_timeout` 来规定 Timeout 时间。

## 接口返回错误码，不抛异常的 Fetch

在 MDN 的 [Using Fetch]() 中有那么一段话：

>The Promise returned from `fetch()` **won’t reject on HTTP error status** even if the response is an HTTP 404 or 500. Instead, it will resolve normally (with `ok` status set to false), and it will only reject on network failure or if anything prevented the request from completing.

翻译过来就是：

从 `fetch()` 返回的 Promise **将不会拒绝HTTP错误状态**, 即使响应是一个 HTTP 404 或 500。相反，它会正常解决 (其中ok状态设置为false),  只有在网络故障时或者请求被阻止时，它才会拒绝。

这个其实这个相比上一个来说并不是什么严重的坑，只不过在开发上变的更加繁琐一些，这个恰恰又和 Fetch 的理念相悖。

```javascript
app.get('/api/error-five-delay', function(req, res) {
    res.type('json');
    res.status(500)
    setTimeout(() => {
        res.send(JSON.stringify({
            message: 'there is a error response'
        }));
    }, 5000)
});
```

添加一个 5 秒后返回 500 错误的接口，使用一个创建正常 Fetch 的工厂方法，绑定到 button 上。

```javascript
function createFetch(url, params={}) {
	return () => {
      return fetch(url, params)
              .then(res => res.json())
              .then(json => textDOM.value += json.message + '\n')
              .catch(err => alert('请求失败'))
    }
}

// 绑定事件
document.getElementById('error-five-fetch').onclick = createFetch('/api/error-five-delay');
```

5 秒之后，message 信息如愿的被添加到了 textarea 上。此时浏览器做到了它职责在控制台中给出了错误，但是 Promise 忽略了它。

![](https://static.zhengxiaowai.cc/ipic/2017-03-28-151745.jpg)

![](https://static.zhengxiaowai.cc/ipic/2017-03-28-151757.jpg)

所以如 MDN 中所言，我们必须手动的检查 response 中 ok 属性是否为 `false` ，好了要在造一个假的 Fetch 了。

```javascript
function xfetch(url, params) {
	return fetch(url, params)
    		// 处理错误时候的 json
            .then(res => res.json().then(json => res.ok ? json : Promise.reject(json)))
            .then(json => textDOM.value += json.message + '\n')
            .catch(err => alert(err.message))
}

function createXfetch(url, params={}) {
	return () => xfetch(url, params)
}

document.getElementById('error-handling-five-fetch').onclick = createXfetch('/api/error-five-delay');
```

网络上有很多这样处理发生错误时候的 json，各种各样的方法都有，其中一样的就是必须先把 json 从 Promise 从解析出来，然后再来处理 response.ok 的状态。

![](https://static.zhengxiaowai.cc/ipic/2017-03-28-155548.jpg)

> 完整例子可以转到该项目的 [Repo](https://github.com/zhengxiaowai/hack-fetch)

其实这么做面对 json 数据时候没有压力，但是对于需要解析多种数据时候还需要更多的参数和封装，比如数据来源是 xml 或者 plain。

好嘛，又违背了 Promise 的设计原则。

## 总结一下

文中没有实现一个 timeout 和 错误 json 处理例子，其实把 timeout 版中替换成 xfetch 就好了。

自从 Promise 的出现，在编写异步任务上有了很大的改进，Fetch 也孕育而生，在使用 Fetch 带来的简单、高效的同时也要主要它的坑点所在。本文只是总结了很小的一部分，在 Promise 还有无数的坑等着别去跳。

async / await 肯定是下一个方向，在还没完善之前，为了新老语法过渡使用 Promise 无疑是非常聪明的选择。可以给老代码以接口的方式打上一个 polyfill，同时新语法兼容 Promise 。这样完美的避开了像 Python 青黄不接的尴尬局面，Python 要加油了。

我是一个 Python 工程师，Python 大发好啊，Python 大发好啊，Python 大发好啊。

