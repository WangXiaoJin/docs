## IDEA生成Getter/Setter Boolean兼容Eclipse

IDEA默认生成Boolean规则如下：

```java
class Test {
        
    private boolean isAdmin;
    
    private Boolean isLeader;

    public boolean isAdmin() {
        return isAdmin;
    }

    public void setAdmin(boolean admin) {
        isAdmin = admin;
    }

    public Boolean getLeader() {
        return isLeader;
    }

    public void setLeader(Boolean leader) {
        isLeader = leader;
    }
}
```

`IDEA` `boolean`类型的生成规则和`Eclipse`一样。`IDEA`  `Boolean`处理流程和`boolean`一样，
而`Eclipse`则不会处理`Boolean`类型的`is`前缀。如果想让IDEA处理`Boolean`和`Eclipse`一样，则配置如下：

在POJO类中 `右键` --> `Generate...` --> `Getter and Setter` --> 点击`Getter template`/`Setter template`右侧按钮 --> 
点击`+`按钮 --> 分别添加如下模板：

* Name：`Boolean Getter - Eclipse`
    ```
    #if($field.modifierStatic)
    static ##
    #end
    $field.type ##
    #set($name = $StringUtil.capitalizeWithJavaBeanConvention($StringUtil.sanitizeJavaIdentifier($helper.getPropertyName($field, $project))))
    #if ($field.boolean && $field.primitive)
    is##
    #elseif($field.boolean && $StringUtil.startsWithIgnoreCase($field.name, "is"))
    getIs##
    #else
    get##
    #end
    ${name}() {
    return $field.name;
    }
    ```

* Name：`Boolean Setter - Eclipse`
    ```
    #set($paramName = $helper.getParamName($field, $project))
    #if($field.modifierStatic)
    static ##
    #end
    #set($name = $StringUtil.capitalizeWithJavaBeanConvention($StringUtil.sanitizeJavaIdentifier($helper.getPropertyName($field, $project))))
    #if($field.boolean && !$field.primitive && $StringUtil.startsWithIgnoreCase($field.name, "is"))
        #set($name = "Is" + $name)
        #set($paramName = $field.name)
    #end
    void set${name}($field.type $paramName) {
    #if ($field.name == $paramName)
        #if (!$field.modifierStatic)
        this.##
        #else
            $classname.##
        #end
    #end
    $field.name = $paramName;
    }
    ```

> 注：更好的方案是POJO中禁止`boolean/Boolean`类型的属性以`is`开头，属性名用形容词格式。
