## Cookie详解

---

参考文章：<http://curl.haxx.se/rfc/cookie_spec.html>, <https://www.nczonline.net/blog/2009/05/05/http-cookies-explained/>

　　本文类容摘自上面两篇文章。上两篇文章写得非常详细，有兴趣的可以看看原文。

### 1. Cookie语法

```
Set-Cookie: NAME=VALUE[; expires=DATE][; domain=DOMAIN][; path=PATH][; secure]
```
多个属性间用分号和空格隔开，方括号中为可选属性。

*  **NAME=VALUE**  
   `NAME=VALUE` 字符窜中不能出现分号、逗号、空格。如果需要空格等字符，推荐使用URL encode`NAME`或`VALUE`，转换成`%XX`格式。  
   
   等号（`=`）字符不是必须的，不写`=`照样可以存储Cookie值，照样能传给服务器。单`NAME=VALUE`已成业界规范，有些但三方框架解析值时需要用到`=`字符。
   
   **此属性为唯一必填属性**
   
*  **expires=DATE**  
   此属性指明了Cookie的有效期。DATE唯一合法格式为GMT：`Wdy, DD-Mon-YYYY HH:MM:SS GMT`。当DATE为过去时间时，Cookie会被删除。
   
   可选属性，**默认值：当前Session有效**。当浏览器关闭后，Cookie就失效了。
   
*  **domain=DOMAIN**  
   设置Cookie的域名。当搜索有效的Cookie时，会拿Cookie的domain属性和当前URL请求的host比较（从末尾开始比较），当`domain = host`或host以domain结尾则匹配成功。例：`domain=abc.com`的Cookie能被`abc.com`、`shop.abc.com`、`backend.abc.com`的页面获取到。
   
   domain属性值必须包含特有的标识（`abc.com`），不能设为`com`、`va.us`等。
   
   默认值：**创建Cookie的当前Response所在域名**
   
*  **path=PATH**  
   设置指定PATH及其子路径能访问的Cookie。如果一个Cookie的domain属性已匹配，则开始匹配它的path属性，匹配规则是从头开始匹配，匹配成功则此Cookie会随着请求一起被发送到服务器。例：`path=/user`会成功匹配`/user`、`/usershop`、`/user/add.html`。
   
   `path=/`是最常用的路径。
   
   默认值：**包含Cookie的URL地址中的path部分**
   
   > 注：在开发中启动项目时，URL地址会经常包含项目名，此项目名也算在path属性里面

*  **secure**  
   如果Cookie中包含secure标识，则表明此Cookie只有在HTTPS的情况下才会被使用。
   
   默认情况下，在HTTPS中的Cookie会自动被设置为secure

### 2. HTTP Request Header中Cookie格式

```
Cookie: value1; name1=value1; name2=value2 ...
```

当客户端发送请求给服务器时，会搜索所有匹配的Cookie，包含进HTTP Request Header中，一起发送到服务器。多个值由分号和空格隔开。

### 3. Cookie的维护和生命周期

Cookie用四个属性来标识它的唯一性：`name`、`domain`、`path`、`secure`。为了以后修改某个Cookie的值，你只需要`Set-Cookie`一个Cookie值，必须确保这个Cookie的`name`、`domain`、`path`是一样的，不然你会发现以前的Cookie值没有改变，反而又新增了一个Cookie。

当你新增了两个Cookie，他们的区别是path不一样，例：

```
Set-Cookie: name=green; domain=abc.com; path=/

Set-Cookie: name=blue; domain=abc.com; path=/blog
```

当你请求 <http://www.abc.com/blog> 时，你会发现有两个Cookie：

```
Cookie: name=blue; name=green
```

Cookie的顺序是：按照`domain --> path --> secure`顺序，最精准匹配的排在前面。意思就是：先从domain开始，谁匹配最精确则谁排在前面。然后是匹配path，谁path匹配最精确谁排前面，最后是secure。

当你创建了一个Cookie使用了`expires`属性，此时你想修改这个Cookie值，你必须保证`name`、`domain`、`path`、`secure`的参数都一样，`expires`属性是不需要的，因为它不作为唯一标识使用。例：

```
Set-Cookie: name=Mike; expires=Sat, 03 May 2025 17:44:22 GMT
```

此Cookie的过期时间已设置，所以下次修改Cookie值时只需要如下方式：

```
Set-Cookie: name=Matt
```

执行上面语句后，Cookie的expires属性不会改变，更不会变成session cookie。实时上，expires属性是不会改变的，除非你手动的再次修改它。这也就意味着，一个session cookie能成为一个持久化的cookie，而一个持久化的cookie不能成为session cookie。如果你想让一个持久化的cookie变成一个session cookie，你必须先把持久化的cookie删除掉，然后在重新建一个session cookie。

**在一些情况下，会出现自动删除cookie：**
* 当session失效时，session cookie会被删除
* 当cookie到期后，持久化的cookie会被删除
* 当浏览器的cookie到达上限时，会主动删除不常用的cookie，用以存放新的cookie

**cookie限制**  
为了阻止滥用cookie，浏览器会设一些限制：
* 浏览器最多支持300个cookie
* 每个cookie最多4KB，超出的部分将会被截掉
* 每个server或domain最多20个cookie。（最多20个cookie是早期的浏览器限制，IE7、IE8最多50个，火狐最多50个，Opera最多30个，Safari和Chrome没有限制。**此数据为2009统计，可能现在有点不准。我也懒得去验证正确性，因为如果你使用的cookie太多话，那你的设计就出了问题。cookie数越多会越影响性能。**）


### 4. JavaScript中使用Cookie

你可以通过`document.cookie`来创建、修改、删除Cookie（当然最方便的是使用JS框架来操作）。此功能和`Set-Cookie`一样，操作Cookie的语法格式也是一样：

```javascript
document.cookie="name=Nicholas; domain=nczonline.net; path=/";
```

上面的语句不会删除之前就存在的Cookie，只是简单的创建或修改Cookie。获取Cookie值：

```javascript
var c = document.cookie;
```

### 5. HTTP-Only cookies
微软在Internet explorer 6 SP1引入了一个新的特性：HTTP-only cookies。就是说这个Cookie不能通过js的`document.cookie`获取。新增的这个特性是为了跨域脚本攻击。谷歌、火狐等浏览器都支持此特性。例：

```
Set-Cookie: name=Nicholas; HttpOnly
```

### 6. Nginx添加Cookie

```nginx
add_header Set-Cookie "name=wang";
```

### 7. SameSite cookies

如跨站请求报如下错误（A cookie associated with a cross-site resource at http://xxx.com/ was set without the `SameSite` attribute.
It has been blocked, as Chrome now only delivers cookies with cross-site requests if they are set with `SameSite=None` and `Secure`.）。
解决方案：
* 服务端设置Cookie时增加`SameSite=None; Secure`配置
* 禁用谷歌浏览器的`SameSite by default cookies` ： <chrome://flags/#same-site-by-default-cookies>

参考链接：
* [SameSite cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
* [Cookie 的 SameSite 属性](https://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html)
* [SameSite Cookie，防止 CSRF 攻击 - 紫云飞](https://www.cnblogs.com/ziyunfei/p/5637945.html)
* [SameSite - 了解篇](https://juejin.im/post/6844904090355367949)

> `SameSite`用于配置`跨站`而不是`跨域`
