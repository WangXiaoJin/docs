## HTTPS安装及使用

###安装Certbot

运行环境：`Nginx`、`CentOS7`

1. Certbot使用的仓库为`EPEL (Extra Packages for Enterprise Linux)`，所以确保`EPEL`仓库已开启，且开启`EPEL optional channel`

    ```bash
    shell> yum -y install epel-release
    ```
    参考文档：[enable the EPEL repository](https://fedoraproject.org/wiki/EPEL#How_can_I_use_these_extra_packages.3F)

2. 安装`Certbot` 

    [安装前置条件 - 官网](https://certbot.eff.org/docs/install.html#system-requirements)
    * Python 2.6, 2.7或 3.3+
    * 默认情况下需要`root`访问权限对`/etc/letsencrypt`、`/var/log/letsencrypt`、`/var/lib/letsencrypt`进行写操作
    
    安装`Certbot` ：
    ```bash
    shell> yum install certbot-nginx
    ```

    ```bash
    shell> certbot --help  #简要信息
    shell> certbot --help all  #详细帮助信息
    ```
    
    如果在执行命令时出现`ImportError: No module named 'requests.packages.urllib3'`错误，则使用`certbot-auto`代替，
    [参考地址](https://github.com/certbot/certbot/issues/5104)：
    ```bash
    shell> wget https://dl.eff.org/certbot-auto
    shell> chmod a+x ./certbot-auto
    shell> ./certbot-auto --help
    ```
    
    如果用`certbot-auto`还是不能安装，则使用`pip install --upgrade certbot`安装，或者使用[acme.sh](https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E)
    安装。建议使用`acme.sh`。
    
3. 生成证书

    ```bash
    shell> certbot --nginx --cert-name xxx.com --agree-tos -m xxx@xx.com -d xxx.com -d www.xxx.com
    ```
    
    > 1. 多数情况下，您需要`root`权限去运行`Certbot`
    > 2. `Let’s Encrypt`将在2018年1月份提供`wildcard certificates（通配符证书）`功能，方便使用多域名证书。
        通配符证书需通过`DNS-01 challenges`验证主域名。
    > 3. `Let’s Encrypt`证书有效期为90天，建议使用crond每天定时执行更新证书。
        `15 2 * * * certbot renew --deploy-hook "nginx -s reload" > certbot.log 2>&1`

4. 参考文档
    * [阿里云使用dns安装letsencrypt ssl](http://i.am.simonkuang.com/post/apply-http-ssl-cert-file-from-a-non-beian-aliyun-ecs/)
    * [CentOS安装certbot - 官网](https://certbot.eff.org/#centosrhel7-nginx)
    * [certbot官方文档](https://certbot.eff.org/docs/using.html#nginx)
    

###acme.sh生成证书 - 【推荐】

acme.sh 实现了 acme 协议, 可以从 letsencrypt 生成免费的证书。

1. 安装`acme.sh` : `curl  https://get.acme.sh | sh` 。安装完成后重新打开终端就可以使用`acme.sh -h`查看命令详情。

2. 使用DNS模式获取证书

    [dns-01 challenge获取证书](https://github.com/Neilpang/acme.sh/tree/master/dnsapi#11-use-aliyun-domain-api-to-automatically-issue-cert)

3. 安装证书，并使`acme.sh`每次自动`renew`证书成功后运行`reloadcmd`命令

    ```bash
    shell> acme.sh --install-cert -d easycodebox.com   \
               --key-file /etc/nginx/ssl/easycodebox.com.key \
               --fullchain-file /etc/nginx/ssl/easycodebox.com.crt \
               --reloadcmd  "nginx -s reload"
    ```
    
    修改Nginx配置，参考下面文档：
    * [acme.sh给Nginx安装Let's Encrypt - 重要](https://ruby-china.org/topics/31983)
    * [官方文档](https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E)
    * [Nginx配置HTTPS服务器](https://aotu.io/notes/2016/08/16/nginx-https/index.html)

###openssl生成证书 - [示例连接](https://github.com/vmware/harbor/blob/master/docs/configure_https.md)