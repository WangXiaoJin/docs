# SSH / SFTP / FileSystem

## SSH SFTP

### JSch

JSch is a pure Java implementation of SSH2.

JSch allows you to connect to an sshd server and use `port forwarding`, `X11 forwarding`, `file transfer`, etc., 
and you can integrate its functionality into your own Java programs.

* [官网](http://www.jcraft.com/jsch/)
* [示例代码](http://www.jcraft.com/jsch/examples/) - 下载的源码里包含这些示例
* [File Transfer using SFTP in Java (JSch)](https://mkyong.com/java/file-transfer-using-sftp-in-java-jsch/)
* [Explanation for SCP protocol implementation in JSch library](https://stackoverflow.com/questions/26220381/explanation-for-scp-protocol-implementation-in-jsch-library)

> **优点**：不依赖其他包，连接支持`Socks5 Proxy`、`HTTP Proxy`，支持功能：`X11Forwarding`/`KnownHosts`/`Sftp`/`PortForwarding`。  
> **缺点**：可扩展性低。

### 示例

```java
import com.jcraft.jsch.ChannelExec;
import com.jcraft.jsch.ChannelSftp;
import com.jcraft.jsch.JSchException;
import com.jcraft.jsch.Session;
import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.net.Proxy.Type;

public class Test {

    public static void main(String[] args) {
        // SSH执行命令
        execCommand();
        // SFTP 上传/下载
        uploadAndDownload();
    }

    /**
     * SSH 执行命令
     */
    public static void execCommand() {
        Session session = null;
        ChannelExec channel = null;
        try {
            ByteArrayOutputStream response = new ByteArrayOutputStream();
            ByteArrayOutputStream error = new ByteArrayOutputStream();
            // 创建 session
            session = proxySession();
            // session 连接
            session.connect();
            // 开启 exec channel
            channel = (ChannelExec) session.openChannel("exec");
            // 设置待执行命令
            channel.setCommand("ls /");
            // 设置输出流
            channel.setOutputStream(response);
            channel.setErrStream(error);
            // channel 连接
            channel.connect();
            // 判断 channel 是否连接着，如果连接没有关闭则等待关闭
            while (channel.isConnected()) {
                Thread.sleep(100);
            }
            // 退出码
            int exitStatus = channel.getExitStatus();
            System.out.println("exitStatus：" + exitStatus);
            // 获取响应流数据
            String responseStr = response.toString();
            String errorStr = error.toString();
            System.out.println(responseStr);
            System.out.println(errorStr);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (session != null) {
                session.disconnect();
            }
            if (channel != null) {
                channel.disconnect();
            }
        }
    }

    /**
     * SFTP 上传/下载
     */
    public static void uploadAndDownload() {
        Session session = null;
        ChannelSftp channel = null;
        try {
            // 创建 session
            session = proxySession();
            // session 连接
            session.connect();
            // 开启 sftp channel
            channel = (ChannelSftp) session.openChannel("sftp");
            // channel 连接
            channel.connect();

            // 上传
            try (FileInputStream inputStream = new FileInputStream("d:/xx.png")) {
                channel.put(inputStream, "/xx.png");
            }
            // 下载
            channel.get("/xx.png", "d:/xx-1.png");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (session != null) {
                session.disconnect();
            }
            if (channel != null) {
                channel.disconnect();
            }
        }
    }

    /**
     * 通过密码 SSH 连接
     */
    private static Session passwordSession() throws JSchException {
        JSchInfo info = JSchInfo.builder("192.168.1.100", "root")
            .password("111111")
            .build();
        return JSchs.getSession(info);
    }

    /**
     * 设置IO阻塞超时时间
     */
    private static Session timeoutSession() throws JSchException {
        JSchInfo info = JSchInfo.builder("192.168.1.100", "root")
            .password("111111")
            .timeout(60000)
            .build();
        return JSchs.getSession(info);
    }

    /**
     * 通过私钥 SSH 连接
     */
    private static Session privateKeySession() throws JSchException {
        JSchInfo info = JSchInfo.builder("192.168.1.100", "root")
            .port(22)
            // 填写私钥字符串
            .privateKey("xxxxxxxxxxxxxxxxxxx")
            .build();
        return JSchs.getSession(info);
    }

    /**
     * SSH 连接使用代理
     */
    private static Session proxySession() throws JSchException {
        JSchInfo info = JSchInfo.builder("192.168.1.100", "root")
            // 默认值 22
            .port(22)
            .password("111111")
            .proxyType(Type.SOCKS)
            .proxyHost("proxy.xxx.xxx")
            .proxyPort(2003)
            .proxyUser("proxyUser")
            .proxyPass("proxyPass")
            .build();
        return JSchs.getSession(info);
    }

}
```

```java
import java.net.Proxy;
import java.net.Proxy.Type;
import lombok.EqualsAndHashCode;
import lombok.Getter;
import lombok.ToString;
import org.apache.commons.lang3.StringUtils;

/**
 * @author WangXiaoJin
 * @date 2021-12-22 19:38
 */
@Getter
@ToString
@EqualsAndHashCode
public class JSchInfo {

    private JSchInfo() {

    }

    /**
     * SSH 远程主机名（必填）
     */
    private String host;

    /**
     * SSH 远程主机端口（可选，默认: 22）
     */
    private int port = 22;

    /**
     * SSH 用户名（必填）
     */
    private String username;

    /**
     * SSH 密码（密码和私钥必填一项）
     */
    private String password;

    /**
     * SSH 私钥（密码和私钥必填一项）
     */
    private String privateKey;

    /**
     * 设置IO阻塞超时时间，默认为0，无限等待（具体最大值由系统决定）。单位为毫秒
     */
    private int timeout;

    /**
     * 代理类型，默认为 DIRECT
     */
    private Proxy.Type proxyType = Type.DIRECT;

    /**
     * 代理主机名
     */
    private String proxyHost;

    /**
     * 代理端口
     */
    private Integer proxyPort;

    /**
     * 代理账号
     */
    private String proxyUser;

    /**
     * 代理密码
     */
    private String proxyPass;

    public static Builder builder(String host, String username) {
        return new Builder(host, username);
    }

    public static class Builder {

        private final JSchInfo info;

        private Builder(String host, String username) {
            Assert.notBlank(host, "host为空");
            Assert.notBlank(username, "username为空");
            info = new JSchInfo();
            info.host = host;
            info.username = username;
        }

        public Builder port(Integer port) {
            info.port = port;
            return this;
        }

        public Builder password(String password) {
            info.password = password;
            return this;
        }

        public Builder privateKey(String privateKey) {
            info.privateKey = privateKey;
            return this;
        }

        public Builder timeout(Integer timeout) {
            info.timeout = timeout;
            return this;
        }

        public Builder proxyType(Proxy.Type proxyType) {
            info.proxyType = proxyType;
            return this;
        }

        public Builder proxyHost(String proxyHost) {
            info.proxyHost = proxyHost;
            return this;
        }

        public Builder proxyPort(Integer proxyPort) {
            info.proxyPort = proxyPort;
            return this;
        }

        public Builder proxyUser(String proxyUser) {
            info.proxyUser = proxyUser;
            return this;
        }

        public Builder proxyPass(String proxyPass) {
            info.proxyPass = proxyPass;
            return this;
        }

        public JSchInfo build() {
            Assert.isFalse(info.password == null && info.privateKey == null, "password 或 privateKey 必填一项");
            if (info.proxyType != Type.DIRECT) {
                if (StringUtils.isEmpty(info.proxyHost)) {
                    throw new IllegalArgumentException("启用代理后，proxyHost必填");
                }
                if (info.proxyPort == null) {
                    throw new IllegalArgumentException("启用代理后，proxyPort必填");
                }
            }
            return info;
        }
    }

}
```

```java
import com.jcraft.jsch.JSch;
import com.jcraft.jsch.JSchException;
import com.jcraft.jsch.ProxyHTTP;
import com.jcraft.jsch.ProxySOCKS5;
import com.jcraft.jsch.Session;
import java.net.Proxy.Type;

public class JSchs {

    private JSchs() {
        throw new UnsupportedOperationException();
    }

    public static Session getSession(JSchInfo info) throws JSchException {
        JSch jSch = new JSch();
        if (info.getPrivateKey() != null) {
            // 设置私钥
            jSch.addIdentity("privateKey", info.getPrivateKey().getBytes(), null, null);
        }
        Session session = jSch.getSession(info.getUsername(), info.getHost(), info.getPort());
        if (info.getPassword() != null) {
            session.setPassword(info.getPassword());
        }
        session.setTimeout(info.getTimeout());
        session.setConfig("StrictHostKeyChecking", "no");
        // 设置代理
        if (info.getProxyType() == Type.SOCKS) {
            ProxySOCKS5 proxy = new ProxySOCKS5(info.getProxyHost(), info.getProxyPort());
            proxy.setUserPasswd(info.getProxyUser(), info.getProxyPass());
            session.setProxy(proxy);
        } else if (info.getProxyType() == Type.HTTP) {
            ProxyHTTP proxy = new ProxyHTTP(info.getProxyHost(), info.getProxyPort());
            proxy.setUserPasswd(info.getProxyUser(), info.getProxyPass());
            session.setProxy(proxy);
        }
        return session;
    }
}
```

### Apache MINA SSHD

Apache SSHD is a 100% pure java library to support the SSH protocols on both the `client` and `server` side. 
This library can leverage Apache MINA, a scalable and high performance asynchronous IO library.

* [官网](https://github.com/apache/mina-sshd)
  * [属性配置及继承模型](https://github.com/apache/mina-sshd/blob/master/docs/internals.md#properties-and-inheritance-model)
  * [SSH Jumps](https://github.com/apache/mina-sshd/blob/master/docs/internals.md#ssh-jumps)

> 通过`ShellChannel#getInvertedIn()` 输入命令时需要在命令后面添加`\n`后缀，否则命令不会执行。

> **优点**：可扩展性高，支持`Server`和`Client`。支持功能：`X11Forwarding`/`KnownHosts`/`Sftp`/`PortForwarding`/`SCP`/`Event listeners and handlers`/`Command line clients`。  
> **缺点**：依赖的包多，暂时没找到连接支持`Socks5 Proxy`、`HTTP Proxy`方式，只找到Server端启用Socks5（`start/stopDynamicPortForwarding`）功能。

### SSH 参考文档

* [SSH Connection With Java](https://www.baeldung.com/java-ssh-connection) - 示例包含`JSch`及`Apache MINA SSHD`
* [Proxies and Jump Hosts](https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Proxies_and_Jump_Hosts)

## File System

### Apache Commons VFS

`Commons Virtual File System`(Commons VFS) provides a single API for accessing various different file systems. 
It presents a uniform view of the files from various different sources, such as the files on local disk, on an HTTP server, or inside a Zip archive.

Some of the features of Commons VFS are:
* A single consistent API for accessing files of different types.
* Support for numerous [file system types](https://commons.apache.org/proper/commons-vfs/filesystems.html) .
* Caching of file information. Caches information in-JVM, and optionally can cache remote file information on the local file system (replicator).
* Event delivery.
* Support for logical file systems made up of files from various different file systems.
* Utilities for integrating Commons VFS into applications, such as a VFS-aware ClassLoader and URLStreamHandlerFactory.
* A set of VFS-enabled [Ant tasks](https://commons.apache.org/proper/commons-vfs/anttasks.html).

