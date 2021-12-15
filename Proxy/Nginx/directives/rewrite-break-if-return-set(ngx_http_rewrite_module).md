## `rewrite` / `break` / `if` / `return` / `set` (ngx_http_rewrite_module)

### ngx_http_rewrite_module

rewrite模块使用`PCRE`正则表达式改变请求`URI`，包含下面几个指令：
* break
* if
* return
* rewrite
* rewrite_log
* set
* uninitialized_variable_warn

`break`、`if`、`return`、`rewrite`和`set`指令执行顺序：
* 在`server`级别按照指令的出现的顺序执行
* 以下为重复执行：
    * 根据Request URI匹配`location`
    * 匹配到的`location`中出现的rewrite模块指令按顺序执行
    * 如果request URI被`rewritten`则循环执行，但不能超过10次，否则返回500 code

### 指令

#### break - 停止执行`rewrite模块`的所有指令，不影响break后面的非rewrite命令执行

```
Syntax:	break;
Default:	—
Context:	server, location, if
```

#### if

```
Syntax:	if (condition) { ... }
Default:	—
Context:	server, location
```

if判断条件：
* 变量名：如果此变量值为`空字符窜`或`"0"`，则为`false`
    > 在`1.0.1`版本之前，以`"0"`开头的字符窜也被视为false
* 使用`=`或`!=`比较变量
* 使用正则表达式比较变量。正则表达式可以使用捕获组功能，便于后面使用`$1..$9`。
如果表达式包含`}`、`;`字符，则整个表达式必须使用`单引号`或`双引号`。
    * `~` - 大小写敏感匹配
    * `~*` - 大小写不敏感匹配
    * `!~` - 大小写敏感不匹配
    * `!~*` - 大小写不敏感不匹配
* 使用`-f`、`!-f`判断`文件`是否存在
* 使用`-d`、`!-d`判断`目录`是否存在
* 使用`-e`、`!-e`判断`文件`、`目录`、`超链接`是否存在
* 使用`-x`、`!-x`判断是否是可执行文件

if指令一定要慎用。下面列出特殊情况，此特殊情况是由于`if指令`和`Nginx分段执行`导致的：

```nginx
location /foo {
  add_header X-Foo 00;
  set $foo foo1;
  if ($foo = foo1) {
    add_header X-Foo 11;
    break;
    add_header X-Foo 22;
  }
  add_header X-Foo 33;
  set $foo foo2;
  if ($foo = foo2) {
    add_header X-Foo 44;
  }
  add_header X-Foo 55;
}
```
返回数据：
```
headers:
X-Foo: 11
X-Foo: 22
```

```nginx
location /foo {
  add_header X-Foo 11;
  break;
  add_header X-Foo 22;
  set $foo foo2;
  if ($foo = foo2) {
    add_header X-Foo 33;
  }
  add_header X-Foo 44;
}
```
返回数据：
```
headers:
X-Foo: 11
X-Foo: 22
X-Foo: 44
```
出现此现象是因为`if指令`。If 属于 rewrite 模块，所以对于 if 来讲，
会和其他的 rewrite 模块执行全部执行完之后再进行下一阶段。如果 if 指令匹配成功，
那么 if 会创建一个内嵌的 location 块，只有这里面的 content阶段处理指令（NGX_HTTP_CONTENT_PHASE 阶段）会执行。
如果if指令中没有content指令，则if外部的content阶段指令依然有效。
* [Nginx指令执行阶段 - 【重要】](https://www.cnblogs.com/one-villager/p/nginx_directive_order.html)
* [If Is Evil - 【重要】](https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/)
* [Nginx if 指令工作原理 - 【重要】](https://www.codercto.com/a/39516.html)

#### return - 停止执行`rewrite模块`的所有指令

```
Syntax:	return code [text];
        return code URL;
        return URL;
Default:	—
Context:	server, location, if
```

停止执行rewrite指令，返回指定的code给客户端。非标准的`444` code关闭连接并不发送响应头信息给客户端。
从`0.8.42`开始，可以使用`301`、`302`、`303`、`307`和`308`指定redirect URL，使用其他的code用于返回响应体(text)。
`redirect URL`和`response body text`可以包含变量。Redirect URL可以使用URI，相对于当前server自动
使用`request scheme`、`server_name_in_redirect`、`port_in_redirect`。

> 在`0.7.51`版本之前，仅以下code有效：`204, 400, 402 — 406, 408, 410, 411, 413, 416, 500 — 504`

> 自`1.1.16`/`1.0.13`版本开始`307` code被认为redirect code

> 自`1.13.0`版本开始`308` code被认为redirect code


#### rewrite

```
Syntax:	rewrite regex replacement [flag];
Default:	—
Context:	server, location, if
```

如果指定的正则表达式匹配`Request URI`，`URI`被替换成`replacement`。rewrite指令是按顺序执行的，
可以通过`flag`来终止后续rewrite阶段的指令执行。

> 注：如果`replacement`以`http://`、`https://`或`$scheme`开头，则停止执行rewrite模块的指令，返回`redirect`给客户端，无论flag参数值为啥。

可选的`flag`参数值：
* 无 - 内部重定向（用户无感知，replacement全路径除外），不会停止执行rewrite模块指令。如果后面没有break、return指令时，也会执行搜索location的操作
* last - 内部重定向（用户无感知，replacement全路径除外），停止执行当前ngx_http_rewrite_module模块的指令，以改变后的URI继续进行搜索/匹配location
* break - 内部重定向（用户无感知，replacement全路径除外），停止执行当前ngx_http_rewrite_module模块的指令，和break指令类似
* redirect - 返回302跳转，停止执行当前ngx_http_rewrite_module模块的指令，当replacement不以`http://`、`https://`或`$scheme`开头时使用
* permanent - 返回301跳转，停止执行当前ngx_http_rewrite_module模块的指令

The full redirect URL is formed according to the `request scheme ($scheme)` and the `server_name_in_redirect` and `port_in_redirect` directives.

> 注：RequestURI的请求参数会自动追加到`replacement`后面，如果不想让参数自动追加至replacement后面，可以在replacement后面增加`?`：`rewrite ^/users/(.*)$ /show?user=$1? last;`。

> 如果表达式包含`}`、`;`字符，则整个表达式必须使用`单引号`或`双引号`。


#### rewrite_log - 是否开启ngx_http_rewrite_module模块指令日志，日志以`notice`级别输出至`error_log`

```
Syntax:	rewrite_log on | off;
Default:	rewrite_log off;
Context:	http, server, location, if
```


#### set - 给指定变量赋值，值可以包含文本和变量

```
Syntax:	set $variable value;
Default:	—
Context:	server, location, if
```

`set`指令可以定义变量，也可以用来赋值变量，Nginx不允许使用未定义的变量。

```nginx
location /test1 {
    set $foo hello;
    echo $foo;
}

location /test2 {
    set $foo "hello";
    echo $foo;
}

location /test3 {
    set $foo "hello world";
    echo $foo;
}

location /test4 {
    set $foo "hello";
    echo "${foo}";
}

location /test5 {
    set $foo "hello";
    echo "${foo}${foo}";
}

location /test6 {
    set $name "Jim";
    set $foo "hello $name";
    echo $foo;
}

location /test7 {
    set $name "Jim";
    set $foo "hello ${name}";
    echo $foo;
}
```

> 注：Nginx中变量值的类型只有一种，即为字符窜。所以`set $foo "hello"`指令中的`hello`可以加
引号，也可以不加，如果值包含`空格`、`;`、`{`或`}`则必须加引号。  

Nginx变量的`定义`和`赋值`发生在`不同的阶段`，赋值是在实际请求中执行的，而定义是在启动时执行的，
且定义的变量是共享的（但变量值不会全局共享，只会在单次请求内共享），定义后值默认为`空字符窜`。如下：

```nginx
server {
    location /foo {
        return 200 "foo = [$foo]";
    }
}
```
注：启动时报错：`nginx: [emerg] unknown "foo" variable`。因为没有定义foo变量。

```nginx
server {
    set $foo 111;
    location /foo {
      return 200 "foo = [$foo]";
    }
}
```
注：请求`/foo`返回`foo = [111]`

```nginx
server {
    location /foo {
        return 200 "foo = [$foo]";
    }
    
    location /bar {
        set $foo bar;
        return 200 "foo = [$foo]";
    }
}
```
注：启动时不会报`没有定义foo变量`的错误，访问`/foo`时也不会报错。因为变量的定义是在启动时完成的，
且`location /bar`中已定义了`$foo`变量：

* 访问`/foo`返回`foo = []` - 因为`$foo`变量由`location /bar`定义，默认值为`空字符窜`。且请求经过`location /foo`的过程中没有给`$foo`变量赋值
* 访问`/bar`返回`foo = [bar]`

```nginx
server {
    location /foo {
      set $foo foo;
      rewrite ^ /bar;
    }
    
    location /bar {
      return 200 "foo = [$foo]";
    }
}
```
注：变量的作用域为当前请求，而不是`变量赋值操作`所在的location。在location中定义的变量不是局部变量，
内部跳转至其他的location中也是可见的。

* 访问`/foo`返回`foo = [foo]`
* 访问`/bar`返回`foo = []`

```nginx
server {
    location /foo {
      set $foo foo1;
      add_header X-Foo1 $foo;
      set $foo foo2;
      add_header X-Foo2 $foo;
      return 200 "foo = [$foo]";
    }
}
```
> 注：访问`/foo`返回的`X-Foo1` header 值并不是`foo1`，而是`foo2`（由于[Nginx指令执行阶段](https://www.cnblogs.com/one-villager/p/nginx_directive_order.html)导致）：
```
return:
foo = [foo2]

header:
X-Foo1=foo2
X-Foo2=foo2
```

如果想对`URI参数值`进行`解码`，可以使用`ngx_set_misc`模块：
```nginx
location /test {
    set_unescape_uri $name $arg_name;
    echo "name: $name";
}
```


#### uninitialized_variable_warn - 未初始化的变量是否输出警告日志

```
Syntax:	uninitialized_variable_warn on | off;
Default:	uninitialized_variable_warn on;
Context:	http, server, location, if
```

### 参考

* [官网](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html)
* [Nginx内建变量](http://nginx.org/en/docs/varindex.html)
* [Nginx中Set变量的“来龙去脉”](https://fankeke.github.io/2017/03/09/Nginx%E4%B8%ADSet%E5%8F%98%E9%87%8F%E7%9A%84%E2%80%9C%E6%9D%A5%E9%BE%99%E5%8E%BB%E8%84%89/)
* [Nginx指令执行阶段 - 【重要】](https://www.cnblogs.com/one-villager/p/nginx_directive_order.html)
* [If Is Evil - 【重要】](https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/)
* [Nginx if 指令工作原理 - 【重要】](https://www.codercto.com/a/39516.html)
