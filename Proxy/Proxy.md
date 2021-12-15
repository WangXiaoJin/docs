# Proxy


## Nginx

* [使用 NGINX 作为 HTTPS 正向代理服务器](https://www.infoq.cn/article/taujwgln6d_6qls6yj6s)

* [TCP and UDP Proxy and Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/tcp-udp-load-balancer/) -【官网】
    * [Accepting the PROXY Protocol](https://docs.nginx.com/nginx/admin-guide/load-balancer/using-proxy-protocol/)    

* `nginx_tcp_proxy_module` - [Support TCP proxy with Nginx](https://github.com/yaoweibin/nginx_tcp_proxy_module)


## Java Net Proxy

* [Java Networking and Proxies](https://docs.oracle.com/javase/8/docs/technotes/guides/net/proxies.html)

示例：
```java
public class Test {

    public static final String PROXY_DEV = "dev";
    public static final String PROXY_TEST = "test";
    public static final String PROXY_PRE = "pre";
    public static final String PROXY_PROD = "prod";

    public static void main(String[] args) {

        // 创建 OkHttpClient.Builder 对象
        OkHttpClient.Builder builder = new OkHttpClient.Builder()
            .connectTimeout(20, TimeUnit.SECONDS);

        // 设置代理对象，当前应用部署在哪个环境就用对应环境的 代理IP
        // 端口号表示代理路由至哪台 Proxy Server
        Map<String, String> proxyMap = new HashMap<>();
        proxyMap.put(PROXY_DEV, "proxy.host:2001");
        proxyMap.put(PROXY_TEST, "proxy.host:2002");
        proxyMap.put(PROXY_PRE, "proxy.host:2003");
        proxyMap.put(PROXY_PROD, "proxy.host:2004");

        // 创建 OkHttpClientProxyCache 对象
        OkHttpClientProxyCache clientProxyCache = new OkHttpClientProxyCache(builder, proxyMap, Type.SOCKS, "username",
            "password");
        // 代理至 PRE 环境
        OkHttpClient client = clientProxyCache.getClient(PROXY_PRE);

        // 发送请求
        Request request = new Builder().url("http://10.75.16.31:8080/actuator").build();
        try (Response execute = client.newCall(request).execute()) {
            System.out.println(execute.code());
            System.out.println(execute.message());
            if (execute.body() != null) {
                System.out.println(execute.body().string());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}
```

```java

import java.net.Authenticator;
import java.net.InetSocketAddress;
import java.net.PasswordAuthentication;
import java.net.Proxy;
import java.net.Proxy.Type;
import java.util.HashMap;
import java.util.Map;
import okhttp3.Credentials;
import okhttp3.OkHttpClient;
import org.apache.commons.lang3.StringUtils;

/**
 * OkHttpClient 代理对象缓存
 *
 * @author WangXiaoJin
 * @date 2021-12-15 11:36
 */
public class OkHttpClientProxyCache {

    private final Map<String, OkHttpClient> clientCache = new HashMap<>();

    public OkHttpClientProxyCache(OkHttpClient.Builder builder, Map<String, String> proxyMap, Proxy.Type type,
        String proxyUser, String proxyPass) {
        Assert.notNull(builder, "builder is null");
        Assert.notNull(proxyMap, "proxyMap is null");
        proxyMap.forEach((key, addr) -> {
            builder.proxy(new Proxy(type, parseAddress(addr)));
            if (StringUtils.isNotBlank(proxyUser) && StringUtils.isNotBlank(proxyPass)) {
                if (type == Type.SOCKS) {
                    Authenticator.setDefault(new Authenticator() {
                        @Override
                        protected PasswordAuthentication getPasswordAuthentication() {
                            return new PasswordAuthentication(proxyUser, proxyPass.toCharArray());
                        }
                    });
                } else if (type == Type.HTTP) {
                    builder.proxyAuthenticator((route, response) -> {
                        if (response.request().header("Proxy-Authorization") != null) {
                            return null;
                        }
                        return response.request().newBuilder()
                            .header("Proxy-Authorization", Credentials.basic(proxyUser, proxyPass))
                            .build();
                    });
                }
            }
            clientCache.put(key, builder.build());
        });
    }

    /**
     * 获取对应的 OkHttpClient 代理
     */
    public OkHttpClient getClient(String key) {
        Assert.notNull(key, "key is null");
        return clientCache.get(key);
    }

    /**
     * 解析 SocketAddress，合法格式：
     * <ul>
     *     <li>192.168.2.1:1111</li>
     *     <li>[2001:db8:85a3:8d3:1319:8a2e:370:7348]:1111</li>
     * </ul>
     */
    private InetSocketAddress parseAddress(String addressString) {
        Assert.notNull(addressString, "addressString is null");
        addressString = addressString.trim();
        int lastColon = addressString.lastIndexOf(":");
        int lastClosingSquareBracket = addressString.lastIndexOf("]");
        String host;
        int port;
        if (lastClosingSquareBracket == -1) {
            String[] parts = addressString.split(":");
            Assert.isTrue(parts.length == 2, "Address {0} is illegal", addressString);
            host = parts[0];
            port = Integer.parseInt(parts[1]);
        } else {
            Assert.isTrue(lastClosingSquareBracket < lastColon, "Address {0} is illegal", addressString);
            host = addressString.substring(0, lastColon);
            port = Integer.parseInt(addressString.substring(lastColon + 1));
        }
        return new InetSocketAddress(host, port);
    }
}
```


## Squid

* [Squid中文权威指南](https://www.phpfans.net/manu/Squid/)
  * [备用地址](https://www.dba.cn/book/squid/)
* [官网配置手册](http://www.squid-cache.org/Doc/config/)
* [Squid 超速缓存代理服务器](https://documentation.suse.com/zh-cn/sles/15-SP2/html/SLES-all/cha-squid.html)
* [Squid父级socks代理](https://cloud-atlas.readthedocs.io/zh_CN/latest/web/proxy/squid/squid_socks_peer.html)


## SOCKs5

* [SOCKS Proxy Primer: What Is SOCKs5 and Why Should You Use It?](https://securityintelligence.com/posts/socks-proxy-primer-what-is-socks5-and-why-should-you-use-it/)
* [Create a SOCKS proxy on a Linux server with SSH to bypass content filters](https://ma.ttias.be/socks-proxy-linux-ssh-bypass-content-filters/)


## V2Ray - `Project V`

* `多入口多出口` - 一个 V2Ray 进程可并发支持多个入站和出站协议，每个协议可独立工作。
* `定制化路由` - 入站流量可按配置由不同地出口发出。轻松实现按区域或按域名分流，以达到最优的网络性能。
* `多协议支持` - V2Ray 可同时开启多个协议支持，包括 Socks、HTTP、Shadowsocks 和 VMess 等。每个协议可单独设置传输载体，比如 TCP、mKCP 和 WebSocket 等。
* `隐蔽性` - V2Ray 的节点可以伪装成正常的网站（HTTPS），将其流量与正常的网页流量混淆，以避开第三方干扰。
* `反向代理` - 通用的反向代理支持，可实现内网穿透功能。
* `多平台支持` - 原生支持所有常见平台，如 Windows、macOS 和 Linux，并已有第三方支持移动平台。

### 安装 V2Ray

1. 下载安装脚本：https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh
2. 下载最新版本`v2ray-linux-64.zip`，下载地址：https://github.com/v2fly/v2ray-core/releases
3. 安装本地包：`./install-release.sh --local v2ray-linux-64.zip`
4. 开启自启动：`systemctl enable v2ray`
5. 启动服务：`systemctl start v2ray`
6. 关闭服务：`systemctl stop v2ray`
7. 卸载：`./install-release.sh --remove`

安装目录：
```
installed: /usr/local/bin/v2ray
installed: /usr/local/bin/v2ctl
installed: /usr/local/share/v2ray/geoip.dat
installed: /usr/local/share/v2ray/geosite.dat
installed: /usr/local/etc/v2ray/config.json
installed: /var/log/v2ray/
installed: /var/log/v2ray/access.log
installed: /var/log/v2ray/error.log
installed: /etc/systemd/system/v2ray.service
installed: /etc/systemd/system/v2ray@.service
```

> 注：因GitHub被墙，所以不能直接通过脚本远程下载，只能本地安装包安装。

* [官网](https://www.v2fly.org/)
* [fhs-install-v2ray](https://github.com/v2fly/fhs-install-v2ray/blob/master/README.zh-Hans-CN.md) - 安装脚本项目


## goproxy

Proxy是golang实现的高性能http,https,websocket,tcp,socks5代理服务器,支持内网穿透,链式代理,通讯加密,智能HTTP,SOCKS5代理,黑白名单,限速,限流量,限连接数,跨平台,KCP支持,认证API

> [官网](https://github.com/snail007/goproxy)


## Tinyproxy

lightweight http(s) proxy daemon

* [官网](http://tinyproxy.github.io/)


## Dante

A free SOCKS server

* [官网](http://www.inet.no/dante/)


## nps

nps是一款轻量级、高性能、功能强大的内网穿透代理服务器。目前支持tcp、udp流量转发，可支持任何tcp、udp上层协议（访问内网网站、本地支付接口调试、ssh访问、远程桌面，内网dns解析等等……），此外还支持内网http代理、内网socks5代理、p2p等，并带有功能强大的web管理端。

* [官网 - Github](https://github.com/ehang-io/nps)

## 参考文档

* [Proxy vs VPN: what are the main differences?](https://nordvpn.com/zh/blog/vpn-vs-proxy/)
* [Choosing Between SOCKS vs HTTP Proxy](https://oxylabs.io/blog/socks-vs-http-proxy)
* [五大开源 Web 代理服务器横评：Squid、Privoxy、Varnish、Polipo、Tinyproxy](https://linux.cn/article-7119-1.html)