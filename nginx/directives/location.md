## location介绍

请求在执行`location`之前，会干以下事情

1. 解码被encoded(`"%XX"`)的内容
2. 转换相对路径，去掉路径中的`"."`、`".."`
3. 合并多个斜杠（`"/"`）为一个斜杠


命令参考手册
---

```
Syntax: 	location [ = | ~ | ~* | ^~ ] uri { ... }
            location @name { ... }
Default: 	—
Context: 	server, location
```
* **=** 精确匹配
* **~** 正则，区分大小写
* __~*__ 正则，不区分大小写
* **^~** 前缀匹配，普通字符窜

location的匹配顺序如下。下面的序号只是`匹配顺序`，不代表`优先级`：

1. 匹配有 `=` 修饰符的location（`= /v1`），匹配成功则跳出匹配，否则执行第2步骤
2. 依次匹配所有`非正则`location（`/v1`），记住最长匹配的那个location，如果最长匹配规则为`location ^~`，
则使用当前`location ^~`规则且跳过下面的匹配规则，否则执行第3步骤。注意：因为`location ^~ /v1` 也在`非正则location`这个范围。
3. 按照配置文件中的location顺序执行`正则匹配`，只要匹配成功就跳出匹配。如果整个配置文件的正则都试完了，就是没匹配成功，只能用第二步中`记住`的`非正则location`，然后跳出匹配。

上面说了很多的废话，来点更直接的表面现象（`优先级`）：  
`=` ==> `^~` ==> `第一个匹配成功正则` ==> `/xx（普通非正则）`

* `location = /api` 只匹配`/api`，不会匹配`/api/`、`/apixx`
* `location = /api/` 只匹配`/api/`，不会匹配`/api`、`/apixx`
* `location /api/` 只匹配`/api/`、`/api/xx`，不会匹配`/api`
* `location /api` 匹配`/api`、`/apixx`、`/api/`、`/api/xx`
* `location ^~ /api` 匹配以`/api`开头的URI，即匹配`/api`、`/apixx`、`/api/`、`/api/test`
* `location ~ /api` / `location ~ ^/api` 都是使用正则匹配以`/api`开头的URI
* `location ~ /api$` 匹配以`/api`结尾的URI
* `location ^~ /v1`属于`非正则匹配`；`location ~ ^/v1`属于`正则匹配`  
* `location ^~ /v1`和`location /v1`不能同时出现，否则报错：`duplicate location "/v1" in /data/nginx.conf`

> 注：Nginx版本为**0.7.1** - **0.8.41**，如果前缀匹配（`/xx`）没有`=`或`^~`修饰符，使用当前匹配规则，终止后面的匹配且不会执行正则匹配。