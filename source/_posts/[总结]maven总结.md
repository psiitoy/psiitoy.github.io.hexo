---
layout: post
title: "[总结]maven总结"
date: 2016-07-25 21:15:06 
categories: 
    - 总结
tags:
    - maven
---

maven总结

<!--more-->

上传源码!

 <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>2.1.2</version>
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>