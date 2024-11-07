---
title: 使用Flink官方提供的骨架不能被Cursor正常识别
published: 2024-11-08
description: ''
image: ''
tags: ['Flink', 'Cursor', 'Maven', 'Java','Vscode']
category: '开发'
draft: false 
lang: ''
---
# 使用Flink官方提供的骨架不能被Cursor正常识别

在使用Flink官方提供的骨架时，可能会遇到Cursor无法正常识别的问题。经过排查，发现问题源自于`pom.xml`文件中的特定插件配置。通过移除这些插件部分，问题得以解决。

## 问题描述

在项目的`pom.xml`文件中，存在以下插件配置：

```xml:pom.xml
<plugin>
    <groupId>org.eclipse.m2e</groupId>
    <artifactId>lifecycle-mapping</artifactId>
    <version>1.0.0</version>
    <configuration>
        <lifecycleMappingMetadata>
            <pluginExecutions>
                <pluginExecution>
                    <pluginExecutionFilter>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-shade-plugin</artifactId>
                        <versionRange>[3.1.1,)</versionRange>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </pluginExecutionFilter>
                    <action>
                        <ignore />
                    </action>
                </pluginExecution>
                <pluginExecution>
                    <pluginExecutionFilter>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-compiler-plugin</artifactId>
                        <versionRange>[3.1,)</versionRange>
                        <goals>
                            <goal>testCompile</goal>
                            <goal>compile</goal>
                        </goals>
                    </pluginExecutionFilter>
                    <action>
                        <ignore />
                    </action>
                </pluginExecution>
            </pluginExecutions>
        </lifecycleMappingMetadata>
    </configuration>
</plugin>
```

上述配置主要用于忽略`maven-shade-plugin`和`maven-compiler-plugin`的某些执行目标。然而，这些配置可能会干扰Flink骨架的正常运行，导致Cursor无法正确识别。

## 解决方案

移除`pom.xml`文件中上述的`<plugin>`部分后，重新构建项目，可以解决Cursor无法识别的问题。具体步骤如下：

1. 打开项目根目录下的`pom.xml`文件。
2. 找到并删除以下插件配置：

    ```xml:pom.xml
    <plugin>
        <groupId>org.eclipse.m2e</groupId>
        <artifactId>lifecycle-mapping</artifactId>
        <version>1.0.0</version>
        <configuration>
            <lifecycleMappingMetadata>
                <pluginExecutions>
                    <pluginExecution>
                        <pluginExecutionFilter>
                            <groupId>org.apache.maven.plugins</groupId>
                            <artifactId>maven-shade-plugin</artifactId>
                            <versionRange>[3.1.1,)</versionRange>
                            <goals>
                                <goal>shade</goal>
                            </goals>
                        </pluginExecutionFilter>
                        <action>
                            <ignore />
                        </action>
                    </pluginExecution>
                    <pluginExecution>
                        <pluginExecutionFilter>
                            <groupId>org.apache.maven.plugins</groupId>
                            <artifactId>maven-compiler-plugin</artifactId>
                            <versionRange>[3.1,)</versionRange>
                            <goals>
                                <goal>testCompile</goal>
                                <goal>compile</goal>
                            </goals>
                        </pluginExecutionFilter>
                        <action>
                            <ignore />
                        </action>
                    </pluginExecution>
                </pluginExecutions>
            </lifecycleMappingMetadata>
        </configuration>
    </plugin>
    ```

3. 保存并关闭`pom.xml`文件。