## IDEA常用配置及插件

### 快捷键

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

### 常用配置

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

### Inspections

* Serializable类必须包含serialVersionUID属性

    `File | Settings | Editor | Inspections | Java | Serialization issues | Serializable class without 'serialVersionUID' | 选中`

* Java导入了没有使用的Class，调整到Error级别

    `File | Settings | Editor | Inspections | Java | Imports | Unused import | Severity -> Error`

### Plugins

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

