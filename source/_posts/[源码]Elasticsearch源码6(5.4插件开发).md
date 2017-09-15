---
layout: post
title: "[源码]Elasticsearch源码6(5.4插件开发)"
date: 2017-08-17 22:12:06 
categories: 
    - 源码
tags:
    - es
---

本文重点讨论基于`5.4.3`版本的ES在`gradle`构建项目的环境下如何做插件开发。

<!--more-->

## 一、前言

### 1.1 5.4 和 2.x 源码开发的差别

* 构建方式从maven变成了gradle这个是最大的差别，而对于ES核心包的插件模块来说只是做了优化，对于开发者来说掌握了2.x的开发就可以迅速上手5.4.x的开发。

### 1.2 插件调试与DEBUG

* 没有办法直接把编译后的发行版目录位置做软链挂载到配置文件了，只能自己编译后解压到`{ES_HOME}/plugins`文件夹下了。

> 原因如下
   
* `2.X`版本的ES插件加载目录获取
   
```java
public class Environment{
    public Environment(Settings settings) {
        //...其他略
        if (settings.get("path.plugins") != null) {
                  pluginsFile = PathUtils.get(cleanPath(settings.get("path.plugins")));//code1
        } else {
                  pluginsFile = homeFile.resolve("plugins");
        }
    }
}
              
```

* `5.4.3`版本的ES插件加载目录获取

```java
public class Environment{
    public Environment(Settings settings) {
        //...其他略
        pluginsFile = homeFile.resolve("plugins");    
    }
}
              
```

- `code1` 生生被干掉了。。。so 只能老老实实 `assembly` then `copy`了。

## 二、5.4.3插件开发

### 2.1 启动项目

> 第一步当然是要把5.4.3的源码项目启动起来。下面我们逐步操作。

1) 下载项目
```bash
$ git clone https://github.com/elastic/elasticsearch.git

```

2) 项目根目录执行`gradle idea`，使项目按照idea的方式构建

3) idea打开项目并且`import gradle project`导入项目。

4) 创建`${ES_HOME}/config`，把`distribution/src/main/resources/config` 的文件都 `copy` 过来(不用拷jvm.options)

* 我们的${ES_HOME}就是`$USER_HOME/elasticsearch-v5.4.3/core`

* jvm.options 是通过启动脚本生效的，我们不用纠结这个文件。

5) 启动发生`access denied`异常

```
2017-03-08 11:06:11,447 main ERROR Could not register mbeans java.security.AccessControlException: access denied ("javax.management.MBeanTrustPermission" "register")
     at java.security.AccessControlContext.checkPermission(AccessControlContext.java:472)
     at java.lang.SecurityManager.checkPermission(SecurityManager.java:585)
     ...
     
```

- 解决方案：添加 -Djava.security.policy=/home/wangdi/IdeaWorkspace/sourcecode/elasticsearch-v5.4.3/core/config/elasticsearch.policy

> 内容如下

```
grant {
    permission javax.management.MBeanTrustPermission "register";
    permission javax.management.MBeanServerPermission "createMBeanServer";
};

```

6) 如果报`Exception: java.security.AccessControlException thrown from the UncaughtExceptionHandler in thread`之类的其他权限异常。

- 解决方案：直接把`elasticsearch.policy`改为。(暴力点)

```
grant {
    permission java.security.AllPermission;
};

```

7) 创建`${ES_HOME}/plugins`文件夹

> 和2.x开发是一样的，先空着。

8) `Unsupported transport.type`异常

```
...
[2017-09-14T11:10:01,550][WARN ][o.e.b.ElasticsearchUncaughtExceptionHandler] [] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: java.lang.IllegalStateException: Unsupported transport.type []
...

```

- 解决方法:
  + 根目录`modules`模块下找到`transport-netty3`或者`transport-netty4`模块，构建`netty`子模块。
  + 然后`modules/transport-netty3/build/distributions`下面找到发行版的zip包。
  + 最后再核心包下的`modules`目录下建个文件夹叫`transport-netty3`或`transport-netty4`(模块名)，把对应的zip包解压到这里。
  
9) jvm参数校验异常

```
...
ERROR: [1] bootstrap checks failed
[1]: initial heap size [130023424] not equal to maximum heap size [2071986176]; this can cause resize pauses and prevents mlockall from locking the entire heap
...

```

- 解决方法: 添加启动参数 `-Xms1g -Xmx1g`。

> 再提一下 jvm.options 是通过启动脚本生效的，我们不用纠结这个文件。自己在idea配置jvm启动参数，启动参数如下。

```
-Xms1g -Xmx1g -ea -Delasticsearch -Des.foreground=yes -Des.path.home=/home/wangdi/IdeaWorkspace/sourcecode/elasticsearch-v5.4.3/core -Djava.security.policy=/home/wangdi/IdeaWorkspace/sourcecode/elasticsearch-v5.4.3/core/config/elasticsearch.policy

```

> elasticsearch.yml如下。详情参见:[elasticsearch.yml的配置属性官方解释](https://www.ibm.com/support/knowledgecenter/zh/SSFPJS_8.5.6/com.ibm.wbpm.main.doc/topics/rfps_esearch_configoptions.html)

```
path.logs: /usr/share/elasticsearch5/logs
path.data: /usr/share/elasticsearch5/data

http.cors.enabled: true
http.cors.allow-origin: "*"

network.host: 0.0.0.0

```

10) 项目启动成功，开始开发调试

### 2.2 插件开发项目结构

> 核心包模块结构如下

![图 es5plugin-core](https://psiitoy.github.io/img/blog/essourcecode/es5plugin-core.png)

> 插件模块结构如下

![图 es5plugin-plugin](https://psiitoy.github.io/img/blog/essourcecode/es5plugin-plugin.png)

### 2.3 5.4.3插件开发源码分析

> 我们简单看下`ActionLoggingPlugin`这个 `ES` 源码测试用例中提供的插件。

```java
public static class ActionLoggingPlugin extends Plugin implements ActionPlugin {

        @Override
        public Collection<Module> createGuiceModules() {
            return Collections.<Module>singletonList(new ActionLoggingModule());
        }

        @Override
        public List<Class<? extends ActionFilter>> getActionFilters() {
            return singletonList(LoggingFilter.class);
        }
    }

    public static class ActionLoggingModule extends AbstractModule {
        @Override
        protected void configure() {
            bind(LoggingFilter.class).asEagerSingleton();
        }

    }

    public static class LoggingFilter extends ActionFilter.Simple {

        private final ThreadPool threadPool;

        @Inject
        public LoggingFilter(Settings settings, ThreadPool pool) {
            super(settings);
            this.threadPool = pool;
        }

        @Override
        public int order() {
            return 999;
        }

        @Override
        protected boolean apply(String action, ActionRequest request, ActionListener<?> listener) {
            requests.add(new RequestAndHeaders(threadPool.getThreadContext().getHeaders(), request));
            return true;
        }
    }

```

> 结论就是，ES的 `>>5.4.x` 版本对插件做了优化重构，但是对于开发者来说和`2.x`插件开发几乎无异。