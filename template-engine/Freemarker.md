# Freemarker

### 总结

* 不存在值检测操作符：
    * `unsafe_expr??`
    * `(unsafe_expr)??`
    * `exp1?exists` - 已废弃

* escape
    * escape directive：[`escape`, `noescape`](http://freemarker.foofun.cn/ref_directive_escape.html)
    * escape sequences：[字符窜](http://freemarker.foofun.cn/dgui_template_exp.html#dgui_template_exp_direct_string)
    * escaping
        * output: [html](http://freemarker.foofun.cn/ref_builtins_string.html#ref_builtin_html), 
            [rtf](http://freemarker.foofun.cn/ref_builtins_string.html#ref_builtin_rtf), 
            [xhtml](http://freemarker.foofun.cn/ref_builtins_string.html#ref_builtin_xhtml), 
            [xml](http://freemarker.foofun.cn/ref_builtins_string.html#ref_builtin_xml)
        * URL: [url](http://freemarker.foofun.cn/ref_builtins_string.html#ref_builtin_url)
        * URL path: [url_path](http://freemarker.foofun.cn/ref_builtins_string.html#ref_builtin_url_path)

### 文档

* [Freemarker中文官方手册](http://freemarker.foofun.cn/)
