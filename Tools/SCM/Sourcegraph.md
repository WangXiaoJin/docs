# Sourcegraph

## 1. 安装

### Install-script 安装

> [Install Sourcegraph via Shell Script](https://docs.sourcegraph.com/admin/deploy/single-node/script)

### 其他安装方式
* [Sourcegraph on Kubernetes](https://docs.sourcegraph.com/admin/deploy/kubernetes/index)
* [Sourcegraph with Docker Compose](https://docs.sourcegraph.com/admin/deploy/docker-compose)

> [docker-compose/kubernetes部署的资源评估](https://docs.sourcegraph.com/admin/deploy/resource_estimator)

### Sourcegraph 浏览器插件

Sourcegraph 浏览器插件增强了`GitHub`、`Phabricator`和`Bitbucket`功能：
* symbol type information & documentation
* go to definition & find references (currently for Go, Java, TypeScript, JavaScript, Python)
* find references

> [插件安装地址](https://docs.sourcegraph.com/integration/browser_extension)

### IDEA 安装

> [配置文档](https://github.com/sourcegraph/jetbrains#settings)


## 2. 配置

### Site configuration

系统配置，配置路径：`Site admin -> configuration -> Site configuration`

```json5
{
  // The externally accessible URL for Sourcegraph (i.e., what you type into your browser)
  // This is required to be configured for Sourcegraph to work correctly.
  "externalURL": "http://sourcegraph.xxx.com",
  // 登录页面是否显示`创建用户请求`的链接
  "auth.accessRequest": {
    "enabled": false
  },
  "auth.providers": [
    {
      "allowSignup": false,
      "type": "builtin"
    }
  ],
  
  // Enables and configures password policy. This will allow admins to enforce password complexity and length requirements.
  "auth.passwordPolicy": null,
  // Other example values:
  // - {
  //     "enabled": true,
  //     "numberOfSpecialCharacters": 1,
  //     "requireAtLeastOneNumber": true,
  //     "requireUpperandLowerCase": true
  //   }
  
  // 密码最小长度
  "auth.minPasswordLength": 6,

  // Customize Sourcegraph homepage logo and search icon.
  "branding": null,
  // Other example values:
  // - {
  //     "dark": {
  //       "logo": "https://example.com/logo_dark.png",
  //       "symbol": "https://example.com/search_symbol_dark_24x24.png"
  //     },
  //     "disableSymbolSpin": true,
  //     "favicon": "https://example.com/favicon.ico",
  //     "light": {
  //       "logo": "https://example.com/logo_light.png",
  //       "symbol": "https://example.com/search_symbol_light_24x24.png"
  //     }
  //   }

  // HTML to inject at the bottom of the `<body>` element on each page, for analytics scripts. Requires env var ENABLE_INJECT_HTML=true.
  "htmlBodyBottom": null,

  // HTML to inject at the top of the `<body>` element on each page, for analytics scripts. Requires env var ENABLE_INJECT_HTML=true.
  "htmlBodyTop": null,

  // HTML to inject at the bottom of the `<head>` element on each page, for analytics scripts. Requires env var ENABLE_INJECT_HTML=true.
  "htmlHeadBottom": null,

  // HTML to inject at the top of the `<head>` element on each page, for analytics scripts. Requires env var ENABLE_INJECT_HTML=true.
  "htmlHeadTop": null,
  
}
```


配置文档：
* [Site configuration](https://docs.sourcegraph.com/admin/config/site_config)
* [Global and user settings](https://docs.sourcegraph.com/admin/config/settings) - 配置用户操作行为
* [Sourcegraph HTTP and HTTPS/SSL configuration](https://docs.sourcegraph.com/admin/http_https_configuration)
* [Rate limits](https://docs.sourcegraph.com/admin/external_service/rate_limits)
* Code host connections
  * [配置 GitLab 代码仓库](https://docs.sourcegraph.com/admin/external_service/gitlab)
* [User authentication](https://docs.sourcegraph.com/admin/auth)
* [SMTP and email delivery](https://docs.sourcegraph.com/admin/config/email)


## 3. 参考

* [Sourcegraph Enterprise vs. Sourcegraph OSS](https://docs.sourcegraph.com/getting-started/oss-enterprise)
* [GitHub code search vs. Sourcegraph](https://docs.sourcegraph.com/getting-started/github-vs-sourcegraph)
* [GitHub 代码搜索语法](https://docs.github.com/zh/search-github/github-code-search/understanding-github-code-search-syntax)


