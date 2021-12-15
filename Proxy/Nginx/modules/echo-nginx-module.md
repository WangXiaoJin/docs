## echo-nginx-module介绍

##### `【参考官网】`<https://github.com/openresty/echo-nginx-module>
---

echo-nginx-module模块用来在客户端输出内容，方便了调试。

安装echo-nginx-module
---

```bash

 $ wget 'http://nginx.org/download/nginx-1.7.10.tar.gz'
 $ tar -xzvf nginx-1.7.10.tar.gz
 $ cd nginx-1.7.10/

 # Here we assume you would install you nginx under /opt/nginx/.
 $ ./configure --prefix=/opt/nginx \
     --add-module=/path/to/echo-nginx-module

 $ make -j2
 $ make install
```

> 注：`make -j2` 中-j2是让处理器同时处理两个编译任务，以提高效率

例子：
---

```nginx

   location /hello {
     echo "hello, world!";
   }
```

```nginx

   location /hello {
     echo -n "hello, ";
     echo "world!";
   }
```

```nginx

   location /timed_hello {
     echo_reset_timer;
     echo hello world;
     echo "'hello world' takes about $echo_timer_elapsed sec.";
     echo hiya igor;
     echo "'hiya igor' takes about $echo_timer_elapsed sec.";
   }
```

```nginx

   location /echo_with_sleep {
     echo hello;
     echo_flush;  # ensure the client can see previous output immediately
     echo_sleep   2.5;  # in sec
     echo world;
   }
```

```nginx

   # in the following example, accessing /echo yields
   #   hello
   #   world
   #   blah
   #   hiya
   #   igor
   location /echo {
       echo_before_body hello;
       echo_before_body world;
       proxy_pass $scheme://127.0.0.1:$server_port$request_uri/more;
       echo_after_body hiya;
       echo_after_body igor;
   }
   location /echo/more {
       echo blah;
   }
```

```nginx

   # the output of /main might be
   #   hello
   #   world
   #   took 0.000 sec for total.
   # and the whole request would take about 2 sec to complete.
   location /main {
       echo_reset_timer;

       # subrequests in parallel
       echo_location_async /sub1;
       echo_location_async /sub2;

       echo "took $echo_timer_elapsed sec for total.";
   }
   location /sub1 {
       echo_sleep 2;
       echo hello;
   }
   location /sub2 {
       echo_sleep 1;
       echo world;
   }
```

```nginx

   # the output of /main might be
   #   hello
   #   world
   #   took 3.003 sec for total.
   # and the whole request would take about 3 sec to complete.
   location /main {
       echo_reset_timer;

       # subrequests in series (chained by CPS)
       echo_location /sub1;
       echo_location /sub2;

       echo "took $echo_timer_elapsed sec for total.";
   }
   location /sub1 {
       echo_sleep 2;
       echo hello;
   }
   location /sub2 {
       echo_sleep 1;
       echo world;
   }
```

```nginx

   # Accessing /dup gives
   #   ------ END ------
   location /dup {
     echo_duplicate 3 "--";
     echo_duplicate 1 " END ";
     echo_duplicate 3 "--";
     echo;
   }
```

```nginx

   # /bighello will generate 1000,000,000 hello's.
   location /bighello {
     echo_duplicate 1000_000_000 'hello';
   }
```

```nginx

   # echo back the client request
   location /echoback {
     echo_duplicate 1 $echo_client_request_headers;
     echo "\r";

     echo_read_request_body;

     echo_request_body;
   }
```

```nginx

   # GET /multi will yields
   #   querystring: foo=Foo
   #   method: POST
   #   body: hi
   #   content length: 2
   #   ///
   #   querystring: bar=Bar
   #   method: PUT
   #   body: hello
   #   content length: 5
   #   ///
   location /multi {
       echo_subrequest_async POST '/sub' -q 'foo=Foo' -b 'hi';
       echo_subrequest_async PUT '/sub' -q 'bar=Bar' -b 'hello';
   }
   location /sub {
       echo "querystring: $query_string";
       echo "method: $echo_request_method";
       echo "body: $echo_request_body";
       echo "content length: $http_content_length";
       echo '///';
   }
```

```nginx

   # GET /merge?/foo.js&/bar/blah.js&/yui/baz.js will merge the .js resources together
   location /merge {
       default_type 'text/javascript';
       echo_foreach_split '&' $query_string;
           echo "/* JS File $echo_it */";
           echo_location_async $echo_it;
           echo;
       echo_end;
   }
```

```nginx

   # accessing /if?val=abc yields the "hit" output
   # while /if?val=bcd yields "miss":
   location ^~ /if {
       set $res miss;
       if ($arg_val ~* '^a') {
           set $res hit;
           echo $res;
       }
       echo $res;
   }
```

