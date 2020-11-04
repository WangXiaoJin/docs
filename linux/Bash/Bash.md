## Bash

* [GNU Bash Manual](https://www.gnu.org/software/bash/manual/) - 【重要】Bash语法官方文档
* [Shell语法详细文档](https://ldp.huihoo.org/LDP/abs/html/) - 【重要】
* [学习 Shell 与 Shell scripts - `鸟哥的 Linux 私房菜`](http://cn.linux.vbird.org/linux_basic/linux_basic.php#part3)

* [Shell 教程 - `RUNOOB`](https://www.runoob.com/linux/linux-shell.html)

* [shell 中各种括号的作用`()`、`(())`、`[]`、`[[]]`、`{}`](https://www.runoob.com/w3cnote/linux-shell-brackets-features.html)

* [shell 判断变量是否为空，变量加不加双引号的区别](https://blog.csdn.net/huyuan7494/article/details/73469994)

* [Bash 脚本 set 命令](http://www.ruanyifeng.com/blog/2017/11/bash-set.html)

*  `cat << EOF`

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




