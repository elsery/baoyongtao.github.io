---
layout: post
title: "SpringBoot 配置文件 application.properties"
date: 2017-11-03 
description: "SpringBoot,application.properties"
tag: java,SpringBoot
--- 

  

### mvc
```xml
spring.mvc.async.request-timeout
#设定async请求的超时时间，以毫秒为单位，如果没有设置的话，以具体实现的超时时间为准，比如tomcat的servlet3的话是10秒.
spring.mvc.date-format
#设定日期的格式，比如dd/MM/yyyy.
spring.mvc.favicon.enabled
#是否支持favicon.ico，默认为: true
spring.mvc.ignore-default-model-on-redirect
#在重定向时是否忽略默认model的内容，默认为true
spring.mvc.locale
#指定使用的Locale.
spring.mvc.message-codes-resolver-format
#指定message codes的格式化策略(PREFIX_ERROR_CODE,POSTFIX_ERROR_CODE).
spring.mvc.view.prefix
#指定mvc视图的前缀.
spring.mvc.view.suffix
#指定mvc视图的后缀.
```

### messages
```xml
spring.messages.basename
#指定message的basename，多个以逗号分隔，如果不加包名的话，默认从classpath路径开始，默认: messages
spring.messages.cache-seconds
#设定加载的资源文件缓存失效时间，-1的话为永不过期，默认为-1
spring.messages.encoding
#设定Message bundles的编码，默认: UTF-8
```
### resource
```xml
spring.resources.add-mappings
#是否开启默认的资源处理，默认为true
spring.resources.cache-period
#设定资源的缓存时效，以秒为单位.
spring.resources.chain.cache
#是否开启缓存，默认为: true
spring.resources.chain.enabled
#是否开启资源 handling chain，默认为false
spring.resources.chain.html-application-cache
#是否开启h5应用的cache manifest重写，默认为: false
spring.resources.chain.strategy.content.enabled
#是否开启内容版本策略，默认为false
spring.resources.chain.strategy.content.paths
#指定要应用的版本的路径，多个以逗号分隔，默认为:[/**]
spring.resources.chain.strategy.fixed.enabled
#是否开启固定的版本策略，默认为false
spring.resources.chain.strategy.fixed.paths
#指定要应用版本策略的路径，多个以逗号分隔
spring.resources.chain.strategy.fixed.version
#指定版本策略使用的版本号
spring.resources.static-locations
#指定静态资源路径，默认为classpath:[/META-INF/resources/,/resources/, /static/, /public/]以及context:/
```

###  freemarker
```xml
spring.freemarker.allow-request-override
#指定HttpServletRequest的属性是否可以覆盖controller的model的同名项
spring.freemarker.allow-session-override
#指定HttpSession的属性是否可以覆盖controller的model的同名项
spring.freemarker.cache
#是否开启template caching.
spring.freemarker.charset
#设定Template的编码.
spring.freemarker.check-template-location
#是否检查templates路径是否存在.
spring.freemarker.content-type
#设定Content-Type.
spring.freemarker.enabled
#是否允许mvc使用freemarker.
spring.freemarker.expose-request-attributes
#设定所有request的属性在merge到模板的时候，是否要都添加到model中.
spring.freemarker.expose-session-attributes
#设定所有HttpSession的属性在merge到模板的时候，是否要都添加到model中.
spring.freemarker.expose-spring-macro-helpers
#设定是否以springMacroRequestContext的形式暴露RequestContext给Spring’s macro library使用
spring.freemarker.prefer-file-system-access
#是否优先从文件系统加载template，以支持热加载，默认为true
spring.freemarker.prefix
#设定freemarker模板的前缀.
spring.freemarker.request-context-attribute
#指定RequestContext属性的名.
spring.freemarker.settings
#设定FreeMarker keys.
spring.freemarker.suffix
#设定模板的后缀.
spring.freemarker.template-loader-path
#设定模板的加载路径，多个以逗号分隔，默认: ["classpath:/templates/"]
spring.freemarker.view-names
#指定使用模板的视图列表.
```

### velocity
```xml
spring.velocity.allow-request-override
#指定HttpServletRequest的属性是否可以覆盖controller的model的同名项
spring.velocity.allow-session-override
#指定HttpSession的属性是否可以覆盖controller的model的同名项
spring.velocity.cache
#是否开启模板缓存
spring.velocity.charset
#设定模板编码
spring.velocity.check-template-location
#是否检查模板路径是否存在.
spring.velocity.content-type
#设定ContentType的值
spring.velocity.date-tool-attribute
#设定暴露给velocity上下文使用的DateTool的名
spring.velocity.enabled
#设定是否允许mvc使用velocity
spring.velocity.expose-request-attributes
#是否在merge模板的时候，将request属性都添加到model中
spring.velocity.expose-session-attributes
#是否在merge模板的时候，将HttpSession属性都添加到model中
spring.velocity.expose-spring-macro-helpers
#设定是否以springMacroRequestContext的名来暴露RequestContext给Spring’s macro类库使用
spring.velocity.number-tool-attribute
#设定暴露给velocity上下文的NumberTool的名
spring.velocity.prefer-file-system-access
#是否优先从文件系统加载模板以支持热加载，默认为true
spring.velocity.prefix
#设定velocity模板的前缀.
spring.velocity.properties
#设置velocity的额外属性.
spring.velocity.request-context-attribute
#设定RequestContext attribute的名.
spring.velocity.resource-loader-path
#设定模板路径，默认为: classpath:/templates/
spring.velocity.suffix
#设定velocity模板的后缀.
spring.velocity.toolbox-config-location
#设定Velocity Toolbox配置文件的路径，比如 /WEB-INF/toolbox.xml.
spring.velocity.view-names
#设定需要解析的视图名称.
```

### http
```xml
spring.hateoas.apply-to-primary-object-mapper
#设定是否对object mapper也支持HATEOAS，默认为: true
spring.http.converters.preferred-json-mapper
#是否优先使用JSON mapper来转换.
spring.http.encoding.charset
#指定http请求和相应的Charset，默认: UTF-8
spring.http.encoding.enabled
#是否开启http的编码支持，默认为true
spring.http.encoding.force
#是否强制对http请求和响应进行编码，默认为true
```

### json
```xml
spring.jackson.date-format
#指定日期格式，比如yyyy-MM-dd HH:mm:ss，或者具体的格式化类的全限定名
spring.jackson.deserialization
#是否开启Jackson的反序列化
spring.jackson.generator
#是否开启json的generators.
spring.jackson.joda-date-time-format
#指定Joda date/time的格式，比如yyyy-MM-dd HH:mm:ss). 如果没有配置的话，dateformat会作为backup
spring.jackson.locale
#指定json使用的Locale.
spring.jackson.mapper
#是否开启Jackson通用的特性.
spring.jackson.parser
#是否开启jackson的parser特性.
spring.jackson.property-naming-strategy
#指定PropertyNamingStrategy (CAMEL_CASE_TO_LOWER_CASE_WITH_UNDERSCORES)或者指定PropertyNamingStrategy子类的全限定类名.
spring.jackson.serialization
#是否开启jackson的序列化.
spring.jackson.serialization-inclusion
#指定序列化时属性的inclusion方式，具体查看JsonInclude.Include枚举.
spring.jackson.time-zone
#指定日期格式化时区，比如America/Los_Angeles或者GMT+10.
```