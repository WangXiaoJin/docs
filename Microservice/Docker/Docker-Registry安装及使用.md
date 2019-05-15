## Docker-Registry安装及使用

###官方Docker Registry

1. 启动registry
    
    ```bash
    shell> docker run -d -p 5000:5000 --restart=always --name registry registry:2
    ```
    
    > 注：生成环境建议开启`TLS`和访问权限控制功能 - [官方文档](https://docs.docker.com/registry/deploying/)

    配置`volume`：
    ```bash
    shell> docker run -d -p 5000:5000 -v /data/registry:/var/lib/registry --restart=always --name registry registry:2
    ```

2. 关闭registry服务
    
    ```bash
    # 关闭registry服务
    shell> docker stop registry
    # 关闭registry服务且删除容器数据
    shell> docker stop registry && docker rm -v registry
    ```
    
3. 文档

    * [Deploy a registry server - 重要](https://docs.docker.com/registry/deploying/)
    * [Configuring a registry - 重要](https://docs.docker.com/registry/configuration/)
    * [Work with notifications](https://docs.docker.com/registry/notifications/)
    * [Authenticate proxy with nginx](https://docs.docker.com/registry/recipes/nginx/)
    * [Registry as a pull through cache](https://docs.docker.com/registry/recipes/mirror/)
    * [Registry HTTP API v2 - 重要](https://docs.docker.com/registry/spec/api/)
    
        * `repository name`被`/`切割的部分必须匹配正则`[a-z0-9]+(?:[._-][a-z0-9]+)*`
        * `repository name`（包含`/`）必须少于256字符

###Harbor

1. 安装

    先决条件：
    * `Python`需要`2.7`或更高版本
    * `Docker engine`需要`1.10`或更高版本
    * `Docker Compose`需要`1.6.0`或更高版本
    
    安装步骤：
    * 下载安装包  
        在[releases](https://github.com/vmware/harbor/releases)地址下下载`online`或`offline`安装包。然后解压安装包：
        ```bash
        # Online installer
        shell> tar xvf harbor-online-installer-<version>.tgz
        # Offline installer
        shell> tar xvf harbor-offline-installer-<version>.tgz
        ```
    * 配置`harbor.cfg`
    
        需修改的配置项：
        ```properties
        # 填写域名或IP
        hostname = reg.easycodebox.com
        db_password = reg_easycodebox
        ```
        
        > 配置项详情参考地址：[配置](https://github.com/vmware/harbor/blob/master/docs/installation_guide.md#configuring-harbor-listening-on-a-customized-port)
        
    * 运行`install.sh`脚本，安装并启动Harbor
        ```bash
        shell> sudo ./install.sh
        ```
        执行完成后就可以访问http://reg.easycodebox.com，执行`docker login reg.easycodebox.com`。如果安装Harbor时没启用
        https，则执行docker client命令之前需要配置`--insecure-registry reg.easycodebox.com`，重启docker服务后才可使用。
        [参考文档(Deploy a plain HTTP registry | Use self-signed certificates)](https://docs.docker.com/registry/insecure/)
        
    > 如果想修改文件存储相关配置项时，请修改`common/templates/registry/config.yml`，默认使用`local filesystem`存储，
    你可以修改成`S3, OpenStack Swift, Ceph`等，[参考地址](https://docs.docker.com/registry/configuration/)。
    
    > 生产环境建议开启HTTPS - [配置HTTPS文档](https://github.com/vmware/harbor/blob/master/docs/configure_https.md)  
    [安装文档 - 重要](https://github.com/vmware/harbor/blob/master/docs/installation_guide.md#configuring-harbor-listening-on-a-customized-port)
    
2. 通过`docker-compose`管理Harbor的生命周期，执行`docker-compose`命令时需与`docker-compose.yml`在相同目录下

    停止Harbor：
    ```bash
    $ sudo docker-compose stop
    Stopping nginx ... done
    Stopping harbor-jobservice ... done
    Stopping harbor-ui ... done
    Stopping harbor-db ... done
    Stopping registry ... done
    Stopping harbor-log ... done
    ```
    
    启动Harbor：
    ```bash
    $ sudo docker-compose start
    Starting log ... done
    Starting ui ... done
    Starting mysql ... done
    Starting jobservice ... done
    Starting registry ... done
    Starting proxy ... done
    ```
    
    修改Harbor配置文件：
    ```bash
    # 停止Harbor服务
    $ sudo docker-compose down -v
    # 修改配置项
    $ vim harbor.cfg
    # 应用修改后的配置项
    $ sudo prepare
    # 起Harbor服务
    $ sudo docker-compose up -d
    ```
    
    使用`sudo docker-compose down -v`删除Harbor容器时并不会删除镜像及DB数据。
    
    删除Harbor database和image数据（适用于纯净重安装）：
    ```bash
    $ rm -r /data/database
    $ rm -r /data/registry
    ```

3. 删除仓库及镜像
    
    删除仓库及镜像分为两个步骤：
    1. 在Harbor操作界面执行删除操作，此为逻辑删除，表明此仓库或镜像不再被Harbor管理。但是实际存储的文件并
    并没有被删除。**注：如果Tag A和Tag B指向同一个镜像，在Harbot管理界面删除Tag A后，Tag B也会被一起删除。
    如果你启用了`content trust`，你需要先用`notary`命令行工具删除Tag签名（tag's signature），然后才能删除镜像。**
    
    2. 使用`registry`的`garbage collection(GC)`删除仓库/镜像的实际存储文件。在你执行GC之前确保没有人正在推镜像或者Harbor
    处于停止运行状态。如果有人正在推送镜像，此时执行GC可能会错误的删除此镜像的数据。
    
        运行下面的命令，查看哪些镜像/文件将会被删除：
        ```bash
        $ docker-compose stop
        $ docker run -it --name gc --rm --volumes-from registry vmware/registry:2.6.2-photon garbage-collect --dry-run /etc/registry/config.yml
        ```
        **注：** 上述命令使用了`--dry-run`，起测试作用，并不会正真的删除数据。验证上述命令没问题后再使用下面命令删除：
        ```bash
        $ docker run -it --name gc --rm --volumes-from registry vmware/registry:2.6.2-photon garbage-collect  /etc/registry/config.yml
        $ docker-compose start
        ```
    
    > 注：`Docker Registry`计划在以后的版本中自动执行`GC`操作，不需要人工执行

4. [Harbor upgrade and database migration guide](https://github.com/vmware/harbor/blob/master/docs/migration_guide.md)

5. 安装及使用`Harborclient`(适合开发和运维使用)，Harbor的命令行管理工具 - [文档](https://github.com/int32bit/python-harborclient/blob/master/README.zh.md)

6. 文档

    * [官方文档 - 列表](https://github.com/vmware/harbor/tree/master/docs)
    * [Configuring Harbor as a local registry mirror](https://github.com/vmware/harbor/blob/master/contrib/Configure_mirror.md)
    * [View and test Harbor REST API via Swagger](https://github.com/vmware/harbor/blob/master/docs/configure_swagger.md)

6. 常见问题

    * [常见问题归纳 - 重要](https://github.com/vmware/harbor/wiki/Harbor-FAQs)
    
    * 解决问题
    
        a. 如果你的Harbor不能正常工作，请运行下面命令，查看所有容器的状态是否为`Up`：
        ```bash
        $ sudo docker-compose ps
                Name                     Command               State                    Ports                   
          -----------------------------------------------------------------------------------------------------
          harbor-db           docker-entrypoint.sh mysqld      Up      3306/tcp                                 
          harbor-jobservice   /harbor/harbor_jobservice        Up                                               
          harbor-log          /bin/sh -c crond && rsyslo ...   Up      127.0.0.1:1514->514/tcp                    
          harbor-ui           /harbor/harbor_ui                Up                                               
          nginx               nginx -g daemon off;             Up      0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp 
          registry            /entrypoint.sh serve /etc/ ...   Up      5000/tcp                  
        ```
        如果有容器的状态不是`Up`，则查看`/var/log/harbor`目录下相对应的日志文件。
        
        b. 当你的Harbor运行在Nginx代理或负载均衡器后面，则查看`common/templates/nginx/nginx.http.conf`文件，删除已经
        配置过的相关proxy配置项：如`location /`、`location /v2/`、`location /service/`。
        ```
        proxy_set_header X-Forwarded-Proto $scheme;
        ```
        修改完成后重新部署Harbor。










