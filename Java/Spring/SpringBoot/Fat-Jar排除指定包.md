# Fat-Jar排除指定包

## 1. Spring-Boot-Jar-Type

`spring-boot-maven-plugin` 在打包时会获取 jar 包中 `MANIFEST.MF` 的 `Spring-Boot-Jar-Type`属性值，如果值为
`dependencies-starter` 或 `annotation-processor`，则此jar不会被打进 fat-jar 中，但是此jar的依赖包不受任何影响。

可以借助此功能忽略SpringBoot项目中的`starter`及`annotation-processor`包，因为这两种类型的包在应用运行时根本就用不到。

> 建议使用此方案

参考示例：
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jar-plugin</artifactId>
      <configuration>
        <archive>
          <!-- 打包时不包含以下两个文件
              1. META-INF/maven/${groupId}/${artifactId}/pom.xml
              2. META-INF/maven/${groupId}/${artifactId}/pom.properties
           -->
          <addMavenDescriptor>false</addMavenDescriptor>
          <manifest>
            <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
            <addBuildEnvironmentEntries>true</addBuildEnvironmentEntries>
          </manifest>
          <manifestEntries>
            <!--
                在MANIFEST.MF文件中增加 Spring-Boot-Jar-Type: dependencies-starter 属性。spring-boot-maven-plugin 打包时
                会判断jar包的Spring-Boot-Jar-Type属性值，如果值为dependencies-starter或annotation-processor，则丢弃此jar，
                但此jar的依赖包会被打进fat-jar。
            -->
            <Spring-Boot-Jar-Type>dependencies-starter</Spring-Boot-Jar-Type>
          </manifestEntries>
        </archive>
      </configuration>
    </plugin>
  </plugins>
</build>
```

参考链接：
* [spring-boot-maven-plugin 插件源码](https://github.com/spring-projects/spring-boot/blob/c59d474ec4e586a437c50afd6b19ae6973d4db64/spring-boot-project/spring-boot-tools/spring-boot-maven-plugin/src/main/java/org/springframework/boot/maven/JarTypeFilter.java#L49)

## 2. spring-boot-maven-plugin 插件 exclude 功能

### 方法一

排除指定的`groupId`、`artifactId`，如有需要可以配置`classifier`。
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>
                        <groupId>com.example</groupId>
                        <artifactId>module1</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 方法二

排除指定`groupId`下的所有artifact。
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludeGroupIds>com.example</excludeGroupIds>
            </configuration>
        </plugin>
    </plugins>
</build>
```

> 注意事项:
> 1. spring-boot-maven-plugin 插件 exclude 功能只会忽略配置自身jar，不会忽略其依赖jar。
> 2. 在当前可项目的`pom.xml`中依赖的jar使用`<optional>true</optional>`或`<scope>provided</scope>`修饰时，打出的 fat-jar 还是会
包含上述jar的。因为对于spring-boot项目来说，其自生就是一个容器，不存在外部容器的说法。
> 3. 间接依赖的jar配置了`<optional>true</optional>`或`<scope>provided</scope>`则忽略，规则和maven保持一直。


参考链接：
* [Dependency Exclusion](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/#packaging.examples.exclude-dependency)



