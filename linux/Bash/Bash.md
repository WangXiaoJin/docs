# Bash

## 常用

### 快捷键

* `Ctrl + L`：清除屏幕并将当前行移到页面顶部。
* `Shift + PageUp`：向上滚动。
* `Shift + PageDown`：向下滚动。
* `Ctrl + U`：从光标位置删除到行首。
* `Ctrl + K`：从光标位置删除到行尾。
* `Ctrl + a`：移到行首。
* `Ctrl + b`：向行首移动一个字符，与左箭头作用相同。
* `Ctrl + e`：移到行尾。
* `Ctrl + f`：向行尾移动一个字符，与右箭头作用相同。
* `Alt + f` ：移动到当前单词的词尾。
* `Alt + b` ：移动到当前单词的词首。
* `Ctrl + v`：将下一个输入的特殊字符变成字面量，比如回车变成^M。
* `Ctrl + [`：等同于 ESC。
* `Alt + .` ：插入上一个命令的最后一个词。
* `Alt + _` ：等同于Alt + .。

> 参考地址：[行操作 - 阮一峰Bash脚本教程](https://www.bookstack.cn/read/bash-tutorial/docs-readline.md)

### [shell 中各种括号的作用`()`、`(())`、`[]`、`[[]]`、`{}`](https://www.runoob.com/w3cnote/linux-shell-brackets-features.html)

### [shell 判断变量是否为空，变量加不加双引号的区别](https://blog.csdn.net/huyuan7494/article/details/73469994)

### [Bash 脚本 set 命令](http://www.ruanyifeng.com/blog/2017/11/bash-set.html)
    * `set -u`
    * `set -x`
    * `set -e`
    * `set -o pipefail`

###  `cat << EOF`

    ```bash
    # $HOSTNAME变量会被替换
    shell> cat << EOF > a.log
    $HOSTNAME
    EOF
    ```

    ```bash
    # $HOSTNAME变量原样输出
    shell> cat << 'EOF' > a.log
    $HOSTNAME
    EOF
    ```

    ```bash
    # “<<-” 会过滤掉内容和EOF前面的“制表符”，不是空格
    shell> cat <<- EOF > a.log
        $HOSTNAME
        EOF
    ```

### `fg`、`bg`、`jobs`、`&`、`ctrl + z`

> [Linux 中`fg`、`bg`、`jobs`、`&`、`ctrl + z`等指令](https://ehlxr.me/2017/01/18/Linux-%E4%B8%AD-fg%E3%80%81bg%E3%80%81jobs%E3%80%81-%E6%8C%87%E4%BB%A4/)


### `trap`

trap命令用于指定在接收到信号后将要采取的动作，常见的用途是在脚本程序被中断时完成清理工作。当shell接收到sigspec指定的信号时，
arg参数（命令）将会被读取，并被执行。

> [trap命令](https://man.linuxde.net/trap)

### `command -v`

* [SHELL脚本中command -v 用法](https://blog.csdn.net/weixin_37991446/article/details/108700679)
* [command 命令详解](https://pubs.opengroup.org/onlinepubs/009604499/utilities/command.html)

### Awk

awk是处理文本文件的一个应用程序。awk其实不仅仅是工具软件，还是一种编程语言。
* [Awk - 官方文档](https://www.gnu.org/software/gawk/manual/gawk.html)
* [Awk - 阮一峰 Bash 脚本教程](https://www.bookstack.cn/read/bash-tutorial/docs-archives-commands-awk.md)
* [30 Examples for Awk Command in Text Processing](https://likegeeks.com/awk-command/)

### cut

cut命令用于在命令行输出文本文件的指定位置的内容。
* [cut - 阮一峰 Bash 脚本教程](https://www.bookstack.cn/read/bash-tutorial/docs-archives-commands-cut.md)

### sed

sed是一个强大的文本编辑工具。
* [sed - 阮一峰 Bash 脚本教程](https://www.bookstack.cn/read/bash-tutorial/docs-archives-text.md#5hsazy)

### JSON

* [jq - Github](https://github.com/stedolan/jq)
    * [jq Manual](https://stedolan.github.io/jq/manual/)
    * [jqplay](https://jqplay.org/) - 在线测试工具

    ```shell script
    # 解析JSON格式的日志文件，转换成普通日志格式
    # 1. `. as $log` - 设置变量$log，目的：当执行 try 后面的表达式报错后，使用catch返回$log原始数据
    # 2. `fromjson` - 格式化原始应用日志的json数据
    # 3. `[.["@timestamp"], .level, .thread, .logger, .msg, .throwable] | join(" ")` - 拼装日志数据，转换成普通日志格式
    jq -rR '. as $log | try ( fromjson | [.["@timestamp"], .level, .thread, .logger, .msg, .throwable] | join(" ") ) catch $log' app-log.json
    
    # 解析K8S控制台的JSON格式日志，转换成普通日志格式
    kubectl logs -f --tail=100 {podname} -n {namespace} | jq -rR '. as $log | try ( fromjson | [.["@timestamp"], .level, .thread, .logger, .msg, .throwable] | join(" ") ) catch $log'
    
    # 解析Docker的json格式日志文件，转换成普通日志格式
    # 1. `.log as $log` - 设置变量$log，目的：当执行 try 后面的表达式报错后，使用catch返回$log原始数据
    # 2. `fromjson` - 格式化原始应用日志的json数据
    # 3. `[.["@timestamp"], .level, .thread, .logger, .msg, .throwable] | join(" ")` - 拼装日志数据，转换成普通日志格式
    jq -r '.log as $log | .log | try ( fromjson | [.["@timestamp"], .level, .thread, .logger, .msg, .throwable] | join(" ") ) catch $log' xxx-json.log
    ```

* [jo - Github](https://github.com/jpmens/jo)

### 管道/重定向

* [sh command: exec 2>&1](https://stackoverflow.com/questions/1216922/sh-command-exec-21)
* [Bash One-Liners Explained, Part III: All about redirections](https://catonmat.net/bash-one-liners-explained-part-three)
* [Difference between `>/dev/null 2>&1 &` and `</dev/null &>/dev/null &`](https://unix.stackexchange.com/questions/497207/difference-between-dev-null-21-and-dev-null-dev-null)
* [重定向](https://www.bookstack.cn/read/bash-tutorial/docs-archives-redirection.md) - 阮一峰 Bash 脚本教程
* [管道和重定向基础](https://www.cnblogs.com/f-ck-need-u/p/7325378.html) - 骏马金龙
* [彻底搞懂shell的高级I/O重定向](https://www.junmajinlong.com/shell/fd_duplicate/) - 骏马金龙
* [Shell脚本深入教程：Bash高级重定向](https://www.junmajinlong.com/shell/script_course/shell_redirection/) - 骏马金龙
* [一个命令行解析和重定向的问题分析](https://www.junmajinlong.com/shell/cmdline_parse_and_redirect/) - 骏马金龙 - `(ls /err /tmp 2>&1) >a.log 2>err.log`

## 参考文档

* [GNU Bash Manual](https://www.gnu.org/software/bash/manual/) - 【重要】Bash语法官方文档
* [Shell语法详细文档](https://ldp.huihoo.org/LDP/abs/html/) - 【重要】
* [学习 Shell 与 Shell scripts - `鸟哥的 Linux 私房菜`](http://cn.linux.vbird.org/linux_basic/linux_basic.php#part3)
* [Shell 教程 - `RUNOOB`](https://www.runoob.com/linux/linux-shell.html)
* [阮一峰 Bash 脚本教程](https://www.bookstack.cn/read/bash-tutorial/README.md) - 【重要】
    * [模式扩展](https://www.bookstack.cn/read/bash-tutorial/docs-expansion.md)
    * [引号和转义](https://www.bookstack.cn/read/bash-tutorial/docs-quotation.md)
    * [变量](https://www.bookstack.cn/read/bash-tutorial/docs-variable.md)
    * [条件判断](https://www.bookstack.cn/read/bash-tutorial/docs-condition.md)
    * [循环](https://www.bookstack.cn/read/bash-tutorial/docs-loop.md)
    * [脚本除错](https://www.bookstack.cn/read/bash-tutorial/docs-debug.md)
    * [正则表达式](https://www.bookstack.cn/read/bash-tutorial/docs-archives-regex.md)
    * [命令的连续执行](https://www.bookstack.cn/read/bash-tutorial/docs-archives-command.md#5nurv3)
        * 组命令
        * 子shell
        * 进程替换
* [Shell系列文章](https://www.junmajinlong.com/shell/index/) - 骏马金龙