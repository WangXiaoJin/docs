# Logback

## 功能

### Implicit actions in practice

The respective Joran configurators of logback-classic and logback-access include just two implicit actions, namely `NestedBasicPropertyIA` and `NestedComplexPropertyIA`.

`NestedBasicPropertyIA` is applicable for any property whose type is a primitive type (or equivalent object type in the java.lang package), 
an enumeration type, or any type adhering to the "valueOf" convention. Such properties are said to be basic or simple. 
A class is said to adhere to the "valueOf" convention if it contains a static method named `valueOf()` taking a java.lang.String as parameter and returning an instance of the type in question. At present, the Level, Duration and FileSize classes follow this convention.

`NestedComplexPropertyIA` action is applicable, in the remaining cases where NestedBasicPropertyIA is not applicable and 
if the object at the top of the object stack has a setter or adder method for a property name equal to the current element name. 
Note that such properties can in turn contain other components. Thus, such properties are said to be complex. 
In presence of a complex property, NestedComplexPropertyIA will instantiate the appropriate class for the nested component and attach it to the parent component (at the top of the object stack) by using the setter/adder method of the parent component and the nested element's name. The corresponding class is specified by the class attribute of the (nested) current element. However, if the class attribute is missing, the class name can be deduced implicitly, if any of the following is true:

1. there is an internal rule associating the parent object's property with a designated class
2. the setter method contains a @DefaultClass attribute designating a given class
3. the parameter type of the setter method is a concrete class possessing a public constructor

#### Default class mapping

In logback-classic, there are a handful of internal rules mapping parent class/property name pairs to a default class. These are listed in the table below.

|Parent class	| property name	| default nested class|
| :-------: | :------: | :------: |
|ch.qos.logback.core.AppenderBase|	encoder|	ch.qos.logback.classic.encoder.PatternLayoutEncoder|
|ch.qos.logback.core.UnsynchronizedAppenderBase|	encoder|	ch.qos.logback.classic.encoder.PatternLayoutEncoder|
|ch.qos.logback.core.AppenderBase|	layout|	ch.qos.logback.classic.PatternLayout|
|ch.qos.logback.core.UnsynchronizedAppenderBase|	layout|	ch.qos.logback.classic.PatternLayout|
|ch.qos.logback.core.filter.EvaluatorFilter|	evaluator|	ch.qos.logback.classic.boolex.JaninoEventEvaluator|

This list may change in future releases. Please see logback-classic 
[JoranConfigurator](https://logback.qos.ch/xref/ch/qos/logback/classic/joran/JoranConfigurator.html)'s `addDefaultNestedComponentRegistryRules` method for the latest rules.

In logback-access, the rules are very similar. In the default class for the nested component, the ch.qos.logback.classic 
package is replaced by ch.qos.logback.access. See logback-access [JoranConfigurator](https://logback.qos.ch/xref/ch/qos/logback/classic/joran/JoranConfigurator.html)'s `addDefaultNestedComponentRegistryRules` method for the latest rules.

#### Collection of properties

Note that in addition to single simple properties or single complex properties, logback's implicit actions support collections of properties, 
be they simple or complex. Instead of a setter method, the property is specified by an "adder" method.

## 官方文档

* [Chapter 2: Architecture](https://logback.qos.ch/manual/architecture.html)
  * Logger, Appenders and Layouts
  * Effective Level aka Level Inheritance
  * Basic Selection Rule
  * Appender Additivity
  * Parameterized logging
  * A peek under the hood - 日志执行过程
* [Chapter 3: Logback configuration](https://logback.qos.ch/manual/configuration.html)
  * 配置文件加载机制
  * [Status Messages](https://logback.qos.ch/manual/configuration.html#automaticStatusPrinting)
  * [Setting the location of the configuration file via a system property](https://logback.qos.ch/manual/configuration.html#configFileProperty)
  * Automatically reloading configuration file upon modification
  * [Enabling packaging data in stack traces](https://logback.qos.ch/manual/configuration.html#packagingData)
  * [Stopping logback-classic](https://logback.qos.ch/manual/configuration.html#stopContext)
  * [Configuration file syntax](https://logback.qos.ch/manual/configuration.html#syntax)
    * Logger
    * Root
    * Appender
    * Context Name
    * Variable
    * Conditional processing
    * File inclusion
    * Context listener
* [Chapter 4: Appenders](https://logback.qos.ch/manual/appenders.html)
* [Chapter 5: Encoders](https://logback.qos.ch/manual/encoders.html)
* [Chapter 6: Layouts](https://logback.qos.ch/manual/layouts.html)
  * [Restrictions on literals immediately following conversion words](https://logback.qos.ch/manual/layouts.html#restrictionsOnLiterals)
* [Chapter 7: Filters](https://logback.qos.ch/manual/filters.html)
* [Chapter 8: Mapped Diagnostic Context](https://logback.qos.ch/manual/mdc.html) - MDC
* [Chapter 9: Logging separation](https://logback.qos.ch/manual/loggingSeparation.html)
* [Chapter 10: JMX Configurator](https://logback.qos.ch/manual/jmxConfig.html)
* [Chapter 11: Joran](https://logback.qos.ch/manual/onJoran.html)
* [Chapter 14: Using SSL](https://logback.qos.ch/manual/usingSSL.html)

* [An introduction to `logback-access` for `Jetty` and `Tomcat`](https://logback.qos.ch/access.html)
  * Logback-access under Tomcat
  * TeeFilter (a servlet-filter)
  * [Capturing incoming HTTP requests and outgoing responses](https://logback.qos.ch/recipes/captureHttp.html)
* [Triggering an email containing the isolated logs of selected transactions](https://logback.qos.ch/recipes/emailPerTransaction.html)

* [Logback error messages and their meanings](https://logback.qos.ch/codes.html)
  * Failed to rename file [x] as [y]
    * On Windows
    * On Unix-*
  * As of logback version 0.9.28, JaninoEventEvaluator expects Java blocks.

## 三方扩展工具

* [logstash-logback-encoder](https://github.com/logfellow/logstash-logback-encoder) - Provides logback encoders, layouts, and appenders to log in JSON and other formats supported by Jackson.
* [Lilith](http://lilith.huxhorn.de/) - Lilith is a Logging- and AccessEvent viewer for logback.
* [Logback-akka](https://github.com/mojolly/logback-akka) - Consists of several akka-based logback utilities, including ActorAppender, HoptoadActorAppender and Logstash redis appender.
* [Logback-android](https://github.com/tony19/logback-android) - Logback-Android brings the power of Logback to Android.
* [Simpledb-appender](http://code.google.com/p/simpledb-appender/) - Logback Appender writing to Amazon SimpleDB. See also [Logging the cloud with SimpleDB](http://www.peecho.com/blog/logging-the-cloud-with-simpledb.html).
* [logback-configuration](https://github.com/carlspring/logback-configuration/) - A service layer (using Spring) and a REST interface which provides methods to: add or update loggers, resolve a log file, resolve the logback configuration file, and upload a logback configuration file and reload it.
* [Logback-gelf](https://github.com/Moocar/logback-gelf) - Logback-gelf can log messages to a [Graylog2](http://graylog2.org/) server via GELF messages.
* [Logback-testng](https://github.com/sbabcoc/logback-testng) - Logback appender for TestNG Reporter.


## logstash-logback-encoder

Provides logback encoders, layouts, and appenders to log in JSON and [other formats supported by Jackson](https://github.com/logfellow/logstash-logback-encoder#data-format).

Supports both regular LoggingEvents (logged through a `Logger`) and AccessEvents (logged via `logback-access`).

Originally written to support output in logstash's JSON format, but has evolved into a highly-configurable, general-purpose, 
structured logging mechanism for JSON and other Jackson dataformats. The structure of the output, and the data it contains, 
is fully configurable.

* [文档 - GitHub](https://github.com/logfellow/logstash-logback-encoder)
* [Details about stack hash](https://github.com/logfellow/logstash-logback-encoder/blob/main/stack-hash.md)

