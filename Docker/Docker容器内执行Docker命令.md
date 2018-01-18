## Docker容器内执行Docker命令

1. Docker容器内安装Docker服务，即常说的`docker-in-docker`，此方案比较重。

    > [Docker-in-Docker 参考文档](https://blog.docker.com/2013/09/docker-can-now-run-within-docker/)  
      [Docker-in-Docker Image](https://docs.docker.com/samples/library/docker/)  

2. Docker容器内安装`statically linked docker binary`，可以理解成Docker容器内只有Docker Client的可执行文件，
通过这个Docker Client与宿主机的docker服务进行通信。此方案非常适合构建服务部署在容器内。

    ```bash
    shell> mkdir docker-build   #创建构建镜像目录
    shell> wget https://download.docker.com/linux/static/stable/x86_64/docker-17.09.0-ce.tgz    #下载`statically linked docker binary`
    shell> tar zxf docker-17.09.0-ce.tgz
    shell> rm -f docker-17.09.0-ce.tgz
    shell> cat << EOF > Dockerfile
           FROM base-image
           COPY docker/* /usr/bin/
           EOF
    shell> docker build -t test-inner-docker .
    shell> docker run -d -v /var/run/docker.sock:/var/run/docker.sock test-inner-docker
    # 此时你就可以在容器中执行docker命令了
    ```
    
    > 注：必须使用静态的二进制可执行文件。[下载地址](https://download.docker.com/linux/static/stable/x86_64/)  
    参考地址：  
    https://github.com/moby/moby/issues/22608  
    https://docs.docker.com/engine/reference/commandline/run/#mount-volume--v-read-only

