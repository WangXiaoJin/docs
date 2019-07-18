# Docker常用

### 删除所有的 `<none>`(dangling) 镜像 

```bash
> docker rmi $(docker images -f "dangling=true" -q)
```
[参考文档](https://docs.docker.com/engine/reference/commandline/images/)


