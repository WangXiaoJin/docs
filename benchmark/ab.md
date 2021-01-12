# ab - Apache Bench

## 例子

上传文件
```shell script
# 安装
> apk add apache2-utils
# 压测
> ab -n 100 -c 100 -s 120 -T 'application/octet-stream' -H 'X-Secret-Key: 7f84dac9d1e74a5fb235dfa8a41be8e0' -p xxx.jar http://localhost:8080/upload
```
