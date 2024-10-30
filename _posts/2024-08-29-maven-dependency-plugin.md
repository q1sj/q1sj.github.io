---
layout: post
title: "maven打包时源码和依赖分开打包"
date: 2024-08-29
tags: [java]
---
因为测试环境网络不佳,为了提升发布速度,可以将源码和依赖分开打包,如果依赖没有变化,发布时只需要将源码部分上传,大大减少网络传输时间
## spring boot默认打包

```xml
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
```

## 分开打包

```xml
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <layout>ZIP</layout>
                    <includes>
                        <include>
                            <groupId>nothing</groupId>
                            <artifactId>nothing</artifactId>
                        </include>
                    </includes>
                </configuration>
            </plugin>
            <!--拷贝依赖到jar外面的lib目录-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy-lib</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>target/lib</outputDirectory>
                            <excludeTransitive>false</excludeTransitive>
                            <stripVersion>false</stripVersion>
                            <includeScope>runtime</includeScope>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```
启动时需要增加参数 指定lib目录
```shell
java -Dloader.path=./lib -jar demo.jar
```


可能出现的报错

https://stackoverflow.com/questions/44312643/getting-unable-to-resolve-persistence-unit-error-when-building-jar-without-ide

```text
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'entityManagerFactory' defined in class path resource [org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaAutoConfiguration.class]: Invocation of init method failed; nested exception is javax.persistence.PersistenceException: Unable to resolve persistence unit root URL
        at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1628) ~[spring-beans-4.3.8.RELEASE.jar:4.3.8.RELEASE]
        
```

启动类增加

```java
@EnableAutoConfiguration(exclude= HibernateJpaAutoConfiguration.class)
```

application.yml

```yaml
spring:
  data:
    jpa:
      repositories:
        enabled: false
```

