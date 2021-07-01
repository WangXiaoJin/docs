# Docker常用

### 删除所有的 `<none>`(dangling) 镜像 

```bash
> docker rmi $(docker images -f "dangling=true" -q)
```
[参考文档](https://docs.docker.com/engine/reference/commandline/images/)

### Docker - Import/export/load/save
* [Docker — Import vs load](https://medium.com/bb-tutorials-and-thoughts/docker-import-vs-load-91d418f0073c)
* [Docker — export vs save](https://medium.com/bb-tutorials-and-thoughts/docker-export-vs-save-7053504546e5)

### gosu

> [为什么你应该在docker 中使用gosu？](https://zhuanlan.zhihu.com/p/151915585)
> [Docker using gosu vs USER](https://stackoverflow.com/questions/36781372/docker-using-gosu-vs-user)

### 搜索 alpine 指令和安装包依赖关系

> [alpine指令和安装包依赖关系](https://pkgs.alpinelinux.org/contents)