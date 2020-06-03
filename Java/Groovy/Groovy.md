# Groovy - README

## 安装

### Operating system/package manager installation

* Install Binary

  `Groovy 2.5` requires `Java 6+` with full support up to Java 8. There are currently some known
  issues for some aspects when using Java 9 snapshots. The `groovy-nio` module requires `Java 7+`.
  Using Groovy’s `invokeDynamic` features require `Java 7+` but we recommend `Java 8`.

  * [下载](http://groovy-lang.org/install.html#download-groovy) Groovy二进制包，并解压至你想要的目录
  * 配置`GROOVY_HOME`环境变量，指向你的安装目录
  * `PATH`环境变量追加`GROOVY_HOME/bin`
  * 配置`JAVA_HOME`环境变量

  创建一个交互式的 groovy shell：
  ```shell script
  $ groovysh
  ```

  运行 Swing interactive console：
  ```shell script
  $ groovyConsole
  ```

  执行指定的Groovy 脚本：
  ```shell script
  $ groovy SomeScript
  ```

  If you embed Groovy in your application. To use the
  [InvokeDynamic](http://docs.groovy-lang.org/latest/html/documentation/invokedynamic-support.html)
  version of the jars just append `:indy` for Gradle or `<classifier>indy</classifier>` for Maven.

* [SDKMAN!](http://sdkman.io/) is a tool for managing parallel versions of multiple Software
  Development Kits on most Unix-based systems:

  ```shell script
  $ sdk install groovy
  ```

  Windows users: see the SDKMAN [install](https://sdkman.io/install) instructions for potential options.

  > `SDKMAN`可以支持安装JDK及Java相关的工具（Maven、Ant、SpringBoot、Scala等），并支持同时安装多个版本

* [Homebrew](http://brew.sh/) is "the missing package manager for macOS":

  ```shell script
  $ brew install groovy
  ```

* [SnapCraft](https://snapcraft.io/) is "the app store for Linux". Groovy is supported in the
  [store](https://snapcraft.io/groovy) or via the commandline:

  ```shell script
  $ sudo snap install groovy --classic
  ```

* [MacPorts](http://www.macports.org/) is a system for managing tools on macOS:

  ```shell script
  $ sudo port install groovy
  ```

* [Scoop](http://scoop.sh/) is a command-line installer for Windows inspired by Homebrew:

  ```shell script
  > scoop install groovy
  ```

* [Chocolatey](https://chocolatey.org/) provides a sane way to manage software on Windows:

  ```shell script
  > choco install groovy
  ```

Linux/*nix users: you may also find Groovy is available using your preferred operating system
package manager, e.g.: apt, dpkg, pacman, etc.

> [参考文档](http://groovy-lang.org/download.html#osinstall)

## Note

### Default imports

All these packages and classes are imported by default, i.e. you do not have to use an explicit import statement to use them:

* java.io.*
* java.lang.*
* java.math.BigDecimal
* java.math.BigInteger
* java.net.*
* java.util.*
* groovy.lang.*
* groovy.util.*

### Static import aliasing

Static imports with the `as` keyword provide an elegant solution to namespace problems. Suppose you
want to get a `Calendar` instance, using its `getInstance()` method. It’s a static method, so we can
use a static import. But instead of calling `getInstance()` every time, which can be misleading when
separated from its class name, we can import it with an alias, to increase code readability:

```groovy
import static Calendar.getInstance as now

assert now().class == Calendar.getInstance().class
```

### Import aliasing

With type aliasing, we can refer to a fully qualified class name using a name of our choice. This
can be done with the as keyword, as before.

For example we can import `java.sql.Date` as `SQLDate` and use it in the same file as
`java.util.Date` without having to use the fully qualified name of either class:

```groovy
import java.util.Date
import java.sql.Date as SQLDate

Date utilDate = new Date(1000L)
SQLDate sqlDate = new SQLDate(1000L)

assert utilDate instanceof java.util.Date
assert sqlDate instanceof java.sql.Date
```


### Shebang line

Beside the single-line comment, there is a special line comment, often called the shebang line
understood by UNIX systems which allows scripts to be run directly from the command-line,
provided you have installed the Groovy distribution and the `groovy` command is available on the `PATH`.

```shell script
#!/usr/bin/env groovy
println "Hello from the shebang line"
```

> The `#` character must be the first character of the file. Any indentation would yield a compilation error.

### [Method pointer operator](http://groovy-lang.org/operators.html#method-pointer-operator)

The method pointer operator (`.&`) call be used to store a reference to a method in a variable, in
order to call it later:

```groovy
def str = 'example of method reference'            
def fun = str.&toUpperCase                         
def upper = fun()                                  
assert upper == str.toUpperCase()
```

There are multiple advantages in using method pointers. First of all, the type of such a method
pointer is a `groovy.lang.Closure`, so it can be used in any place a closure would be used. In
particular, it is suitable to convert an existing method for the needs of the strategy pattern:

```groovy
def transform(List elements, Closure action) {                    
    def result = []
    elements.each {
        result << action(it)
    }
    result
}
String describe(Person p) {                                       
    "$p.name is $p.age"
}
def action = this.&describe                                       
def list = [
    new Person(name: 'Bob',   age: 42),
    new Person(name: 'Julia', age: 35)]                           
assert transform(list, action) == ['Bob is 42', 'Julia is 35']
```

Method pointers are bound by the receiver and a method name. Arguments are resolved at runtime, meaning that if you have multiple methods with the same name, the syntax is not different, only resolution of the appropriate method to be called will be done at runtime:

```groovy
def doSomething(String str) { str.toUpperCase() }    
def doSomething(Integer x) { 2*x }                   
def reference = this.&doSomething                    
assert reference('foo') == 'FOO'                     
assert reference(123)   == 246   
```

### [Regular expression operators](http://groovy-lang.org/operators.html#_regular_expression_operators)

#### Pattern operator

The pattern operator (`~`) provides a simple way to create a `java.util.regex.Pattern` instance:

```groovy
def p = ~/foo/
assert p instanceof Pattern
```

while in general, you find the pattern operator with an expression in a slashy-string, it can be
used with any kind of `String` in Groovy:

```groovy
// using single quote strings
p = ~'foo'                
// using double quotes strings                                        
p = ~"foo"                     
// the dollar-slashy string lets you use slashes and the dollar sign without having to escape them                                   
p = ~$/dollar/slashy $ string/$       
// you can also use a GString!                            
p = ~"${pattern}"       
```

#### Find operator

Alternatively to building a pattern, you can directly use the find operator `=~` to build a
`java.util.regex.Matcher` instance:

```groovy
def text = "some text to match"
// `=~` creates a matcher against the `text` variable, using the pattern on the right hand side
def m = text =~ /match/     
// the return type of `=~` is a `Matcher`                                    
assert m instanceof Matcher         
// equivalent to calling `if (!m.find())`                  `            
if (!m) {                                                         
    throw new RuntimeException("Oops, text not found!")
}
```

Since a `Matcher` coerces to a `boolean` by calling its `find` method, the `=~` operator is
consistent with the simple use of Perl’s `=~` operator, when it appears as a predicate (in `if`,
`while`, etc.).

#### Match operator

The match operator (`==~`) is a slight variation of the find operator, that does not return a
`Matcher` but a boolean and requires a strict match of the input string:

```groovy
// `==~` matches the subject with the regular expression, but match must be strict
m = text ==~ /match/                  
// the return type of `==~` is therefore a `boolean`                          
assert m instanceof Boolean           
// equivalent to calling `if (text ==~ /match/)`                            
if (m) {                                                          
    throw new RuntimeException("Should not reach that point!")
}
```

### [Spread operator](http://groovy-lang.org/operators.html#_spread_operator) - 详情请参考文档

The Spread-dot Operator (`*.`), often abbreviated to just Spread Operator, is used to invoke an
action on all items of an aggregate object. It is equivalent to calling the action on each item and
collecting the result into a list:

```groovy
class Car {
    String make
    String model
}
def cars = [
       new Car(make: 'Peugeot', model: '508'),
       new Car(make: 'Renault', model: 'Clio')]       
def makes = cars*.make                                
assert makes == ['Peugeot', 'Renault']  
```

The spread operator is null-safe, meaning that if an element of the collection is null, it will
return null instead of throwing a `NullPointerException`:

```groovy
cars = [
   new Car(make: 'Peugeot', model: '508'),
   null,                                              
   new Car(make: 'Renault', model: 'Clio')]
assert cars*.make == ['Peugeot', null, 'Renault']     
assert null*.make == null    
```

以下功能请参考官方文档：
* Spreading method arguments
* Spread list elements
* Spread map elements

### Identity operator

In Groovy, using `==` to test equality is different from using the same operator in Java. In Groovy,
it is calling `equals`. If you want to compare reference equality, you should use `is` like in the
following example:

```groovy
// Create a list of strings
def list1 = ['Groovy 1.8','Groovy 2.0','Groovy 2.3']    
// Create another list of strings containing the same elements    
def list2 = ['Groovy 1.8','Groovy 2.0','Groovy 2.3']      
// using `==`, we test object equality  
assert list1 == list2           
// but using `is`, we can check that references are distinct                            
assert !list1.is(list2)    
```

### [Coercion operator](http://groovy-lang.org/operators.html#_coercion_operator)

The coercion operator (`as`) is a variant of casting. Coercion converts object from one type to
another without them being compatible for assignment. Let’s take an example:

```groovy
Integer x = 123
// `Integer` is not assignable to a `String`, so it will produce a `ClassCastException` at runtime
String s = (String) x  
```

This can be fixed by using coercion instead:

```groovy
Integer x = 123
// `Integer` is not assignable to a `String`, but use of `as` will coerce it to a `String`
String s = x as String   
```

When an object is coerced into another, unless the target type is the same as the source type,
coercion will return a new object. The rules of coercion differ depending on the source and target
types, and coercion may fail if no conversion rules are found. Custom conversion rules may be
implemented thanks to the `asType` method:

```groovy
class Identifiable {
    String name
}
class User {
    Long id
    String name
    def asType(Class target) {                                              
        if (target == Identifiable) {
            return new Identifiable(name: name)
        }
        throw new ClassCastException("User cannot be coerced into $target")
    }
}
def u = new User(name: 'Xavier')                                            
def p = u as Identifiable                                                   
assert p instanceof Identifiable                                            
assert !(p instanceof User) 
```

### Call operator

The call operator `()` is used to call a method named `call` implicitly. For any object which
defines a `call` method, you can omit the `.call` part and use the call operator instead:

```groovy
class MyCallable {
    //`MyCallable` defines a method named `call`. Note that it doesn’t need to implement `java.util.concurrent.Callable`
    int call(int x) {           
        2*x
    }
}

def mc = new MyCallable()
// we can call the method using the classic method call syntax
assert mc.call(2) == 4          
// or we can omit `.call` thanks to the call operator
assert mc(2) == 4  
```

### Operator overloading

Groovy allows you to overload the various operators so that they can be used with your own classes. Consider this simple class:

```groovy
class Bucket {
    int size

    Bucket(int size) { this.size = size }
    
    // `Bucket` implements a special method called `plus()`
    Bucket plus(Bucket other) {                     
        return new Bucket(this.size + other.size)
    }
}
```

Just by implementing the `plus()` method, the `Bucket` class can now be used with the + operator
like so:

```groovy
def b1 = new Bucket(4)
def b2 = new Bucket(11)
// The two `Bucket` objects can be added together with the `+` operator
assert (b1 + b2).size == 15 
```

All (non-comparator) Groovy operators have a corresponding method that you can implement in your own
classes. The only requirements are that your method is public, has the correct name, and has the
correct number of arguments. The argument types depend on what types you want to support on the
right hand side of the operator. For example, you could support the statement

```groovy
assert (b1 + 11).size == 15
```
by implementing the `plus()` method with this signature:

```groovy
Bucket plus(int capacity) {
    return new Bucket(this.size + capacity)
}
```

Here is a complete list of the operators and their corresponding methods:

| Operator   |  Method  |  Operator  |  Method  |
|:---|:---|:---|:---|
|  `+`  |  `a.plus(b)`  |  `a[b]`  |  `a.getAt(b)`  |
|  `-`  |  `a.minus(b)`  |  `a[b] = c`  |  `a.putAt(b, c)`  |
|  `*`  |  `a.multiply(b)`  |  `a in b`  |  `b.isCase(a)`  |
|  `/`  |  `a.div(b)`  |  `<<`  |  `a.leftShift(b)`  |
|  `%`  |  `a.mod(b)`  |  `>>`  |  `a.rightShift(b)`  |
|  `**`  |  `a.power(b)`  |  `>>>`  |  `a.rightShiftUnsigned(b)`  |
|  `|`  |  `a.or(b)`  |  `++`  |  `a.next()`  |
|  `&`  |  `a.and(b)`  |  `--`  |  `a.previous()`  |
|  `^`  |  `a.xor(b)`  |  `+a`  |  `a.positive()`  |
|  `as`  |  `a.asType(b)`  |  `-a`  |  `a.negative()`  |
|  `a()`  |  `a.call()`  |  `~a`  |  `a.bitwiseNegate()`  |


## 文档

* [官方文档](http://groovy-lang.org/documentation.html)
  * [System requirements](http://groovy-lang.org/download.html#requirements)
  * [IDE integration](http://docs.groovy-lang.org/latest/html/documentation/tools-ide.html)

  * [Differences with Java -【重要】](http://groovy-lang.org/differences.html)

  * Language Specification
    * Syntax
      * [Shebang line](http://groovy-lang.org/syntax.html#_shebang_line) - Shell脚本
      * [Quoted identifiers](http://groovy-lang.org/syntax.html#_quoted_identifiers) -
        引号包裹的标识符，引号中可包含Java非法标识符
      * [Strings](http://groovy-lang.org/syntax.html#all-strings)
        * `Single-quoted string` - 等效于`java.lang.String`，不支持插值操作。
        * `Triple-single-quoted string` - 和`Single-quoted string`一样，且支持多行字符窜。怎么处理开头结尾换行及缩进参考文档。
        * `Escaping special characters` - `\`
        * `Unicode escape sequence`
        * `Double-quoted string` - Double-quoted strings are plain `java.lang.String` if there’s no
          interpolated expression, but are `groovy.lang.GString` instances if interpolation is
          present. 支持插值操作，详情请参考文档。
        * `Triple-double-quoted string` - 和`Double-quoted string`一样，且支持多行字符窜。
        * `Slashy string` - 简化定义正则表达式
        * `Dollar slashy string`
        * [String summary table](http://groovy-lang.org/syntax.html#_string_summary_table)
      * [Characters](http://groovy-lang.org/syntax.html#_characters)
      * [Numbers](http://groovy-lang.org/syntax.html#_numbers)
      * [Lists](http://groovy-lang.org/syntax.html#_lists)
      * [Arrays](http://groovy-lang.org/syntax.html#_arrays)
      * [Maps](http://groovy-lang.org/syntax.html#_maps)

    * [Operators](http://groovy-lang.org/operators.html)
      * `Elvis operator`
      * `Safe navigation operator`
      * `Direct field access operator` - use of `.@` forces usage of the field instead of the
        getter: `user.@name == 'Bob'`
      * `Method pointer operator`
      * `Regular expression operators`
      * `Spread operator`
      * `Range operator`
      * `Spaceship operator` - The spaceship operator (`<=>`) delegates to the `compareTo` method
      * `Subscript operator`
      * `Membership operator` - he membership operator (`in`) is equivalent to calling the `isCase`
        method. In the context of a `List`, it is equivalent to calling `contains`.
      * `Coercion operator`
      * `Call operator`
      * [`Operator precedence`](http://groovy-lang.org/operators.html#_operator_precedence)
      * `Operator overloading`

    * [Program structure](http://groovy-lang.org/structure.html) - 讲解程序整体结构
      * Scripts versus classes - 简易说明Groovy怎么转变Script为Class

    * Object orientation - 讲解Groovy面向对象编程的语法及概念
      * [Primitive types](http://groovy-lang.org/objectorientation.html#_primitive_types) - 自动拆箱/装箱
      * [Meta-annotations](http://groovy-lang.org/objectorientation.html#_meta_annotations) -
        给多个注解起别名，便于开发及维护：`@AnnotationCollector`
      * [Traits](http://groovy-lang.org/objectorientation.html#_traits) - 有点类似JDK8接口的默认实现功能

    * [Closures](http://groovy-lang.org/closures.html) - Groovy闭包

    * [Semantics](http://groovy-lang.org/semantics.html) - Groovy语法
      * `Statements`
      * `GPath expressions`
      * `Promotion and coercion`
      * `Optionality`
      * `The Groovy Truth`
      * `Typing`
      * `Type checking extensions`


  * Tools
    * [groovyc - the Groovy compiler](http://groovy-lang.org/groovyc.html)
      * `groovyc`
      * `Ant task`
      * `Gant`
      * `Maven integration`
      * `GMaven` - 不在支持，转用`GMavenPlus`
      * `GMaven 2` -
        [Groovy Maven Plugin](http://groovy.github.io/gmaven/groovy-maven-plugin/index.html) -
        不是`GMaven`升级版，不是为了替换`GMaven`，它删除了`GMaven`非脚本功能。
        * `groovy:console` - Open a Groovy console window.
        * `groovy:execute` - Execute a Groovy script.
        * `groovy:shell` - Run `groovysh` shell.
      * `GMavenPlus`
        * [GMavenPlus - 插件配置详解](http://groovy.github.io/GMavenPlus/plugin-info.html)
        * [About GMavenPlus](https://github.com/groovy/GMavenPlus/wiki/About)
          * Why? - These were some of my goals in creating this project.
          * Why not patch GMaven?
          * What's so different?
          * Why not just use groovyc in the AntRun Plugin?
          * Why not just use the Groovy-Eclipse Compiler Plugin for Maven?
        * [Choosing Your Build Tool](https://github.com/groovy/GMavenPlus/wiki/Choosing-Your-Build-Tool)
          - 怎样选择适合你的构建工具
        * [Examples](https://github.com/groovy/GMavenPlus/wiki/Examples)
        * [Known Issues](https://github.com/groovy/GMavenPlus/wiki/Known-Issues)
        * [Usage](https://github.com/groovy/GMavenPlus/wiki/Usage)
      * `The Groovy Eclipse Maven plugin`
        * [Catalog of Features](https://github.com/groovy/groovy-eclipse/wiki/Catalog-of-Features)
        * [DSL Descriptors](https://github.com/groovy/groovy-eclipse/wiki/DSL-Descriptors)
        * [Groovy Eclipse Architecture](https://github.com/groovy/groovy-eclipse/wiki/Groovy-Eclipse-Architecture)
        * [A Groovier Eclipse experience](https://spring.io/blog/2009/07/30/a-groovier-eclipse-experience)
        * [Groovy Eclipse Maven plugin](https://github.com/groovy/groovy-eclipse/wiki/Groovy-Eclipse-Maven-plugin)
          * [Examples](https://github.com/groovy/groovy-eclipse/tree/master/extras/groovy-eclipse-compiler-tests/src/it)
      * `Joint compilation`
      * `Android support`

    * [groovysh - the Groovy repl-like shell](http://groovy-lang.org/groovysh.html)

    * [groovyConsole - the Groovy Swing console](http://groovy-lang.org/groovyconsole.html)
      * `Groovy Console` 可以查看 `AST`(`Abstract Syntax Tree`) 及 `Class Generation`

    * [IDE integration](http://groovy-lang.org/ides.html)


  * [The Groovy Development Kit](http://groovy-lang.org/groovy-dev-kit.html)
    * [Working with IO](http://groovy-lang.org/groovy-dev-kit.html#_working_with_io) - 各种文件操作
    * [Lists](http://groovy-lang.org/groovy-dev-kit.html#Collections-Lists) - 操作List
    * [Maps](http://groovy-lang.org/groovy-dev-kit.html#Collections-Maps) - 操作Map
    * [Ranges](http://groovy-lang.org/groovy-dev-kit.html#Collections-Ranges) - 快速创建范围List
    * [Syntax enhancements for collections](http://groovy-lang.org/groovy-dev-kit.html#_syntax_enhancements_for_collections)
      * GPath support
      * Spread operator
      * The star-dot `*.` operator
      * Slicing with the subscript operator
    * [Working with legacy Date/Calendar types](http://groovy-lang.org/groovy-dev-kit.html#_working_with_legacy_date_calendar_types)
    * [Working with Date/Time types](http://groovy-lang.org/groovy-dev-kit.html#_working_with_date_time_types)
    * [Handy utilities](http://groovy-lang.org/groovy-dev-kit.html#_handy_utilities)
      * ConfigSlurper - 读配置文件的工具类
      * Expando - 动态扩展对象
      * Observable list, map and set - 观察list、map和set集合的变动

  * Runtime and compile-time metaprogramming -【重要】
    * [Runtime metaprogramming](http://groovy-lang.org/metaprogramming.html#_runtime_metaprogramming)
      -【重要】运行时元编程
    * [Compile-time metaprogramming](http://groovy-lang.org/metaprogramming.html#_compile_time_metaprogramming)
      -【重要】编译时元编程，通常使用注解在编译时修改AST
    * [Groovy interception mechanism](http://groovy-lang.org/metaprogramming.html#_runtime_metaprogramming)
      -【重要】POGO类方法调用过程
    * [Compilation phases guide](http://groovy-lang.org/metaprogramming.html#_compilation_phases_guide)
    * [Developing AST transformations](http://groovy-lang.org/metaprogramming.html#developing-ast-xforms)
      -【重要】自定义开发AST转换规则
    * [Unleashing the power of AST transformations](http://melix.github.io/ast-workshop/) - 非官方文档

  * [The Grape dependency manager](http://groovy-lang.org/grape.html) -【重要】

    通过`@Grab`注解或`Grape.grab()`下载所依赖的Jar：
    ```groovy
    @Grab(group='org.springframework', module='spring-orm', version='3.2.5.RELEASE')

    Grape.grab(group:'com.jidesoft', module:'jide-oss', version:'[2.2.0,)')
    ```

  * [Testing guide - 编写单元测试](http://groovy-lang.org/testing.html)
    * `assert` - 扩展了Java的assert功能，且默认启用
    * `Mocking` and `Stubbing` - Groovy内置功能
    * Testing with `JUnit`
    * Testing with `Spock`
    * Functional Tests with `Geb`

  * [Domain-Specific Languages](http://groovy-lang.org/dsls.html) -【重要】
    * `Command chains`
    * `Operator overloading`
    * `Script base classes`
    * `Adding properties to numbers`
    * `@DelegatesTo`
    * `Compilation customizers`
    * `Builders` - `MarkupBuilder`、`StreamingMarkupBuilder`、`SaxBuilder`、`StaxBuilder`、
      `DOMBuilder`、`NodeBuilder`、`JsonBuilder`、`StreamingJsonBuilder`、`SwingBuilder`、
      `AntBuilder`、`CliBuilder`、`ObjectGraphBuilder`、`JmxBuilder`、`FileTreeBuilder`

  * [Type checking extensions](http://docs.groovy-lang.org/latest/html/documentation/type-checking-extensions.html) -【重要】

    The DSL relies on a support class called `org.codehaus.groovy.transform.stc.GroovyTypeCheckingExtensionSupport`

  * [Integrating Groovy into applications](http://groovy-lang.org/integrating.html) -【重要】
    * `Eval`
    * `GroovyShell`
    * `GroovyClassLoader`
    * `GroovyScriptEngine`
    * `JSR 223 javax.script API`

  * [Design patterns in Groovy](http://groovy-lang.org/design-patterns.html)

  * [Style guide](http://groovy-lang.org/style-guide.html) -【重要】列出了部分Groovy语言特色，和Java形成对比
    * `No semicolons`
    * `Return keyword optional`
    * `Def and type`
    * `Public by default`
    * `Omitting parentheses`
    * `Classes as first-class citizens`
    * `Getters and Setters`
    * `Initializing beans with named parameters and the default constructor`
    * Using `with()` and `tap()` for repeated operations on the same bean
    * `Equals` and `==`
    * `GStrings` (interpolation, multiline)
    * `Native syntax for data structures`
    * `The Groovy Development Kit`
    * The power of `switch`
    * `Import aliasing`
    * `Groovy Truth`
    * `Safe graph navigation`
    * `Assert`
    * `Elvis operator` for default values
    * `Catch any exception`
    * Optional typing advice

  * Groovy module guides
    * [Parsing and producing JSON](http://groovy-lang.org/json.html)
    * [Working with a relational database](http://groovy-lang.org/databases.html)
    * [Processing XML](http://groovy-lang.org/processing-xml.html)
    * [Scripting Ant tasks](http://groovy-lang.org/scripting-ant.html)
    * [Template engines](http://groovy-lang.org/templating.html) -【重要】
    * [Creating Swing UIs](http://groovy-lang.org/swing.html)
    * [Servlet support](http://groovy-lang.org/servlet.html)
    * [Working with JMX](http://groovy-lang.org/jmx.html) -【重要】

  * API documentation
    * [GroovyDoc documentation of the Groovy APIs](http://groovy-lang.org/api.html)
    * [The Groovy Development Kit enhancements](http://groovy-lang.org/gdk.html) -【重要】可查看哪些Java类被增强了
