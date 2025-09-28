# Python-工具

> Python安装包官方仓库: https://pypi.org

## 1. 包管理/虚拟环境

### pip

```shell
# 设置缓存目录，默认值：%LocalAppData%\pip\Cache
pip config set global.cache-dir "D:\temp-data\pip-cache"
```

### uv

An extremely fast Python package and project manager, written in Rust.

配置：

1. 设置缓存目录：配置环境变量`UV_CACHE_DIR`，例如：`D:\uv_cache`
2. 设置镜像源：`UV_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple`

```shell
# 缓存目录
export UV_CACHE_DIR=/path/to/cache/dir

# 镜像地址
export UV_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple

# 额外镜像地址
export EXTRA_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple

# 不用缓存
export UV_NO_CACHE=0

# 下载包时的超时时间，单位为秒
UV_HTTP_TIMEOUT=60
```

> [文档](https://docs.astral.sh/uv/)

### conda

### poetry

## 2. 数据解析

### JioNLP

JioNLP 是一个面向 NLP 开发者的工具包，提供了包括 时间语义解析、身份证、地址、分词、词性标注、命名实体识别、依存句法分析、关键词提取、文本分类、文本摘要等功能。

> Github地址：https://github.com/dongrixinyu/JioNLP

## 3. 其他

### Python Tutor

Python Tutor 是⼀个优秀的在线⼯具，以图解⽅式阐释 Python 的运作机制。支持 Python、Java、C、C++、JavaScript多种语⾔。

> https://pythontutor.com/render.html#mode=edit