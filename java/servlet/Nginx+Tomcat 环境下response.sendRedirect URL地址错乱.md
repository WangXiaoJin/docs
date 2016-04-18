## Nginx+Tomcat 环境下response.sendRedirect URL地址错乱


今天发现一个非常诡异的现象。我在测试服务器上部署了一个单点登录系统（CAS），CAS的端口号为8080。然后用Nginx反向代理这个CAS服务，配置如下：

```bash
server {
    listen        80;
    server_name   sso.xxx.com;
    index         index.html index.htm index.jhtml index.jsp;
    root          /data/webroot/cas/;
    location / {
        index               index.html;
		proxy_set_header    Host $host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_pass          http://localhost:8080;
    }
}
```

然后我在本机部署了一个项目，这个项目用到了CAS且端口也设置为8080，访问的地址为`http://localhost:8080/pro`。登录的地址为：`http://sso.xxx.com/login?service=http://localhost:8080/pro/login`。
正常来说，当登录成功后浏览器会302到`http://localhost:8080/pro/login?ticket=ST`。但是实际上302的重定向地址竟然改为：`http://sso.xxx.com/pro/login?ticket=ST`，就是说sso.xxx.com替换掉了localhost:8080。

当时我以为是CAS的代码写的有问题，看了下CAS的源码发现根本就没问题，CAS的redirect地址明明就是`http://localhost:8080/pro/login?ticket=ST`啊。
怎么就会无缘无故变成`http://sso.xxx.com/pro/login?ticket=ST`呢？

搞不清状况下，我把本地的项目端口号改为8081，然后抱着侥幸心理登录，妹的，这回竟然重定向正确了。难道是Nginx的中`proxy_pass http://localhost:8080`配置在作祟？
然后我试着不用域名访问CAS，改成用IP地址访问CAS，即不走Nginx的反向代理，发现又重定向正确了。这什么鬼？？？Nginx + Tomcat的Redirect的内部原理到底是什么，希望有人能指点指点！！！

最后我又试了另一种情况，在Nginx新增代理了另外一个项目，配置为`proxy_pass http://localhost:8081`，然后本地项目的端口号也改成8081。主要是想测试最终会不会Redirect到服务器上`localhost:8081`这个项目中。最终测试结果是：一切都正常，CAS也OK，Redirect也正确。

最后只能自己修改服务上CAS服务的端口号为不常用的，要是有人知道其他的解决方案或者了解其原理的话，求指点。
