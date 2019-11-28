# IDEA常用配置及插件

## 快捷键

复制一份Default快捷键配置，在新拷贝的配置中修改：

`File | Settings | Keymap | Default | Duplicate`

如果在修改快捷键时提示信息：`The shortcut already assigned to other actions. Do you want to remove other assignments.`，
说明你配置的快捷键已被占用，记住占用的快捷键项，直接点击 `Leave`，不用管它，如果出问题后再修改占用快捷键项。

* Editor Actions | Delete Line = Ctrl+D
* Editor Actions | Duplicate Entire Lines = Ctrl+Alt+向下箭头
* Editor Actions | Scroll to Bottom = Alt+B
* Editor Actions | Scroll to Top = Alt+T
* Main menu | Code | Completion | Basic = Ctrl+空格 Alt+/
* Main menu | Code | Completion | SmartType = Ctrl+Shift+空格 Alt+Shift+/
* Main menu | Code | Completion | Cyclic Expand Word = Alt+.
* Main menu | Code | Completion | Cyclic Expand Word (Backward) = Alt+Shift+.

## 常用配置

* 当SCM子文件有改变时，上级所有文件夹都标记

    `File | Settings | Version Control | 勾选Show directories with changed descendants`

* 修改字体大小

    `File | Settings | Editor | Font | Size: 14`

* 修改文本的背景色

    `File | Settings | Editor | Color Scheme | General | Text | Default text | 右侧Background: F3F1F1`

* 配置Maven settings.xml文件

    `File | Settings | Build, Execution, Deployment | Build Tools | Maven | User settings file`

* Terminal使用Linux环境

    `File | Settings | Tools | Terminal | Shell path | "C:\Program Files\Git\bin\sh.exe" -login -i`

* SCM忽略IDEA自动生成的*.iml文件

    `File | Settings | Version Control | Ignored Files | 点击新增按钮 | 选择Ignore all files matching | 输入 *.iml`

* IDEA转换*.properties中文

    `File | Settings | Editor | File Encodings`
    
    配置如下：
    ```
    Global Encoding：UTF-8
    Project Encoding：UTF-8
    Default encoding for properties files：UTF-8
    选中Transparent native-to-ascii conversion
    ```
    

* 创建Java类，自动生成`@author`/`@date` JavaDoc

    `File | Settings | Editor | File and Code Templates | Includes | File Header `
    
    配置内容如下：
    
    ```
    /**
     *
     * @author WangXiaoJin
     * @date ${YEAR}-${MONTH}-${DAY} ${TIME}
     */ 
    ```
    
    > 注：如果你想默认使用系统的时间格式，请换成`${DATE} ${TIME}`

* 使用`Live Templates`自动生成`@author`、`@date`等一系列JavaDoc

    `File | Settings | Editor | Live Templates`
    
    1. 点击右侧 `+` 选择 `Template Group` ，输入`注释`，点击 `OK`
    
    2. 在新建的`注释`组下面，新建`@author` Live Template
        * `Abbreviation`：`@author`
        * `Description`: `生成@author 注解`
        * `Template text`: `@author $author$`
        * `Define applicable contexts`: 选择所有
        * 选中`Reformat according to style`
        * 点击`Edit variables` -> `Expression` -> `author`: `user()`
        
    3. 在新建的`注释`组下面，新建`@date` Live Template
        * `Abbreviation`：`@date`
        * `Description`: `生成@date 注释`
        * `Template text`: `@date $date$ $time$`
        * `Define applicable contexts`: 选择所有
        * 选中`Reformat according to style`
        * 点击`Edit variables` -> `Expression`
            * `date`: `date("yyyy-MM-dd")`
            * `time`: `time()`


## Inspections

* Serializable类必须包含serialVersionUID属性

    `File | Settings | Editor | Inspections | Java | Serialization issues | Serializable class without 'serialVersionUID' | 选中`

* Java导入了没有使用的Class，调整到Error级别

    `File | Settings | Editor | Inspections | Java | Imports | Unused import | Severity -> Error`


## 常见问题

#### IDEA 2018 字体模糊，尤其在使用Markdown时。解决方案是删除 IDEA 安装目录下的`jre64`：[参考地址](https://blog.csdn.net/zaemyn2015/article/details/84584458)

注：下载`Markdown Navigator`也可解决此字体模糊的问题，需要禁用默认的`Markdown support`。但`Markdown Navigator` 的
`Preview Browser` 默认为 `Default - swing`，在预览外网的图片时非常卡。建议切换到`JavaFX WebView`模式，
在此模式下预览外网图片不会卡顿，但你此时发现字体又模糊了。`Markdown Navigator`比IDEA原有Markdown功能强大，
所以最终建议方案是：  

1. 删除 IDEA 安装目录下的`jre64`
2. 安装`Markdown Navigator`插件，禁用原有的`Markdown support`
3. `File | Settings | Languages & Frameworks | Markdown | Preview`
* `Preview Browser` - 选择 `JavaFX WebView`
* `Page Zoom` - 填 `1.2` （用于放大预览页面的字体）

> Markdown 超出当前可见视图则自动换行显示：`File | Settings | Languages & Frameworks | Markdown | Editor` => `Soft Wrap: Enabled`

## Plugins

* Auto filling Java call arguments - 自动填充调用的方法、构造函数参数
* BashSupport - Bash语法及高亮显
* CheckStyle-IDEA - CheckStyle
* Alibaba Java Coding Guidelines - 阿里代码规范
* Kubernetes and OpenShift Resource Support - 编辑K8S、OpenShift资源文件
* Lombok Plugin 
* Maven Helper 
* SonarLint 
* String Manipulation - 字符窜各种大小写/驼峰规则转换
* Swagger Plugin 
* vue.js
* Eclipse Code Formatter 
* Nginx Support - 支持Nginx配置自动补全、开启/关闭Nginx服务

