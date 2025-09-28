# Docker常用

## 删除所有的 `<none>`(dangling) 镜像 

```bash
> docker rmi $(docker images -f "dangling=true" -q)
```
[参考文档](https://docs.docker.com/engine/reference/commandline/images/)

## Docker - Import/export/load/save
* [Docker — Import vs load](https://medium.com/bb-tutorials-and-thoughts/docker-import-vs-load-91d418f0073c)
* [Docker — export vs save](https://medium.com/bb-tutorials-and-thoughts/docker-export-vs-save-7053504546e5)

## gosu

> [为什么你应该在docker 中使用gosu？](https://zhuanlan.zhihu.com/p/151915585)
> [Docker using gosu vs USER](https://stackoverflow.com/questions/36781372/docker-using-gosu-vs-user)

## 搜索 alpine 指令和安装包依赖关系

> [alpine指令和安装包依赖关系](https://pkgs.alpinelinux.org/contents)


## Watchtower

With watchtower you can update the running version of your containerized app simply by pushing a new image to the 
Docker Hub or your own image registry. Watchtower will pull down your new image, gracefully shut down your existing 
container and restart it with the same options that were used when it was deployed initially. 

> [官网](https://containrrr.dev/watchtower/)

## 镜像

* [【译】Alpine、Slim、Bookworm、Bullseye、Stretch、Jessie - 我应该选择哪个 Docker Image？](https://www.kazoottt.top/article/alpine-slim-stretch-bookworm-bullseye-buster-jessie-which-docker-image-should-i-choose)
* [What's the difference between "Shared" and "Simple" tags?](https://github.com/docker-library/faq#whats-the-difference-between-shared-and-simple-tags)
* [eclipse-temurin:17.0.12_7-jdk-alpine](https://github.com/adoptium/containers/blob/43fcefc483338727126a5263d7110f5711ad8506/17/jdk/alpine/Dockerfile) - openjdk17-eclipse-temurin Dockerfile文件
* [使用多阶段构建 Dockerfile 创建<100MB 的 Java17 Alpine 镜像](https://juejin.cn/post/7306706954678943780)
