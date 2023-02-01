# MobaXterm

Enhanced terminal for Windows with X11 server, tabbed SSH client, network tools and much more.

## 安装

进入 [下载](https://mobaxterm.mobatek.net/download-home-edition.html) 页面下载` MobaXterm Portable`免安装版本。

## 配置

点击 `Settings` -> `Configuration`

### 1. 配置 Password

`General` -> `MobaXterm passwords management` -> 选择`Credentials` Tab -> 点击`New`添加用户 -> 点击`OK`

* `Name` - 表示此用户的标识名
* `Username` - 连接目标地址的用户名
* `Password` - 连接目标地址的密码

> 注：添加用户后，新建Session时可以选择已添加的用户，而无需再输入用户名、密码。

> 注：下面的`Master Password settings`是配置整个软件的密码。

### 2. 修改 Session 默认配置

`General` -> `Edit my sessions presets` -> `SSH` 

* `Specify username` -> 选择默认用户，使用此用户连接SSH
* `Advanced SSH settings` -> 勾选`Follow SSH path` - 作用：当终端进入另一个目录时，SFTP会自动跟随。
* `Advanced SSH settings` -> 勾选`Use private key` - 作用：尝试使用此私钥连接SSH

### 3. 配置SSH默认登录用户名

`SSH` -> `SSH settings` -> Default login - 作用：`Specify username`功能的备选功能，只起到简化输入用户名
