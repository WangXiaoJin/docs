## proxy

#### proxy_pass

> 注：理解`proxy_pass`的重点在于`proxy_pass`后面的地址中是否包含`URI`，包含和不包含会出现两种不同的现象。

很多人对`proxy_pass`的功能理解有误，他们简单的认为`proxy_pass`的这种现象是由地址后面的是否包含斜杠引起的，其实不然。
其实`proxy_pass`是判断地址中是否包含`URI`而不是判断是否以`/`结尾。

**判断`proxy_pass`地址中是否包含`URI`依据：**
* `proxy_pass http://localhost:8070` 不包含URI
* `proxy_pass http://localhost:8070/` 包含URI，URI为`/`
* `proxy_pass http://localhost:8070/app` 包含URI，URI为`/app`
* `proxy_pass http://localhost:8070/app/` 包含URI，URI为`/app/`

**现象：**
* 当`proxy_pass`地址中包含`URI`，则配置的`location` URI会被`proxy_pass`中的`URI`替换掉。直接一点就是`location`的URI部分会被去除
* 当`proxy_pass`地址没有`URI`时，则配置的`location` URI会被追加至`proxy_pass`中

##### proxy_pass 例子

```nginx
location /api/ {
  proxy_pass http://localhost:8070/;
}
```

* `http://test.com/api` => 自动301跳转到 `http://test.com/api/` => `http://localhost:8070/` (后台获取的请求全路径为：`http://test.com/`)
* `http://test.com/api/` => `http://localhost:8070/` (后台获取的请求全路径为：`http://test.com/`)
* `http://test.com/api/user` => `http://localhost:8070/user` (后台获取的请求全路径为：`http://test.com/user`)

```nginx
location /api/ {
  proxy_pass http://localhost:8070/app/;
}
```

* `http://test.com/api` => 自动301跳转到 `http://test.com/api/` => `http://localhost:8070/app/` (后台获取的请求全路径为：`http://test.com/app/`)
* `http://test.com/api/` => `http://localhost:8070/app/` (后台获取的请求全路径为：`http://test.com/app/`)
* `http://test.com/api/user` => `http://localhost:8070/app/user` (后台获取的请求全路径为：`http://test.com/app/user`)

```nginx
location /api/ {
  proxy_pass http://localhost:8070;
}
```

* `http://test.com/api` => 自动301跳转到 `http://test.com/api/` => `http://localhost:8070/api/` (后台获取的请求全路径为：`http://test.com/api/`)
* `http://test.com/api/` => `http://localhost:8070/api/` (后台获取的请求全路径为：`http://test.com/api/`)
* `http://test.com/api/user` => `http://localhost:8070/api/user` (后台获取的请求全路径为：`http://test.com/api/user`)

```nginx
location /api/ {
  proxy_pass http://localhost:8070/app;
}
```

* `http://test.com/api` => 自动301跳转到 `http://test.com/api/` => `http://localhost:8070/app` (后台获取的请求全路径为：`http://test.com/app`)
* `http://test.com/api/` => `http://localhost:8070/app` (后台获取的请求全路径为：`http://test.com/app`)
* 注：`http://test.com/api/user` => `http://localhost:8070/apiuser` (后台获取的请求全路径为：`http://test.com/apiuser`)
    

    
```nginx
location /api {
  proxy_pass http://localhost:8070/;
}
```

* `http://test.com/api` => 自动301跳转到 `http://test.com/api/` => `http://localhost:8070//` (后台获取的请求全路径为：`http://test.com//`)
* `http://test.com/api/` => `http://localhost:8070//` (后台获取的请求全路径为：`http://test.com//`)
* `http://test.com/api/user` => `http://localhost:8070//user` (后台获取的请求全路径为：`http://test.com//user`)
    

```nginx
location /api {
  proxy_pass http://localhost:8070/app/;
}
```

* `http://test.com/api` => 自动301跳转到 `http://test.com/api/` => `http://localhost:8070/app//` (后台获取的请求全路径为：`http://test.com/app//`)
* `http://test.com/api/` => `http://localhost:8070/app//` (后台获取的请求全路径为：`http://test.com/app//`)
* `http://test.com/api/user` => `http://localhost:8070/app//user` (后台获取的请求全路径为：`http://test.com/app//user`)

```nginx
location /api {
  proxy_pass http://localhost:8070;
}
```

* `http://test.com/api` => 自动301跳转到 `http://test.com/api/` => `http://localhost:8070/api/` (后台获取的请求全路径为：`http://test.com/api/`)
* `http://test.com/api/` => `http://localhost:8070/api/` (后台获取的请求全路径为：`http://test.com/api/`)
* `http://test.com/api/user` => `http://localhost:8070/api/user` (后台获取的请求全路径为：`http://test.com/api/user`)

```nginx
location /api {
  proxy_pass http://localhost:8070/app;
}
```

* `http://test.com/api` => 自动301跳转到 `http://test.com/api/` => `http://localhost:8070/app/` (后台获取的请求全路径为：`http://test.com/app/`)
* `http://test.com/api/` => `http://localhost:8070/app/` (后台获取的请求全路径为：`http://test.com/app/`)
* 注：`http://test.com/api/user` => `http://localhost:8070/api/user` (后台获取的请求全路径为：`http://test.com/api/user`)
    