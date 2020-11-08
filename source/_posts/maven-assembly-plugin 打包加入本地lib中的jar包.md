---
title: maven-assembly-plugin打包加入本地lib中的jar包
date: 2020-08-31 12:42:01
comments: true
categories:
 - 工具
 - 编译工具
 - Maven
tags: 
 - maven
---

# pom.xml
```xml
<plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>Main</mainClass>
                        </manifest>
                        <manifestEntries>
                            <Class-Path>.</Class-Path>
                        </manifestEntries>
                    </archive>
                    	<!-- 注释这个 -->
<!--                    <descriptorRefs>-->
<!--                        <descriptorRef>jar-with-dependencies</descriptorRef>-->
<!--                    </descriptorRefs>-->

                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <!-- 增加配置 -->
                        <configuration>
                            <!-- assembly.xml文件路径 -->
                            <descriptors>
                                <descriptor>${project.basedir}/assembly.xml</descriptor>
                            </descriptors>
                        </configuration>
                    </execution>
                </executions>

            </plugin>
```
# assembly.xml

```xml
<assembly>
    <id>all</id>
    <formats>
        <format>jar</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>
    <dependencySets>
        <!-- 默认的配置 -->
        <dependencySet>
            <outputDirectory>/</outputDirectory>
            <useProjectArtifact>true</useProjectArtifact>
            <unpack>true</unpack>
            <scope>runtime</scope>
        </dependencySet>

        <!-- 增加scope类型为system的配置 -->
        <dependencySet>
            <outputDirectory>/</outputDirectory>
            <useProjectArtifact>true</useProjectArtifact>
            <unpack>true</unpack>
            <scope>system</scope>
        </dependencySet>

    </dependencySets>
</assembly>
```
打包时会插件会从maven仓库去找本地的jar的依赖，会报错请忽略，打开打包好的jar发现它已经被打包了。