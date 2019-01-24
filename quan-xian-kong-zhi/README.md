# 权限控制

hsweb提供了一个灵活的权限控制,设置方式,实现了多维度,可自定义的RBAC和数据权限控制.

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

## 定义用户和权限

在`application.yml`中加入配置

```yaml
hsweb:
    users:
        admin:
          name: 超级管理员
          username: admin
          password: admin
          roles: #用户的角色
            - id: admin
              name: 管理员
            - id: user
              name: 用户
          permissions:  # 用户的权限
            - id: test #权限标识,可以理解为资源
              actions: query,get,update,delete #用户持有对该资源的操作
              dataAccesses:   # 数据权限配置,此配置表示在对test进行query操作的时候,不能查询password和salt字段
                - action: query
                  type: DENY_FIELDS
                  fields:     
                    - password
                    - salt
```

## 定义要控制的接口

在`TestController`中加入权限控制,在类上注解`@Authorize(permission = "test")`

```java
@RestController
@RequestMapping("/test")
@Authorize(permission = "test")
public class TestController implements SimpleGenericEntityController<TestEntity, String, QueryParamEntity> {

    @Autowired
    TestService testService;

    @Override
    public CrudService<TestEntity, String> getService() {
        return testService;
    }
}
```
