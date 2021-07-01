# Git

## Git

### 参考文档

* [官方文档](https://git-scm.com/docs)
    * [git tutorial](https://git-scm.com/docs/gittutorial)
    * [git everyday](https://git-scm.com/docs/giteveryday)
    * [Git User’s Manual](https://git-scm.com/docs/user-manual)
    * [git-config](https://git-scm.com/docs/git-config)
        * [配置文件`include`/`includeIf`-【重要】](https://git-scm.com/docs/git-config#_includes)
    * [Git命令行使用规范 -【重要】](https://git-scm.com/docs/gitcli)
    * [Git Repository Layout](https://git-scm.com/docs/gitrepository-layout)
    * [Specifying revisions and ranges for Git -【重要】](https://git-scm.com/docs/gitrevisions)
    * [可配环境变量](https://git-scm.com/docs/git#_environment_variables)
    * Credential
        * [gitcredentials[7]](https://git-scm.com/docs/gitcredentials)
        * [git-credential](https://git-scm.com/docs/git-credential)
        * [git-credential-cache](https://git-scm.com/docs/git-credential-cache)
        * [git-credential-store](https://git-scm.com/docs/git-credential-store)
    * [Frequently asked questions](https://git-scm.com/docs/gitfaq)
        * How do I configure a different editor?
        * How do I specify my credentials when pushing over HTTP?
        * How do I read a password or token from an environment variable?
        * How do I change the password or token I’ve saved in my credential manager?
        * How do I use multiple accounts with the same hosting provider using HTTP?
* [Pro Git book -【官网】](https://git-scm.com/book/zh/v2)
    * `1.6章节 - Git配置文件`
    * `2.2 Git 基础 - 记录每次更新到仓库` - Git常用基本命令
    * `2.7 Git 基础 - Git 别名` - 给命令设置别名
* [Pro Git v2 中文版 - 备用地址](https://doc.yonyoucloud.com/doc/wiki/project/pro-git-two/index.html)
    * Git迁移
        * [迁移到 Git](https://doc.yonyoucloud.com/doc/wiki/project/pro-git-two/move-to-git.html)
        * [SVN migration](https://training.github.com/downloads/subversion-migration/)
    * 内部原理
        * [环境变量](https://doc.yonyoucloud.com/doc/wiki/project/pro-git-two/environment-variable.html)
    * Git命令自动补全
        * [Bash 中的 Git](https://doc.yonyoucloud.com/doc/wiki/project/pro-git-two/bash.html) - 配置`git-completion.bash`及`git-prompt.sh`
        * [Zsh 中的 Git](https://doc.yonyoucloud.com/doc/wiki/project/pro-git-two/zsh.html) - 配置`git-completion.zsh`及`git-prompt.sh`
        * [Powershell 中的 Git](https://doc.yonyoucloud.com/doc/wiki/project/pro-git-two/powershell.html)
    * 将 Git 嵌入你的应用
        * [命令行 Git 方式](https://doc.yonyoucloud.com/doc/wiki/project/pro-git-two/command-line-git.html)
        * [Libgit2](https://doc.yonyoucloud.com/doc/wiki/project/pro-git-two/libgit2.html)
        * [JGit](https://doc.yonyoucloud.com/doc/wiki/project/pro-git-two/jgit.html)

* [Git教程 - 腾讯云开发者手册](https://cloud.tencent.com/developer/doc/1096)

* [Git简易使用手册](https://training.github.com/downloads/zh_CN/github-git-cheat-sheet/)
* [Submodules vs. Subtrees](https://training.github.com/downloads/submodule-vs-subtree-cheat-sheet/)

## Git Large File Storage（Git LFS）

* [官网](https://git-lfs.github.com/)
* [GitLab Git Large File Storage (LFS) Administration](https://docs.gitlab.com/ee/administration/lfs/)

## GitLab

### 问题

* [GitLab 502问题的解决](https://www.cnblogs.com/linkenpark/p/8405327.html)

### 参考文档

* [官方文档](https://docs.gitlab.com/)
    * [安装](https://about.gitlab.com/install/)
        * [Install Requirements](https://docs.gitlab.com/ee/install/requirements.html)
    * Omnibus GitLab
        * [Package defaults](https://docs.gitlab.com/omnibus/package-information/defaults.html) - 默认的启用组件/通信协议/端口
        * [OS Versions that are no longer supported](https://docs.gitlab.com/omnibus/package-information/deprecated_os.html)
        * [GitLab Docker images](https://docs.gitlab.com/omnibus/docker/README.html)
        * [Manually download and install a GitLab package](https://docs.gitlab.com/omnibus/manual_install.html)
        * [Configuring Omnibus GitLab](https://docs.gitlab.com/omnibus/settings/README.html)
        * [Maintenance commands](https://docs.gitlab.com/omnibus/maintenance/README.html)
    * [Reference architectures](https://docs.gitlab.com/ee/administration/reference_architectures/) - Gitlab架构
        * [Reference architecture: up to 1,000 users](https://docs.gitlab.com/ee/administration/reference_architectures/1k_users.html)
        * [Reference architecture: up to 2,000 users](https://docs.gitlab.com/ee/administration/reference_architectures/2k_users.html)
        * [Reference architecture: up to 3,000 users](https://docs.gitlab.com/ee/administration/reference_architectures/3k_users.html)
        * [Troubleshooting a reference architecture setup](https://docs.gitlab.com/ee/administration/reference_architectures/troubleshooting.html)
    * Administrator
        * Backup/Restore/Migrate/Import
            * [Back up and restore GitLab](https://docs.gitlab.com/ee/raketasks/backup_restore.html)
            * [Backups](https://docs.gitlab.com/omnibus/settings/backups.html)
            * [Moving repositories](https://docs.gitlab.com/ee/administration/operations/moving_repositories.html)
            * [Import bare repositories](https://docs.gitlab.com/ee/raketasks/import.html)
            * [Project import/export administration](https://docs.gitlab.com/ee/administration/raketasks/project_import_export.html)
            * [Uploads migrate](https://docs.gitlab.com/ee/administration/raketasks/uploads/migrate.html)
            * [Group Import/Export](https://docs.gitlab.com/ee/user/group/settings/import_export.html)
            * [Group Import/Export API](https://docs.gitlab.com/ee/api/group_import_export.html)
            * [Import groups from another instance of GitLab](https://docs.gitlab.com/ee/user/group/import/)
            * [Project Import/Export](https://docs.gitlab.com/ee/user/project/settings/import_export.html)
            * [Project Import/Export API](https://docs.gitlab.com/ee/api/project_import_export.html)
            * [Migrate projects to a GitLab instance](https://docs.gitlab.com/ee/user/project/import/) - 包含所有来源
            * [Project/group import/export rate limits](https://docs.gitlab.com/ee/user/admin_area/settings/import_export_rate_limits.html)
        * Feature Flags
            * [Enable and disable GitLab features deployed behind feature flags](https://docs.gitlab.com/ee/administration/feature_flags.html)
            * [GitLab functionality may be limited by feature flags](https://docs.gitlab.com/ee/user/feature_flags.html)
        * Hooks
            * [System hooks](https://docs.gitlab.com/ee/system_hooks/system_hooks.html)
            * [Webhooks](https://docs.gitlab.com/ee/user/project/integrations/webhooks.html)
            * [File hooks](https://docs.gitlab.com/ee/administration/file_hooks.html)
            * [Server hooks](https://docs.gitlab.com/ee/administration/server_hooks.html)
            * [Webhooks and insecure internal web services](https://docs.gitlab.com/ee/security/webhooks.html)
            * [Webhooks administration](https://docs.gitlab.com/ee/raketasks/web_hooks.html)
        * [Git Protocol v2](https://docs.gitlab.com/ee/administration/git_protocol.html)
        * [Global user settings](https://docs.gitlab.com/ee/administration/user_settings.html)
            * Disallow users creating top-level groups
            * Disallow users changing usernames
        * [Invalidate Markdown Cache](https://docs.gitlab.com/ee/administration/invalidate_markdown_cache.html)
        * [Merge request diffs storage](https://docs.gitlab.com/ee/administration/merge_request_diffs.html)
        * [Moderate users](https://docs.gitlab.com/ee/user/admin_area/moderate_users.html)
            * Blocking and unblocking users
            * Activating and deactivating users
            * Ban and unban users
        * Repository storage
            * [Repository storage types](https://docs.gitlab.com/ee/administration/repository_storage_types.html)
                * From project name to hashed path
                * From hashed path to project name
            * [How Git object deduplication works in GitLab](https://docs.gitlab.com/ee/development/git_object_deduplication.html)
        * Rails console
            * [Rails console](https://docs.gitlab.com/ee/administration/operations/rails_console.html)
            * [GitLab Rails Console Cheat Sheet](https://docs.gitlab.com/ee/administration/troubleshooting/gitlab_rails_cheat_sheet.html)
        * [General maintenance](https://docs.gitlab.com/ee/administration/raketasks/maintenance.html)
        * [Clean up](https://docs.gitlab.com/ee/raketasks/cleanup.html)
        * [Reduce repository size](https://docs.gitlab.com/ee/user/project/repository/reducing_the_repo_size_using_git.html)
        * [Integrity check](https://docs.gitlab.com/ee/administration/raketasks/check.html)
        * [Upgrading GitLab](https://docs.gitlab.com/ee/update/)
    * [Permissions and roles](https://docs.gitlab.com/ee/user/permissions.html)
    * [Code Intelligence](https://docs.gitlab.com/ee/user/project/code_intelligence.html)
    * [Description templates](https://docs.gitlab.com/ee/user/project/description_templates.html)
    * [Static Site Editor](https://docs.gitlab.com/ee/user/project/static_site_editor/)
    * [Branch filter search box](https://docs.gitlab.com/ee/user/project/repository/branches/#branch-filter-search-box)
        * `^feature` matches only branch names that begin with ‘feature’.
        * `feature$` matches only branch names that end with ‘feature’.
    * [Locked files](https://docs.gitlab.com/ee/user/project/file_lock.html)
    * [Repository mirroring](https://docs.gitlab.com/ee/user/project/repository/repository_mirroring.html)



