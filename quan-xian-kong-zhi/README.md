# 权限控制

hsweb提供了一个灵活的权限控制,设置方式,实现了多维度,可自定义的RBAC和数据权限控制.

以下例子建立在[快速入门](kuai-su-kai-shi)的基础上

## 引入依赖
```xml
    <!--权限控制-->
    <dependency>
        <groupId>org.hswebframework.web</groupId>
        <artifactId>hsweb-authorization-basic</artifactId>
        <version>${hsweb.framework.version}</version>
    </dependency>
```

## 添加注解

在启动类上注解:`@EnableAopAuthorize`

```java
@SpringBootApplication
@MapperScan(basePackages = "com.mycompany.dao", markerInterface = Dao.class)
@EnableAopAuthorize //开启AOP权限控制
public class MyProjectApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyProjectApplication.class, args);
    }
}
```

## 控制权限
