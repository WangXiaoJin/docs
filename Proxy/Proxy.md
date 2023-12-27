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


## iptables

### 代理当前主机发起的请求

```shell
# 当前主机执行的80请求转发至 192.168.200.223:80
shell> iptables -t nat -A OUTPUT -p tcp --dport 80 -j DNAT --to-destination 192.168.200.223:80
```

### 代理别的主机发起的请求

当前主机充当代理人。

#### 1. 启用 ip_forward

```shell
shell> /etc/sysctl.conf 设置 net.ipv4.ip_forward = 1
shell> sysctl -p
```

#### 2. 配置 iptables

通过当前主机 80 端口的请求全部转发至 192.168.200.223:80
```shell
shell> iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.200.223:80
shell> iptables -t nat -A POSTROUTING -d 192.168.200.223 -j MASQUERADE
# 代理服务器的两端IP都需要放行，所以这里通过掩码配置范围IP。
# 如果不想这么麻烦，可以注掉 `-A FORWARD -j REJECT --reject-with icmp-host-prohibited` 配置，达到同等效果。
shell> iptables -I FORWARD -d 192.168.0.0/16 -p tcp -j ACCEPT
```


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

```shell
# 安装和更新 V2Ray
# 安装可执行文件和 .dat 数据文件
shell> bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)

# 安装最新发行的 geoip.dat 和 geosite.dat
shell> bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh)

# 移除 V2Ray
shell> bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh) --remove
```

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

### V2Ray 透明代理（TPROXY）

#### a. 启用 ip_forward

```shell
shell> /etc/sysctl.conf 设置 net.ipv4.ip_forward = 1
shell> sysctl -p
```

#### b. 安装 V2Ray

#### c. 修改配置文件

修改`/usr/local/etc/v2ray/config.json`配置文件，重点关注**代理服务器**相关配置。

```json5
{
  "log": {
    // Log level, one of "debug", "info", "warning", "error", "none"
    "loglevel": "warning",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
  },
  "inbounds": [
    {
      "tag": "transparent",
      "port": 10101,
      "protocol": "dokodemo-door",
      "settings": {
        "network": "tcp,udp",
        "followRedirect": true
      },
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      },
      "streamSettings": {
        "sockopt": {
          // 透明代理使用 TPROXY 方式
          "tproxy": "tproxy",
          "mark": 255
        }
      }
    },
    {
      "port": 1080,
      // 入口协议为 SOCKS 5
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      },
      "settings": {
        "auth": "noauth"
      }
    }
  ],
  "outbounds": [
    {
      "tag": "proxy",
      "protocol": "trojan",
      // 代理服务器
      "settings": {
        "servers": [
          {
            "address": "trojan.domain01",
            "port": 9303,
            "password": "a968d818-fc5d-56e4-9734-b21695bc009c"
          },
          {
            "address": "trojan.domain02",
            "port": 6301,
            "password": "a968d818-fc5d-56e4-9734-b21695bc009c"
          }
        ]
      },
      "streamSettings": {
        "security": "tls",
        "tlsSettings": {
          "allowInsecure": true
        },
        "sockopt": {
          "mark": 255
        }
      }
    },
    {
      "tag": "direct",
      "protocol": "freedom",
      "settings": {
        "domainStrategy": "UseIP"
      },
      "streamSettings": {
        "sockopt": {
          "mark": 255
        }
      }
    },
    {
      "tag": "block",
      "protocol": "blackhole",
      "settings": {
        "response": {
          "type": "http"
        }
      }
    },
    {
      "tag": "dns-out",
      "protocol": "dns",
      "streamSettings": {
        "sockopt": {
          "mark": 255
        }
      }
    }
  ],
  "dns": {
    "servers": [
      {
        //中国大陆域名使用阿里的 DNS
        "address": "223.5.5.5",
        "port": 53,
        "domains": [
          "geosite:cn",
          // NTP 服务器
          "ntp.org",
          // 此处改为你 VPS 的域名，代理服务器域名
          "trojan.domain01",
          "trojan.domain02"
        ]
      },
      {
        //中国大陆域名使用 114 的 DNS (备用)
        "address": "114.114.114.114",
        "port": 53,
        "domains": [
          "geosite:cn",
          // NTP 服务器
          "ntp.org",
          // 此处改为你 VPS 的域名，代理服务器域名
          "trojan.domain01",
          "trojan.domain02"
        ]
      },
      {
        //非中国大陆域名使用 Google 的 DNS
        "address": "8.8.8.8",
        "port": 53,
        "domains": [
          "geosite:geolocation-!cn"
        ]
      },
      {
        //非中国大陆域名使用 Cloudflare 的 DNS
        "address": "1.1.1.1",
        "port": 53,
        "domains": [
          "geosite:geolocation-!cn"
        ]
      },
      // 本机预设的 DNS 配置（备选）
      "localhost"
    ]
  },
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [
      {
        // 劫持 53 端口 UDP 流量，使用 V2Ray 的 DNS
        "type": "field",
        "inboundTag": [
          "transparent"
        ],
        "port": 53,
        "network": "udp",
        "outboundTag": "dns-out"
      },
      {
        // 直连 123 端口 UDP 流量（NTP 协议）
        "type": "field",
        "inboundTag": [
          "transparent"
        ],
        "port": 123,
        "network": "udp",
        "outboundTag": "direct"
      },
      {
        "type": "field",
        "ip": [
          // 设置 DNS 配置中的国内 DNS 服务器地址直连，以达到 DNS 分流目的
          "223.5.5.5",
          "114.114.114.114"
        ],
        "outboundTag": "direct"
      },
      {
        "type": "field",
        "ip": [
          // 设置 DNS 配置中的国外 DNS 服务器地址走代理，以达到 DNS 分流目的
          "8.8.8.8",
          "1.1.1.1"
        ],
        "outboundTag": "proxy"
      },
      {
        // 广告拦截
        "type": "field",
        "domain": [
          "geosite:category-ads-all"
        ],
        "outboundTag": "block"
      },
      {
        // BT 流量直连
        "type": "field",
        "protocol": [
          "bittorrent"
        ],
        "outboundTag": "direct"
      },
      {
        // 直连中国大陆主流网站 ip 和 保留 ip
        "type": "field",
        "ip": [
          "geoip:private",
          "geoip:cn"
        ],
        "outboundTag": "direct"
      },
      {
        // 直连中国大陆主流网站域名
        "type": "field",
        "domain": [
          "geosite:cn"
        ],
        "outboundTag": "direct"
      }
    ]
  }
}
```

使用`Shadowsocks`协议代理配置：
```json5
{
  "tag": "proxy",
  "protocol": "shadowsocks",
  // 代理服务器
  "settings": {
    "servers": [
      {
        "address": "xxxx.me",
        "port": 563,
        "method": "aes-128-gcm",
        "password": "8c07404f-dc34-4cd8-afdf-ece73ae52f20"
      }
    ]
  },
  "streamSettings": {
    "sockopt": {
      "mark": 255
    }
  }
}
```

#### d. 重启 V2Ray 服务

```shell
shell> systemctl restart v2ray
```

#### e. 配置 iptables

```shell
# 设置策略路由
ip rule add fwmark 1 table 100
ip route add local 0.0.0.0/0 dev lo table 100

# 代理局域网设备
iptables -t mangle -N V2RAY
iptables -t mangle -A V2RAY -d 0.0.0.0/8 -j RETURN
iptables -t mangle -A V2RAY -d 127.0.0.0/8 -j RETURN
iptables -t mangle -A V2RAY -d 169.254.0.0/16 -j RETURN
iptables -t mangle -A V2RAY -d 224.0.0.0/4 -j RETURN
iptables -t mangle -A V2RAY -d 240.0.0.0/4 -j RETURN
# 直连局域网，避免 V2Ray 无法启动时无法连网关的 SSH
iptables -t mangle -A V2RAY -d 10.0.0.0/8 -p tcp -j RETURN
iptables -t mangle -A V2RAY -d 172.16.0.0/12 -p tcp -j RETURN
iptables -t mangle -A V2RAY -d 192.168.0.0/16 -p tcp -j RETURN
# 直连局域网，53 端口除外（因为要使用 V2Ray 的 DNS)
iptables -t mangle -A V2RAY -d 10.0.0.0/8 -p udp ! --dport 53 -j RETURN
iptables -t mangle -A V2RAY -d 172.16.0.0/12 -p udp ! --dport 53 -j RETURN
iptables -t mangle -A V2RAY -d 192.168.0.0/16 -p udp ! --dport 53 -j RETURN
# 直连 SO_MARK 为 0xff 的流量(0xff 是 16 进制数，数值上等同与上面V2Ray 配置的 255)，此规则目的是解决v2ray占用大量CPU（https://github.com/v2ray/v2ray-core/issues/2621）
iptables -t mangle -A V2RAY -j RETURN -m mark --mark 0xff
# 给 UDP 打标记 1，转发至 10101 端口
iptables -t mangle -A V2RAY -p udp -j TPROXY --on-ip 127.0.0.1 --on-port 10101 --tproxy-mark 1
iptables -t mangle -A V2RAY -p tcp -j TPROXY --on-ip 127.0.0.1 --on-port 10101 --tproxy-mark 1
# 应用规则
iptables -t mangle -A PREROUTING -j V2RAY

# 代理网关本机
iptables -t mangle -N V2RAY_MASK
iptables -t mangle -A V2RAY_MASK -d 0.0.0.0/8 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 127.0.0.0/8 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 169.254.0.0/16 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 224.0.0.0/4 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 240.0.0.0/4 -j RETURN
# 直连局域网
iptables -t mangle -A V2RAY_MASK -d 10.0.0.0/8 -p tcp -j RETURN
iptables -t mangle -A V2RAY_MASK -d 172.16.0.0/12 -p tcp -j RETURN
iptables -t mangle -A V2RAY_MASK -d 192.168.0.0/16 -p tcp -j RETURN
# 直连局域网，53 端口除外（因为要使用 V2Ray 的 DNS）
iptables -t mangle -A V2RAY_MASK -d 10.0.0.0/8 -p udp ! --dport 53 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 172.16.0.0/12 -p udp ! --dport 53 -j RETURN
iptables -t mangle -A V2RAY_MASK -d 192.168.0.0/16 -p udp ! --dport 53 -j RETURN
# 直连 SO_MARK 为 0xff 的流量(0xff 是 16 进制数，数值上等同与上面V2Ray 配置的 255)，此规则目的是避免代理本机(网关)流量出现回环问题
iptables -t mangle -A V2RAY_MASK -j RETURN -m mark --mark 0xff
# 给 UDP 打标记，重路由
iptables -t mangle -A V2RAY_MASK -p udp -j MARK --set-mark 1
iptables -t mangle -A V2RAY_MASK -p tcp -j MARK --set-mark 1
# 应用规则
iptables -t mangle -A OUTPUT -j V2RAY_MASK

# 新建 DIVERT 规则，避免已有连接的包二次通过 TPROXY，理论上有一定的性能提升
iptables -t mangle -N DIVERT
iptables -t mangle -A DIVERT -j MARK --set-mark 1
iptables -t mangle -A DIVERT -j ACCEPT
iptables -t mangle -I PREROUTING -p tcp -m socket -j DIVERT
```

> 注：至此你可以访问外网，可访问方式：
> 1. 本机直接访问：`curl https://www.youtube.com`
> 2. 本地代理访问：`curl -x socks5://127.0.0.1:1080 https://www.youtube.com`
> 3. 局域网中其他机器代理访问：`curl -x socks5://{透明代理IP}:1080 https://www.youtube.com`
> 4. 局域网中其他机器通过网关访问，网关配置为`{透明代理IP}`即可

> 参考资料：
> * [透明代理(TPROXY)](https://guide.v2fly.org/app/tproxy.html)
> * [漫谈各种黑科技式 DNS 技术在代理环境中的应用](https://tachyondevel.medium.com/%E6%BC%AB%E8%B0%88%E5%90%84%E7%A7%8D%E9%BB%91%E7%A7%91%E6%8A%80%E5%BC%8F-dns-%E6%8A%80%E6%9C%AF%E5%9C%A8%E4%BB%A3%E7%90%86%E7%8E%AF%E5%A2%83%E4%B8%AD%E7%9A%84%E5%BA%94%E7%94%A8-62c50e58cbd0)

#### f. 卸载透明代理配置及 v2ray

当你不需要透明代理功能时，通过以下步骤来卸载。
```shell
shell> ip rule del fwmark 1 table 100
shell> ip route del local 0.0.0.0/0 dev lo table 100
shell> iptables -t mangle -F
shell> systemctl disable v2ray
shell> systemctl stop v2ray
shell> ./install-release.sh --remove
```


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