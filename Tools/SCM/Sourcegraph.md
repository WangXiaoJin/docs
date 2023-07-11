# Sourcegraph

## 安装

* [Sourcegraph on Kubernetes](https://docs.sourcegraph.com/admin/deploy/kubernetes/index)
* [Sourcegraph with Docker Compose](https://docs.sourcegraph.com/admin/deploy/docker-compose)
* [Docker Single Container Deployment](https://docs.sourcegraph.com/admin/deploy/docker-single-container)

## 配置

### Site configuration

`Site admin` -> `configuration` -> `Site configuration`

```json5
{
  // The externally accessible URL for Sourcegraph (i.e., what you type into your browser)
  // This is required to be configured for Sourcegraph to work correctly.
  // "externalURL": "https://sourcegraph.example.com",
  "auth.providers": [
    {
      "allowSignup": true,
      "type": "builtin"
    }
  ],
  "disablePublicRepoRedirects": true,
  // 密码最小长度
  "auth.minPasswordLength": 6,
  // session 过期时间
  "auth.sessionExpiry": "1h",
  // When enabled, only site admins can create and apply batch changes.
  "batchChanges.restrictToAdmins": false,
}
```

> 注：[参考配置](https://docs.sourcegraph.com/admin/config/site_config)

## 参考

* [Sourcegraph Enterprise vs. Sourcegraph OSS](https://docs.sourcegraph.com/getting-started/oss-enterprise)
* [GitHub code search vs. Sourcegraph](https://docs.sourcegraph.com/getting-started/github-vs-sourcegraph)