# 日志

## 访问日志 API

引入依赖

{% code-tabs %}
{% code-tabs-item title="pom.xml" %}
```markup
  <dependency>
      <groupId>org.hswebframework.web</groupId>
      <artifactId>hsweb-access-logging-api</artifactId>
      <version>${hsweb.framework.version}</version>
  </dependency>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Controller类或者方法上,注解 `@AccessLogger("功能描述")` 如果正在使用swagger,只需要注解swagger的`@Api(tags="功能说明",value="XXX功能")`

## 开启访问日志

引入依赖

{% code-tabs %}
{% code-tabs-item title="pom.xml" %}
```markup
<dependency>
      <groupId>org.hswebframework.web</groupId>
      <artifactId>hsweb-access-logging-aop</artifactId>
      <version>${hsweb.framework.version}</version>
  </dependency>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

引入依赖,`hsweb-access-logging-aop`,在启动类中注解`@EnableAccessLogger`.

日志模块只生产日志，日志产生后会推送2个事件:

1. AccessLoggerBeforeEvent 在接收到请求，执行被请求接口之前触发
2. AccessLoggerAfterEvent 在请求接口被执行后触发

事件中包含了`AccessLoggerInfo`日志信息

如: 

```java
@EventListener
public void handleAccessLogger(AccessLoggerAfterEvent event){
     AccessLoggerInfo logInfo =event.getLogger();
    //提取日志并记录到数据库
}
```

{% hint style="danger" %}
注意，AccessLoggerInfo不能直接序列化，请提取出需要的信息进行处理。
{% endhint %}

