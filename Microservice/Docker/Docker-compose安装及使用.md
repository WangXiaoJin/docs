## Docker-compose安装及使用

###CentOS安装docker-compose

1. 先安装`Docker Engine`
2. 安装`Docker Compose`
    
    ```bash
    # 从Github上下载Docker Compose
    shell> sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    # 添加可执行权限
    shell> sudo chmod +x /usr/local/bin/docker-compose
    ```
    
    > [安装文档 - 【官网】](https://docs.docker.com/compose/install/)  
      注： 如果通过`curl -L`下载的文件有问题可直接通过浏览器下载

3. 安装`bash`/`zsh` shell环境下的`command completion`（可选）

    #### Bash
    
    * 安装`bash-completion`
    
        Linux：
        ```bash
        # 查看本地是否已安装
        shell> yum list installed | grep bash-completion
        # 安装 bash-completion
        shell> yum install bash-completion
        # 使当前terminal的bash-completion生效（可选操作）
        shell> source /etc/profile.d/bash_completion.sh
        ```
        
        Mac：
        ```bash
        # 安装 bash-completion
        shell> brew install bash-completion
        ```
    
    * 下载Docker Compose的自动命令补全文件至`/etc/bash_completion.d/`（Mac系统路径为：`/usr/local/etc/bash_completion.d/`）
    
        ```bash
        shell> sudo curl -L https://raw.githubusercontent.com/docker/compose/1.16.1/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
        ```
        
        **Mac系统**添加以下脚本至`~/.bash_profile`：
        ```bash
        if [ -f $(brew --prefix)/etc/bash_completion ]; then
        . $(brew --prefix)/etc/bash_completion
        fi
        ```
        `source ~/.bash_profile`使当前terminal 生效
        
    #### Zsh
    
    下载自动补全脚本至`/path/to/zsh/completion`，例：`~/.zsh/completion/`：
    ```bash
    shell> mkdir -p ~/.zsh/completion
    shell> curl -L https://raw.githubusercontent.com/docker/compose/1.16.1/contrib/completion/zsh/_docker-compose > ~/.zsh/completion/_docker-compose
    ```
    
    `$fpath`包含目录，例：`~/.zshrc`：
    ```bash
    fpath=(~/.zsh/completion $fpath)
    ```
    
    确保`compinit`已经加载或者把它添加进`~/.zshrc`:
    ```bash
    autoload -Uz compinit && compinit -i
    ```
    
    最后重启shell：
    ```bash
    exec $SHELL -l
    ```

    > 注：[官方文档](https://docs.docker.com/compose/completion/)

4. 测试`Docker Compose`

    ```bash
    shell> docker-compose --version
    docker-compose version 1.16.1, build 1719ceb
    ```

    > 注：`docker-compose`可以运行在`非swarm`环境中,`docker-compose.yml`的`3.x`版本可以兼容`docker swarm`（`docker stack deploy -c docker-compose.yml hello`）

###文档

* [Get started - 重要](https://docs.docker.com/compose/gettingstarted/)
* [Compose file语法 - 重要](https://docs.docker.com/compose/compose-file/)

