## ngx_http_autoindex_module介绍

##### `【原创】`
---

ngx_http_autoindex_module模块处理以（`/`）结尾的请求，显示该目录下的文件列表。当`ngx_http_index_module`没有找到index file时，才会执行此功能。

#### 例子:

```nginx
location / {
    autoindex on;
    autoindex_exact_size off;
}
```

命令参考手册
---

```nginx
Syntax: 	autoindex on | off;
Default: 	autoindex off;
Context: 	http, server, location
```
开启、关闭文件列表功能

```nginx
Syntax: 	autoindex_exact_size on | off;
Default: 	autoindex_exact_size on;
Context: 	http, server, location
```
供`autoindex_format=html`使用。指定文件是否以`B`准备的显示文件大小。当关闭此功能后，文件将会转换成近似的`KB`/`MB`/`GB`格式。

```nginx
Syntax: 	autoindex_format html | xml | json | jsonp;
Default: 	autoindex_format html;
Context: 	http, server, location
#This directive appeared in version 1.7.9. 
```
设置目录列表的格式。当指定jsonp时需要传有callback参数，不指定callback参数或为空值时将被当做json格式。当你想要输出xml格式时可以考虑使用`ngx_http_xslt_module`模块。

```nginx
Syntax: 	autoindex_localtime on | off;
Default: 	autoindex_localtime off;
Context: 	http, server, location
```
供`autoindex_format=html`使用。确定文件的时间属性是以`本地时间格式`还是`UTC`显示。

