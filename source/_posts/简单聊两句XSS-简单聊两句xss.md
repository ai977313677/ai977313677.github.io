---
title: 简单聊两句XSS
date: 2020-12-27 16:28:06.838
updated: 2022-03-14 20:59:55.186
url: https://iachieveall.com/archives/简单聊两句xss
categories: 
- 前端基础
tags: 
- xss
- 安全
---

> XSS（跨站脚本攻击），聊两句，五毛的。

#### XSS的危害：

* 窃取Cookie，盗用用户身份信息

这玩意儿是大多数XSS的目标，也好解决，可以先治个标，直接设置`HttpOnly=true` ，即不允许客户端脚本访问，设置完成后，通过js去读取cookie，你会发现`document.cookie` 无法读取到被标识为HttpOnly的Cookie内容了。

* 配合其他漏洞，如CSRF（跨站请求伪造）

这个其实就没那么好解决了，因为XSS利用用户身份构造的请求其实对于服务端来说是合法的。比如说咱在B站上传了一条视频，发现没几个人点赞，于是动了歪心思，打开控制台找到了投币点赞的接口，然后拿到了对应的请求参数。自己构造了一条投币请求，然后诱导其他人点击含有这个脚本的页面为咱的视频投币，这样就完成了一套攻击流程。

> 不用尝试了，没用的。别问我怎么知道的 =。=。
>
> 要是没做校验的话，那这就是一个高危漏洞，还传啥视频啊，赶紧发邮件给阿B领赏金去啊。

* 广告

只要能发起XSS，我就能往页面里插广告，啥权限都不要，但是能引发这个问题的原因主要有两个。

1. XSS。
2. 用户自己安装外部脚本。

> 使用外部脚本一定要保证脚本来源的可信性，脚本的安全性。如果脚本是恶意的，那么他所能做的可就不只是弹弹广告这么简单了，替换个按钮，诱导点击钓鱼页面，替换某一条搜索结果，这都是可能的。

#### XSS扫描及防范

XSS风险有些是可以通过code review发现的，比如：

```js
let result = document.getElementById('test');
result.innerHTML = await getInfo();
```

这段代码很容易看到风险位置——`innerHTML` ，如果后端返回的数据中包含恶意的代码片段，那么就能够被攻击。所以在使用Vue和React框架时，需要评估是否真的需要使用`v-html` 和`dangerouslySetInnerHTML` 功能，在前端的render（渲染）阶段就避免`innerHTML` 和 `outerHTML` [^1]。

> 如果不使用框架，那就避免直接使用`innerHTML` 就好了。

至于review时无法发现的风险，那就交给扫描器吧。

防范XSS，除了少使用、不使用`innerHTML` 外，还可以设置严格的CSP[^2]，限制用户的输入长度。

> XSS是一个安全问题，它不只是前端的职责，这也是所有RD和QA的职责。
>
> 前端过滤用户输入后发给后端，后端如果不做处理存入数据库，那么这就是一个攻击点：直接抓前端的包，重新组装一下参数，发给后端，完成存储型XSS第一步，用户再访问这部分内容，就完成了一次XSS。
>
> QA的总能搞出来一些奇奇怪怪的payload（亦称测试用例），这些可能都是RD未能考虑到的方面。

附一段白名单过滤用户输入的代码，点击[GitHub](https://github.com/ai977313677/blog/blob/master/snippet/xssFilter.js)查看。

[^1]: [如何防止xss攻击](https://tech.meituan.com/2018/09/27/fe-security.html)
[^2]: [内容安全策略](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)
