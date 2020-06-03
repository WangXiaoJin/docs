# OSGi - Jigsaw

## OSGi

There are a number of commercial and open source OSGi framework implementations. The open source options include `Apache
Felix`, `Eclipse Concierge`, `Eclipse Equinox`, `Knopflerfish`.

### 文档

* [enRoute Tutorials](https://enroute.osgi.org/) - enRoute Tutorials and Examples provide a simple developer onramp for
  those unfamiliar with OSGi™.

* [官网](https://www.osgi.org/developer/where-to-start/)

* [OSGi模块化框架详解](http://www.uml.org.cn/zjjs/201905273.asp)

### 插件

#### bnd-maven-plugin

The bnd-maven-plugin is a key plugin when producing OSGi bundles. The bnd-maven-plugin is responsible for parsing the bytecode
of the classes included in the JAR file being produced by a Maven module. Based on the discovered annotations and dependencies
the bnd-maven-plugin will construct an OSGi Manifest file for the bundle, and any other required metadata
(for example Declarative Services XML descriptors).

#### bnd-export-maven-plugin

The bnd-export-maven-plugin is used to export an OSGi application as a runnable JAR. The input to the bnd-export-maven-plugin
is a bndrun file. This file declares a set of bundles and launch properties that should be used to start an OSGi framework
 containing the application.

#### bnd-indexer-maven-plugin

The bnd-indexer-maven-plugin is used to generate an OSGi repository index from the set of maven dependencies in your module’s pom.
This repository index can be used for resolving or exporting the application.

#### bnd-resolver-maven-plugin

The bnd-resolver-maven-plugin is not normally part of the main build, but it can be used from the command line to resolve
an application or integration testing bndrun. This resolve operation takes a set of run requirements and uses an OSGi repository
 index to find the complete set of bundles that need to b deployed to satisfy the run requirements.

#### bnd-baseline-maven-plugin

The bnd-baseline-maven-plugin is used to validate the semantic versioning of a bundle’s exported API by comparing it against
the last released version. This plugin will fail the build if the API version has not been increased when a change has been made,
or if the version increase is insufficient to communicate the semantics of the change.

#### bnd-testing-maven-plugin

The bnd-testing-maven-plugin is used to provide bundle-level testing. The tests are written using JUnit and packaged into
a tester bundle (often produced in the same build project). The bnd-testing-maven-plugin then uses one or more bndrun files
to launch an OSGi framework containing the tester bundle and the bundles under test. The test cases are then run by the
bnd-testing-maven-plugin.



## Karaf


## Apache Felix


## Eclipse Equinox



## Jigsaw

### 文档

* [Jigsaw - 官网](http://openjdk.java.net/projects/jigsaw/)
* [java9 模块化 jigsaw](https://www.cnblogs.com/krcys/p/8657282.html)




## 杂项

* [Java 9，OSGi和模块化的未来（1）](https://mindawei.github.io/2018/02/05/Java-9%EF%BC%8COSGi%E5%92%8C%E6%A8%A1%E5%9D%97%E5%8C%96%E7%9A%84%E6%9C%AA%E6%9D%A5%EF%BC%881%EF%BC%89/)
* [Java 9，OSGi和模块化的未来（2）](https://mindawei.github.io/2018/02/06/Java-9%EF%BC%8COSGi%E5%92%8C%E6%A8%A1%E5%9D%97%E5%8C%96%E7%9A%84%E6%9C%AA%E6%9D%A5%EF%BC%882%EF%BC%89/)
















