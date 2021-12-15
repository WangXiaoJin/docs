## ngx_http_random_index_module介绍

##### `【原创】`
---

ngx_http_random_index_module模块处理以（`/`）结尾的请求，随机选择该目录下任意文件作为index file。这个模块在`ngx_http_index_module`之前执行。开启该功能，需要在编译时增加`--with-http_random_index_module`参数。

#### 例子:

```nginx
location / {
    random_index on;
}
```

```nginx
Syntax: 	random_index on | off;
Default: 	random_index off;
Context: 	location
```
