## ngx_http_index_module介绍

##### `【原创】`
---

ngx_http_index_module模块处理以（`/`）结尾的请求。这些请求也能被`ngx_http_autoindex_module`和`ngx_http_random_index_module`处理。注：三个模块的处理顺序是 `ngx_http_random_index_module` --> `ngx_http_index_module` --> `ngx_http_autoindex_module`

#### 例子:

```nginx
location / {
    index index.$geo.html index.html index.htm;
}
```

```nginx
Syntax: 	index file ...;
Default: 	index index.html;
Context: 	http, server, location
```

index指令的值是按照顺序查找的。index最后一个参数可以使用绝对路径：`/index.html`
```nginx
location /test {
    index index.htm /index.html;
}
```

**注：**index指令实际上会产生一个内部跳转请求（`internal redirect`），`internal redirect`可以跳转到另一个`location`。例：

```nginx
location = / {
	default_type text/plain;
	index index_eq.html;
	#echo deng;
}
location / {
	default_type text/plain;
	#index index.html;
	echo xiegang;
}
```
上面的例子会返回`xiegang`，而不会返回index_eq.html页面。分析：当请求`http://localhost`时首先会被`location = /`捕获到，然后请求`/index_eq.html`，`/index_eq.html`的请求会被`location /`捕捉到，此时返回`xiegang`内容。