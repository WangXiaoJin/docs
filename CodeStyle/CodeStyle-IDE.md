## CodeStyle-IDE

#### Eclipse应用Google Code Style

* 下载地址：https://github.com/google/styleguide/blob/gh-pages/eclipse-java-google-style.xml
* 安装之前需要修改几处配置项
    * `<setting id="org.eclipse.jdt.core.formatter.tabulation.size" value="4"/>` - 缩进习惯了4个空格
    * `<setting id="org.eclipse.jdt.core.formatter.continuation_indentation" value="1"/>` - 连续缩进设置为1
    * `<setting id="org.eclipse.jdt.core.formatter.continuation_indentation_for_array_initializer" value="1"/>`
    * `<setting id="org.eclipse.jdt.core.formatter.lineSplit" value="120"/>` - 代码最大长度为120
    * `<setting id="org.eclipse.jdt.core.formatter.comment.line_length" value="120"/>` - 注释最大长度为120
    * `<setting id="org.eclipse.jdt.core.formatter.number_of_empty_lines_to_preserve" value="1"/>` - 保留的空行数
    * `<setting id="org.eclipse.jdt.core.formatter.alignment_for_enum_constants" value="16"/>` - 枚举对象太多时换行
* 安装文档参考[Checkstyle.md](Checkstyle.md)

#### Eclipse设置JS、CSS、HTML、XML

* 通用

  * `General | Editors | Text Editors | Displayed tab width : 2`
  * `General | Editors | Text Editors | 勾选 Insert spaces for tabs`
  
* JS

  配置路径：`JavaScript | Code Style | Formatter | 选中对应profile点击Edit`

  * 缩进使用空格替代且为2个空格
    * `Indentation | Tab policy | Spaces only`
    * `Indentation | Indentation size | 2`
    * `Indentation | Tab size | 2`
    
* CSS
  * 缩进使用空格替代
  
    `Web | CSS Files | Editor -> 选中Indent using spaces 且 Indentation size设置为2`
    
* HTML
  * 缩进使用空格替代
  
    `Web | HTML Files | Editor -> 选中Indent using spaces 且 Indentation size设置为2`

* XML
  * 缩进使用空格替代
  
    `XML | XML Files | Editor -> 选中Indent using spaces 且 Indentation size设置为2`

* JSON
  * 缩进使用空格替代
  
    `JSON | JSON Files | Editor -> 选中Indent using spaces 且 Indentation size设置为2`

#### IDEA应用Google Code Style
    
* 下载地址：https://github.com/google/styleguide/blob/gh-pages/intellij-java-google-style.xml
* 安装之前需要修改几处配置项
    * 只修改Java语言`<codeStyleSettings language="JAVA">`：
      * `<option name="INDENT_SIZE" value="4" />`
      * `<option name="CONTINUATION_INDENT_SIZE" value="4" />`
      * `<option name="TAB_SIZE" value="4" />`
      * 增加配置项：`<option name="ENUM_CONSTANTS_WRAP" value="1" />` - 枚举对象太多时换行
      * 删除JavaDoc及Comments配置项，使用默认值：
        * `<option name="JD_P_AT_EMPTY_LINES" value="false" />`
        * `<option name="JD_KEEP_EMPTY_PARAMETER" value="false" />`
        * `<option name="JD_KEEP_EMPTY_EXCEPTION" value="false" />`
        * `<option name="JD_KEEP_EMPTY_RETURN" value="false" />`
        * `<option name="WRAP_COMMENTS" value="true" />`
    * 修改所有语言：`<option name="RIGHT_MARGIN" value="120" />`
    * 增加JSP配置：
      ```xml
      <codeStyleSettings language="JSP">
        <indentOptions>
          <option name="INDENT_SIZE" value="2" />
          <option name="CONTINUATION_INDENT_SIZE" value="4" />
          <option name="TAB_SIZE" value="2" />
        </indentOptions>
      </codeStyleSettings>
      ```
    * 增加JSPX配置：
      ```xml
      <codeStyleSettings language="JSPX">
        <indentOptions>
          <option name="INDENT_SIZE" value="2" />
          <option name="CONTINUATION_INDENT_SIZE" value="4" />
          <option name="TAB_SIZE" value="2" />
        </indentOptions>
      </codeStyleSettings>
      ```
    * 增加Velocity配置：
      ```xml
      <codeStyleSettings language="VTL">
        <indentOptions>
          <option name="INDENT_SIZE" value="2" />
          <option name="CONTINUATION_INDENT_SIZE" value="4" />
          <option name="TAB_SIZE" value="2" />
        </indentOptions>
      </codeStyleSettings>
      ```
* 安装文档参考[Checkstyle.md](Checkstyle.md)

