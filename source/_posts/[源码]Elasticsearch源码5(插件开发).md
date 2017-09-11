---
layout: post
title: "[源码]Elasticsearch源码5(插件开发)"
date: 2017-08-12 20:15:06 
categories: 
    - 源码
tags:
    - es
---

Elasticsearch源码5(插件开发)

<!--more-->

## 一、前言

elasticsearch之所以功能比较强大，更多的是因为其插件机制比较灵活，可以直接不需要改动源码的情况下，被es的节点扫描加载。本篇文章就简单的讲一下如何进行调试插件，其实无论是river、analysis或者是其他的plugin，都是差不多的写法，所以我们用其中一个river的插件来演示下如何进行调试。
首先，在github上git clone对应的elasticsearch的源码，然后在intellij中将其import进来作为project。

Module类其实就是定义了依赖注入规则，如果不清楚，可以去查看Google Guice的文档，基本上是一致的。如上例中的HelloModule

## 二、开发

### 2.1 插件代码结构

- 插件源码包 

![图 plugin6](https://psiitoy.github.io/img/blog/essourcecode/es-plugin6.jpg)

- 插件发行包

![图 plugin7](https://psiitoy.github.io/img/blog/essourcecode/es-plugin7.jpg)

- es源码插件加载位置

![图 plugin8](https://psiitoy.github.io/img/blog/essourcecode/es-plugin8.jpg)


### 2.2 es提供的插件相关功能类

* PluginManager类，插件管理类，负责插件的安装，卸载，下载等工作。
* PluginsHelper类，插件帮助类，负责列出环境下面所有的site插件。
* PluginsService类，插件服务类，负责插件的加载，实例化和维护插件信息。

- 初始化Node的时候，首先会实例化 PluginsService 类，然后注入PluginsModule模块(将我们开发的插件加入容器)。

> `tmpEnv.pluginsFile()`获取的地址就是`ES_HOME/plugins/`

![图 plugin3](https://psiitoy.github.io/img/blog/essourcecode/es-plugin3.jpg)

![图 plugin4](https://psiitoy.github.io/img/blog/essourcecode/es-plugin4.jpg)

![图 plugin5](https://psiitoy.github.io/img/blog/essourcecode/es-plugin5.jpg)

> 补充：附上带中文注释的`PluginsService`构造过程
```java
public class PluginsService extends AbstractComponent {
   public PluginsService(Settings settings, Environment environment) {
        super(settings);
        this.environment = environment;

        Map<String, Plugin> plugins = Maps.newHashMap();

        //首先，我们从配置文件加载，默认的插件类
        String[] defaultPluginsClasses = settings.getAsArray("plugin.types");
        for (String pluginClass : defaultPluginsClasses) {
            Plugin plugin = loadPlugin(pluginClass, settings);
            plugins.put(plugin.name(), plugin);
        }

        // 现在, 我们查找,所有的在ClassPath下面的插件
        loadPluginsIntoClassLoader();
        plugins.putAll(loadPluginsFromClasspath(settings)); //加载JVM插件
        Set<String> sitePlugins = PluginsHelper.sitePlugins(this.environment); //加载站点插件
        
        //强制依赖的插件，如果没有找到
        String[] mandatoryPlugins = settings.getAsArray("plugin.mandatory", null);
        if (mandatoryPlugins != null) {
            Set<String> missingPlugins = Sets.newHashSet();
            for (String mandatoryPlugin : mandatoryPlugins) {
                if (!plugins.containsKey(mandatoryPlugin) && !sitePlugins.contains(mandatoryPlugin) && !missingPlugins.contains(mandatoryPlugin)) {
                    missingPlugins.add(mandatoryPlugin);
                }
            }
            if (!missingPlugins.isEmpty()) {
            	//抛出异常，整个节点启动失败！
                throw new ElasticSearchException("Missing mandatory plugins [" + Strings.collectionToDelimitedString(missingPlugins, ", ") + "]");
            }
        }
        logger.info("loaded {}, sites {}", plugins.keySet(), sitePlugins);
        this.plugins = ImmutableMap.copyOf(plugins);

        //现在，所有插件都加载好了,处理插件实现类的 onModule 方法的引用 ,这里有 依赖注入的秘密。
        
        MapBuilder<Plugin, List<OnModuleReference>> onModuleReferences = MapBuilder.newMapBuilder();
        for (Plugin plugin : plugins.values()) {
            List<OnModuleReference> list = Lists.newArrayList();
            //....
        }
        this.onModuleReferences = onModuleReferences.immutableMap();
        this.refreshInterval = componentSettings.getAsTime("info_refresh_interval", TimeValue.timeValueSeconds(10));

    }
}

```

### 2.3 我们开发的插件源码分析

> 首先在例子中我们开发的插件名称叫`ActionMonitor`，功能是做TP监控统计。（本文不讨论监控的实现），那么通过`ActionFilter`实现。
注意:ActionMonitor是我们自己定义的类。

- 后面我们讨论`ActionFilter`是做什么的，我们首先从`ActionMonitorPlugin`看起
 
```java
public class ActionMonitorPlugin extends Plugin {
    private final ESLogger log = Loggers.getLogger(ActionMonitorPlugin.class);


    public ActionMonitorPlugin() {
        log.info("Starting Action Monitor Plugin");
    }

    public void onModule(ActionModule module) { //code 1 插件初始化
        module.registerFilter(ActionMonitorFilter.class);
    }

    @Override
    public Collection<Module> nodeModules() {  //code 2  插件模块注入
        return Collections.<Module>singletonList(new ActionMonitorModule());
    }

    public String description() {
        return "description";
    }

    public String name() {
        return "ActionMonitor";
    }
}

```

- `code1` 插件初始化的源码分析(通过`PluginsService`进行初始化)

![图 plugin1](https://psiitoy.github.io/img/blog/essourcecode/es-plugin1.jpg)

![图 plugin2](https://psiitoy.github.io/img/blog/essourcecode/es-plugin2.jpg)

- `code2` 就是负责插件注入容器 

### 2.3 如何扩展现有模块的？

> 上一节我们分析了插件如何初始化和如何注入容器。

- 可扩展的模块，一般都提供了 addXXX，registerXXX 等方法

```java
    //智能提示
    public void onModule(SuggestModule suggestModule) {
        suggestModule.registerSuggester(MySuggester.class);
    }
    //REST
    public void onModule(RestModule restModule) {
        restModule.addRestAction(MyRestAction.class);
    }
    //高亮
    public void onModule(HighlightModule highlightModule) {
        highlightModule.registerHighlighter(MyHighlighter.class);
    }

```

- 可替换的模块，一般是实现了SpawnModules接口的模块，比如DiscoveryModule

```java
    @Override
    public Iterable<? extends Module> spawnModules() {
        Class<? extends Module> defaultDiscoveryModule;
        if (settings.getAsBoolean("node.local", false)) {
            defaultDiscoveryModule = LocalDiscoveryModule.class;
        } else {
            defaultDiscoveryModule = ZenDiscoveryModule.class;
        }
        return ImmutableList.of(Modules.createModule(settings.getAsClass("discovery.type", defaultDiscoveryModule, "org.elasticsearch.discovery.", "DiscoveryModule"), settings));
    }

```

- 根据配置项discovery.type来确定加载那个模块，

- 不可以扩展或替换的组件，比如  Internal 开头的组件，InternalClusterService，InternalIndicesService    等是不可以替换的。

- 只要把Filter注册到ActionModule，ActionModule便会将在Node启动时加载到容器中，我们再回忆下Node加载的源码 

![图 plugin4](https://psiitoy.github.io/img/blog/essourcecode/es-plugin4.jpg)

- ActionModule包含actionFilters中，就是包含我们此次开发的Filter集合

```java
public class ActionModule extends AbstractModule {

    private final Map<String, ActionEntry> actions = Maps.newHashMap();
    private final List<Class<? extends ActionFilter>> actionFilters = new ArrayList<>();
}

```

- 而后我们看下 ActionFilter拦截生效的对象 TransportAction。

```java
public abstract class TransportAction<Request extends ActionRequest, Response extends ActionResponse> extends AbstractComponent {
    
    //属性略
    
    protected TransportAction(Settings settings, String actionName, ThreadPool threadPool, ActionFilters actionFilters,
                              IndexNameExpressionResolver indexNameExpressionResolver) {
        super(settings);
        this.threadPool = threadPool;
        this.actionName = actionName;
        this.filters = actionFilters.filters();
        this.parseFieldMatcher = new ParseFieldMatcher(settings);
        this.indexNameExpressionResolver = indexNameExpressionResolver;
    }
    
    //其他方法略
 
    public final void execute(Request request, ActionListener<Response> listener) {
        ActionRequestValidationException validationException = request.validate();
        if (validationException != null) {
            listener.onFailure(validationException);
            return;
        }

        if (filters.length == 0) {
            try {
                doExecute(request, listener);
            } catch(Throwable t) {
                logger.trace("Error during transport action execution.", t);
                listener.onFailure(t);
            }
        } else {
            RequestFilterChain requestFilterChain = new RequestFilterChain<>(this, logger);//code3
            requestFilterChain.proceed(actionName, request, listener);
        }
    }

    protected abstract void doExecute(Request request, ActionListener<Response> listener);
     
    private static class RequestFilterChain<Request extends ActionRequest, Response extends ActionResponse> implements ActionFilterChain {

        private final TransportAction<Request, Response> action;
        private final AtomicInteger index = new AtomicInteger();
        private final ESLogger logger;

        private RequestFilterChain(TransportAction<Request, Response> action, ESLogger logger) {
            this.action = action;
            this.logger = logger;
        }

        @Override @SuppressWarnings("unchecked")
        public void proceed(String actionName, ActionRequest request, ActionListener listener) {
            int i = index.getAndIncrement();
            try {
                if (i < this.action.filters.length) {
                    this.action.filters[i].apply(actionName, request, listener, this); //code4
                } else if (i == this.action.filters.length) {
                    this.action.doExecute((Request) request, new FilteredActionListener<Response>(actionName, listener, new ResponseFilterChain(this.action.filters, logger))); //code5
                } else {
                    listener.onFailure(new IllegalStateException("proceed was called too many times"));//code6
                }
            } catch(Throwable t) {
                logger.trace("Error during transport action execution.", t);
                listener.onFailure(t);
            }
        }

        @Override
        public void proceed(String action, ActionResponse response, ActionListener listener) {
            assert false : "request filter chain should never be called on the response side";
        }
    }

    private static class ResponseFilterChain implements ActionFilterChain {

        private final ActionFilter[] filters;
        private final AtomicInteger index;
        private final ESLogger logger;

        private ResponseFilterChain(ActionFilter[] filters, ESLogger logger) {
            this.filters = filters;
            this.index = new AtomicInteger(filters.length);
            this.logger = logger;
        }

        @Override
        public void proceed(String action, ActionRequest request, ActionListener listener) {
            assert false : "response filter chain should never be called on the request side";
        }

        @Override @SuppressWarnings("unchecked")
        public void proceed(String action, ActionResponse response, ActionListener listener) {
            int i = index.decrementAndGet();
            try {
                if (i >= 0) {
                    filters[i].apply(action, response, listener, this);
                } else if (i == -1) {
                    listener.onResponse(response);
                } else {
                    listener.onFailure(new IllegalStateException("proceed was called too many times"));
                }
            } catch (Throwable t) {
                logger.trace("Error during transport action execution.", t);
                listener.onFailure(t);
            }
        }
    }

    private static class FilteredActionListener<Response extends ActionResponse> implements ActionListener<Response> {

        private final String actionName;
        private final ActionListener listener;
        private final ResponseFilterChain chain;

        private FilteredActionListener(String actionName, ActionListener listener, ResponseFilterChain chain) {
            this.actionName = actionName;
            this.listener = listener;
            this.chain = chain;
        }

        @Override
        public void onResponse(Response response) {
            chain.proceed(actionName, response, listener);
        }

        @Override
        public void onFailure(Throwable e) {
            listener.onFailure(e);
        }
    }    
}

```

- code3 对所有request进行filter。(通过对加载的filter开启内部计数器的方式)

- code4 执行真正的request方法。

- code5 这里个人感觉就是一个异常逻辑，通常应该不会执行到。