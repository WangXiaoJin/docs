## IDEA常用配置及插件

### 快捷键

### 常用配置

* 创建Java类，自动生成`@author`/`@date` JavaDoc

    `Window | Preferences | Java | Code Style | Code Templates | Comments | Types | Edit`
    
    配置内容如下：
    
    ```
    /**
     * @author ${user}
     * @date ${d:date('yyyy-MM-dd HH:mm')}
     *
     * ${tags}
     */
    ```
    
    > 注：勾选`Automatically add comments for new methods and types`，让创建类时自动生成。

### Plugins

* CheckStyle-IDEA - CheckStyle
* SonarLint 

