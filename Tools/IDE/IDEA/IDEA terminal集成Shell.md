## IDEA Window环境terminal集成Shell

IDEA的terminal可以集成Git Bash或者Cygwin，这里主要说下集成Git Bash。非常的简单。

1. 下载安装Git客户端，这个非常简单，就不详说了。

2. `File -> Settings -> Tools -> Terminal`界面`Shell path` 选项输入`"C:\Program Files\Git\bin\sh.exe" -login -i`，不要忘记引号，保存就OK了。

> 注：当你系统中运行了有道词典时，且开启了“划词”功能，这时你拖动IDEA的终端时会触发一次`CTRL + C`的效果。非常让人抓狂，
这时只需要关闭有道的“划词”功能或干脆关闭有道软件即可。

