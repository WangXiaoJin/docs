# Web常用

## HTTP Caching

可缓存StatusCode：
* 200 OK
* 203 Non-Authoritative Information
* 204 No Content
* 206 Partial Content
* 300 Multiple Choices
* 301 Moved Permanently
* 404 Not Found
* 405 Method Not Allowed
* 410 Gone
* 414 URI Too Long
* 501 Not Implemented

可缓存HTTP Method：
* GET
* HEAD
* POST - 规范定义为可缓存，多数未实现

> 参考 - [Which HTTP status codes are cacheable?](https://cassiomolin.com/2016/09/09/which-http-status-codes-are-cacheable/)

* [HTTP 缓存 - MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching)
* [彻底弄懂浏览器缓存策略](https://www.jiqizhixin.com/articles/2020-07-24-12)

## HTTP详解 - MDN Web Doc -【重要】

* [参考文档](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)
* `headers`
* `CORS`
* `response status codes`
* `request methods`
* `redirects`
* `cookies`
* `content negotiation`
* `compression`
* `authentication`


## HTTP2

* [HTTP2官网](https://http2.github.io/)
* 规范
  * [HTTP/2](https://httpwg.org/specs/rfc7540.html) - RFC7540
  * [HPACK: Header Compression for HTTP/2](https://httpwg.org/specs/rfc7541.html) - RFC7541
* [HTTP/2 Frequently Asked Questions](https://http2.github.io/faq/)
* [实现HTTP2的组件](https://github.com/httpwg/http2-spec/wiki/Implementations)


## URI

* [Uniform Resource Identifier (URI): Generic Syntax](https://datatracker.ietf.org/doc/html/rfc3986#section-2.2) - URI规范
  * [保留字符](https://datatracker.ietf.org/doc/html/rfc3986#section-2.2) - `:/?#[]@!$&'()*+,;=` - 使用时必须转换为`%格式`

## Media Types

* [Media Types](https://www.iana.org/assignments/media-types/media-types.xhtml) - 最全的格式列表

## 参考链接

* [Web technology (MDN)](https://developer.mozilla.org/en-US/docs/Web) -【重要】Web 参考文献

* [Is JavaScript Single-Threaded?](https://www.red-gate.com/simple-talk/dotnet/asp-net/javascript-single-threaded/) - 讲解`Event loop`及JS单线程执行原理

* [URI Template](https://tools.ietf.org/html/rfc6570) - A URI Template is a compact sequence of characters for describing
a range of Uniform Resource Identifiers through variable expansion.

* [HTML规范](https://whatwg-cn.github.io/html/) - 官网翻译版