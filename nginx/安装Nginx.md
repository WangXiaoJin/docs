## 安装Nginx

##### 参考自[官方文档](http://nginx.org/en/docs/configure.html)、<http://blog.csdn.net/staricqxyz/article/details/17015401>
---

### Linux安装Nginx
Linux版本：CentOS release 6.7 (Final)  
Nginx版本：nginx-1.8.0

1. 源码安装  
   常用安装例子：
	
	```
	tar xzvf nginx-1.x.x.tar.gz
	cd nginx-1.x.x
	./configure --with-pcre --with-http_gzip_static_module --with-http_ssl_module
	make && make install
	```
	
   	当发生`./configure: error: C compiler cc is not found`错误时：
	```
	yum install gcc
	```
	
	当发生`./configure: error: the HTTP rewrite module requires the PCRE library.`错误时：
	```
	yum install pcre-devel
	```
   
   	当发生`./configure: error: SSL modules require the OpenSSL library.`错误时：
	```
	yum install openssl-devel
	```
   
   `configure`参数可通过`./configure --help`查看：
   * `--prefix=PATH` 定义Nginx安装目录，下面很多路径参数都是基于此参数（`prefix`）。默认值：`/usr/local/nginx`
   
   * `--sbin-path=PATH` Nginx可执行文件的安装目录。默认值：`prefix/sbin/nginx`
   
   * `--conf-path=PATH` 	设置`nginx.conf`配置文件的安装目录。通常Nginx启动时都会带`-c file`命令，所以此参数基本忽略。默认值：`prefix/conf/nginx.conf`
   
   * `--error-log-path=PATH` 设置错误、警告等日志文件。安装过后，可以在`nginx.conf`配置文件中通过`error_log`命令修改。默认值：`prefix/logs/error.log`
   
   * `--pid-path=PATH` 设置`nginx.pid`文件的目录，该文件存储了Nginx主进程的进程ID。安装过后，可以在`nginx.conf`配置文件中通过`pid`命令修改。默认值：`prefix/logs/nginx.pid`
   
   * `--lock-path=PATH` 设置`nginx.lock`文件的目录（安装文件锁定，防止安装文件被别人利用，或自己误操作。）
   
   ---
   
   * `--user=name` 设置一个普通用户（an unprivileged user），Nginx的工作进程将以此用户运行。安装过后，可以在`nginx.conf`配置文件中通过`user`命令修改。默认用户名：`nobody`
   
   * `--group=name` 设置一个组名，Nginx的工作进程将以此组运行。安装过后，可以在`nginx.conf`配置文件中通过`user`命令修改。默认组名：`nobody`
   
   ---
   
   * `--build=NAME` 设置编译文件名
   
   * `--builddir=DIR` 设置编译路径
   
   ---
   
   * `--with-rtsig_module` 启用rtsig模块（实时信号）
   
   * `--with-select_module`  
   	 `--without-select_module` -- 启用或禁用select模块。此模块允许服务使用`select()`方法，
   当系统没有更适合的方法时（kqueue, epoll, 或 /dev/poll），该模块会自动构建。
   
   * `--with-poll_module`  
   	 `--without-poll_module` -- 启用或禁用poll模块。此模块允许服务使用`poll()`方法，
   当系统没有更适合的方法时（kqueue, epoll, 或 /dev/poll），该模块会自动构建。
   
   ---
   
   * `--with-threads` 启用线程池
   
   ---
   
   * `--with-file-aio` 启用file aio支持（一种APL文件传输格式）
   
   * `--with-ipv6` 启用ipv6支持
   
   ---
   
   * `--with-http_ssl_module` 启用ngx_http_ssl_module模块。作用：让HTTP服务支持HTTPS协议。默认不构建此模块。使用前提是需要`OpenSSL`库
   
   * `--with-http_spdy_module` 启用ngx_http_spdy_module模块
   
   * `--with-http_realip_module` 启用ngx_http_realip_module模块（这个模块允许从请求头更改客户端的IP地址值，默认为关）
   
   * `--with-http_addition_module` 启用ngx_http_addition_module模块（作为一个输出过滤器，支持不完全缓冲，分部分响应请求）
   
   * `--with-http_xslt_module` 启用ngx_http_xslt_module模块（过滤转换XML请求）
   
   * `--with-http_image_filter_module` 启用http_image_filter_module模块（传输JPEG/GIF/PNG 图片的一个过滤器）（默认为不启用。gd库要用到）
   
   * `--with-http_geoip_module` 启用ngx_http_geoip_module模块（该模块创建基于与MaxMind GeoIP二进制文件相配的客户端IP地址的ngx_http_geoip_module变量）
   
   * `--with-http_sub_module` 启用ngx_http_sub_module模块（允许用一些其他文本替换nginx响应中的一些文本）
   
   * `--with-http_dav_module` 启用ngx_http_dav_module模块（增加PUT,DELETE,MKCOL：创建集合,COPY和MOVE方法）默认情况下为关闭，需编译开启
   
   * `--with-http_flv_module` 启用ngx_http_flv_module模块（提供寻求内存使用基于时间的偏移量文件）
   
   * `--with-http_mp4_module` 启用ngx_http_mp4_module模块
   
   * `--with-http_gunzip_module` 启用ngx_http_gunzip_module模块
   
   * `--with-http_gzip_static_module` 启用ngx_http_gzip_static_module模块。预先压缩文件文件，文件扩展名为`.gz`。避免每次请求都要实时压缩，浪费性能。
   
   * `--with-http_auth_request_module` 启用ngx_http_auth_request_module模块
   
   * `--with-http_random_index_module` 启用ngx_http_random_index_module模块（从目录中随机挑选一个目录索引）
   
   * `--with-http_secure_link_module` 启用ngx_http_secure_link_module模块（计算和检查要求所需的安全链接网址）
   
   * `--with-http_degradation_module` 启用ngx_http_degradation_module模块（允许在内存不足的情况下返回204或444码）
   
   * `--with-http_stub_status_module` 启用ngx_http_stub_status_module模块（获取nginx自上次启动以来的工作状态）
   
   ---
   
   * `--without-http_charset_module` 禁用ngx_http_charset_module模块（重新编码web页面，但只能是一个方向–服务器端到客户端，并且只有一个字节的编码可以被重新编码）
   
   * `--without-http_gzip_module` 禁用ngx_http_gzip_module模块（该模块同-with-http_gzip_static_module功能一样）。此模块作用：压缩HTTP请求返回数据。使用此模块的前提条件是需要`zlib`库
   
   * `--without-http_ssi_module` 禁用ngx_http_ssi_module模块（该模块提供了一个在输入端处理服务器包含文件（SSI）的过滤器，目前支持SSI命令的列表是不完整的）
   
   * `--without-http_userid_module` 禁用ngx_http_userid_module模块（该模块用来处理客户端后续请求的cookies）
   
   * `--without-http_access_module` 禁用ngx_http_access_module模块（该模块提供了一个简单的基于主机的访问控制。允许/拒绝基于ip地址）
   
   * `--without-http_auth_basic_module` 禁用ngx_http_auth_basic_module模块（该模块是可以使用用户名和密码基于http基本认证方法来保护你的站点或其部分内容）
   
   * `--without-http_autoindex_module` 禁用ngx_http_autoindex_module模块（该模块用于自动生成目录列表，只在ngx_http_index_module模块未找到索引文件时发出请求。）
   
   * `--without-http_geo_module` 禁用ngx_http_geo_module模块（创建一些变量，其值依赖于客户端的IP地址）
   
   * `--without-http_map_module` 禁用ngx_http_map_module模块（使用任意的键/值对设置配置变量）
   
   * `--without-http_split_clients_module` 禁用ngx_http_split_clients_module模块（该模块用来基于某些条件划分用户。条件如：ip地址、报头、cookies等等）
   
   * `--without-http_referer_module` 禁用ngx_http_referer_module模块（该模块用来过滤请求，拒绝报头中Referer值不正确的请求）
   
   * `--without-http_rewrite_module` 禁用ngx_http_rewrite_module模块。此模块作用：重定向HTPP请求，并修改HTTP请求的URI地址。使用此模块的前提条件是需要`PCRE`库
   
   * `--without-http_proxy_module` 禁用ngx_http_proxy_module模块
   
   * `--without-http_fastcgi_module` 禁用ngx_http_fastcgi_module模块（该模块允许Nginx 与FastCGI 进程交互，并通过传递参数来控制FastCGI 进程工作。 ）FastCGI一个常驻型的公共网关接口。
   
   * `--without-http_uwsgi_module` 禁用ngx_http_uwsgi_module模块
   
   * `--without-http_scgi_module` 禁用ngx_http_scgi_module模块（该模块用来启用SCGI协议支持，SCGI协议是CGI协议的替代。它是一种应用程序与HTTP服务接口标准。它有些像FastCGI但他的设计 更容易实现。）
   
   * `--without-http_memcached_module` 禁用ngx_http_memcached_module模块（该模块用来提供简单的缓存，以提高系统效率）
   
   * `--without-http_limit_conn_module` 禁用ngx_http_limit_conn_module模块
   
   * `--without-http_limit_req_module` 禁用ngx_http_limit_req_module模块
   
   * `--without-http_empty_gif_module` 禁用ngx_http_empty_gif_module模块（该模块在内存中常驻了一个1*1的透明GIF图像，可以被非常快速的调用）
   
   * `--without-http_browser_module` 禁用ngx_http_browser_module模块（该模块用来创建依赖于请求报头的值。如果浏览器为modern ，则$modern_browser等于modern_browser_value指令分配的值；如 果浏览器为old，则$ancient_browser等于 ancient_browser_value指令分配的值；如果浏览器为 MSIE中的任意版本，则 $msie等于1）
   
   * `--without-http_upstream_hash_module` 禁用ngx_http_upstream_hash_module模块
   
   * `--without-http_upstream_ip_hash_module` 禁用ngx_http_upstream_ip_hash_module模块（该模块用于简单的负载均衡）
   
   * `--without-http_upstream_least_conn_module` 禁用ngx_http_upstream_least_conn_module模块
   
   * `--without-http_upstream_keepalive_module` 禁用ngx_http_upstream_keepalive_module模块
   
   ---
   
   * `--with-http_perl_module` 启用ngx_http_perl_module模块（该模块使nginx可以直接使用perl或通过ssi调用perl）
   
   * `--with-perl_modules_path=PATH` 设置perl模块的路径
   
   * `--with-perl=PATH` 设置perl binary路径
   
   ---
   
   * `--http-log-path=PATH` 设置HTTP请求的日志文件。安装过后，可以在`nginx.conf`配置文件中通过`access_log`命令修改。默认值：`prefix/logs/access.log`
   
   * `--http-client-body-temp-path=PATH` 客户端请求的request body临时文件存放路径
   
   * `--http-proxy-temp-path=PATH` HTTP代理临时文件存放路径
   
   * `--http-fastcgi-temp-path=PATH` HTTP fastcgi临时文件存放路径
   
   * `--http-uwsgi-temp-path=PATH` HTTP uwsgi临时文件存放路径
   
   * `--http-scgi-temp-path=PATH` HTTP scgi临时文件存放路径
   
   ---
   
   * `--without-http` 禁用HTTP服务
   
   * `--without-http-cache` 禁用HTTP缓存
   
   ---
   
   * `--with-mail` 启用POP3/IMAP4/SMTP代理模块
   
   * `--with-mail_ssl_module` 启用ngx_mail_ssl_module模块
   
   * `--without-mail_pop3_module` 禁用ngx_mail_pop3_module模块（POP3即邮局协议的第3个版本,它是规定个人计算机如何连接到互联网上的邮件服务器进行收发邮件的协议。是因特网电子邮件的第一个离线协议标 准,POP3协议允许用户从服务器上把邮件存储到本地主机上,同时根据客户端的操作删除或保存在邮件服务器上的邮件。POP3协议是TCP/IP协议族中的一员，主要用于 支持使用客户端远程管理在服务器上的电子邮件）
   
   * `--without-mail_imap_module` 禁用ngx_mail_imap_module模块（一种邮件获取协议。它的主要作用是邮件客户端可以通过这种协议从邮件服务器上获取邮件的信息，下载邮件等。IMAP协议运行在TCP/IP协议之上， 使用的端口是143。它与POP3协议的主要区别是用户可以不用把所有的邮件全部下载，可以通过客户端直接对服务器上的邮件进行操作。）
   
   * `--without-mail_smtp_module` 禁用ngx_mail_smtp_module模块（SMTP即简单邮件传输协议,它是一组用于由源地址到目的地址传送邮件的规则，由它来控制信件的中转方式。SMTP协议属于TCP/IP协议族，它帮助每台计算机在发送或中转信件时找到下一个目的地。）
   
   ---
   
   * `--with-google_perftools_module` 启用ngx_google_perftools_module模块（调试用，剖析程序性能瓶颈）
   
   * `--with-cpp_test_module` 启用ngx_cpp_test_module模块
   
   ---
   
   * `--add-module=PATH` 添加外部模块
   
   ---
   
   * `--with-cc=PATH` 设置C编译器的路径
   
   * `--with-cpp=PATH` 设置C预处理器的路径
   
   * `--with-cc-opt=OPTIONS` 设置C编译器附加的参数（PCRE库，需要指定–with-cc-opt=”-I /usr/local/include”，如果使用select()函数则需要同时增加文件描述符数量，可以通过–with-cc- opt=”-D FD_SETSIZE=2048”指定。）
   
   * `--with-ld-opt=OPTIONS` 设置附加的linker参数（PCRE库，需要指定–with-ld-opt=”-L /usr/local/lib”。）
   
   * `--with-cpu-opt=CPU` 用指定的CPU来构建、编译。有效值：pentium、pentiumpro、pentium3、pentium4、athlon、opteron、sparc32、sparc64、ppc64
   
   ---
   
   * `--without-pcre` 禁用PCRE库
   
   * `--with-pcre` 强制使用PCRE库
   
   * `--with-pcre=DIR` 设置PCRE库路径。`location`命令和`ngx_http_rewrite_module`模块的正则表达式需要用到此库
   
   * `--with-pcre-opt=OPTIONS` 设置PCRE附加参数
   
   * `--with-pcre-jit` 编译时PCRE支持JIT(just-in-time compilation)
   
   ---
   
   * `--with-md5=DIR` 设置MD5库的路径
   
   * `--with-md5-opt=OPTIONS` 在编译时为md5库设置附加参数
   
   * `--with-md5-asm` 使用md5汇编源
   
   ---
   
   * `--with-sha1=DIR` 设置SHA1库的路径
   
   * `--with-sha1-opt=OPTIONS` 在编译时为SHA1库设置附加参数
   
   * `--with-sha1-asm` 使用SHA1汇编源
   
   ---
   
   * `--with-zlib=DIR` 设置zlib库的路径。`ngx_http_gzip_module`模块使用此库
   
   * `--with-zlib-opt=OPTIONS` 在编译时为zlib库设置附加参数
   
   * `--with-zlib-asm=CPU` 为指定的CPU使用zlib汇编源进行优化，CPU类型为pentium, pentiumpro
   
   ---
   
   * `--with-libatomic` 为原子内存的更新操作的实现提供一个架构
   
   * `--with-libatomic=DIR` 指向libatomic_ops安装目录
   
   ---
   
   * `--with-openssl=DIR` 指向openssl安装目录
   
   * `--with-openssl-opt=OPTIONS` 在编译时为openssl设置附加参数
   
   ---
   
   * `--with-debug` 启用debug日志
   

