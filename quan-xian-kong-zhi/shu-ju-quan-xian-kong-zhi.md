# 数据权限控制

在RBAC权限控制中实现了基于角色的权限控制，但是只能控制到操作\(按钮\)级别的权限。 在某些场景，可能还需要更细粒度的权限控制，比如: 控制普通用户只能查看自己创建的数据，部门经理可以查看自己部门下所有人员的数据，管理员可以查看和修改所有数据。hsweb实现了灵活的数据权限控制，默认提供了多种控制逻辑。

## 术语

1. 数据权限配置\(DataAccessConfig\): 用于配置数据权限的控制逻辑
2. 动态条件\(QueryParamEntity\): 用于支持动态查询的参数实体

## 原理

使用AOP拦截请求，根据用户的`数据权限配置`修改请求参数或返回值。

![](../.gitbook/assets/autz-handle-flow.png)

## 声明式数据权限

在方法或者类上注解 @Authorize\(permission="user",action="query",dataAccess=@RequiresDataAccess\) 即开启数据权限控制，此注解只用于声明需要进行数据权限控制，具体的权限控制逻辑由数据权限配置去实现。

{% hint style="warning" %}
数据权限只会修改被控制接口的参数，不会侵入到更下层的Service和Dao，因此在使用声明式数据权限的时候，查询时应该使用`动态条件`。
{% endhint %}

{% hint style="warning" %}
数据权限配置是根据permission和action进行关联，因此在最终的权限控制声明\(`AuthorizeDefinition`\)中，permission和action不能为空。
{% endhint %}

## 编程式数据权限

除了声明式数据权限控制，还可以通过编程式进行控制:

```java
@GetMapping
public ResponseMessage<PagerResult<Order>> queryOrder(Authentication auth,QueryParamEntity param){

  String userId = auth.getUser().getId();//当前登录用户ID

  //where creatorId = ? and ( 前端传入的参数 )
  return ok(param.toNestQuery(query->query.and("creatorId",userId))
              .execute(service::selectPager));

}
```

## 数据权限配置

数据权限配置是根据`permission`和`action`进行关联的，在权限信息\(`Authentication(permissions[{dataAccess:[ 数据权限配置 ]}])`\)中保存用户设置的数据权限。数据权限配置在权限\(`Authentication`\)初始化的时候根据具体的`AuthenticationManager`实现进行初始化。

### 数据结构

`DataAccessConfig`接口由`action`和`type`字段组成.`action`用于标识在进行什么操作的时候需要进行数据权限控制,和RBAC权限中的action对应。 `type`为配置类型的标识。具体的数据权限控制配置，需要实现此接口。

### 初始化逻辑

不同的数据权限有不同的配置，在权限初始化的时候，不同的配置由对应的`DataAccessConfigConvert`实现进行初始化。使用`DataAccessConfigConvert`的目的是：数据权限的配置是动态的，配置的来源也不同，可能是配置文件，数据库或者jwt，使用`DataAccessConfigConvert`接口，可以灵活的拓展各种数据权限配置。

### 控制逻辑

`hsweb-authorization-basic`模块实现了`DataAccessController`来进行数据权限控制，获取spring容器中所有实现`DataAccessHandler`接口的bean. `DataAccessHandler` 就是具体的控制实现。在接收到请求的时候，如果需要进行数据权限控制\(`permission`和`action`匹配的`DataAccessConfig`\)，使用策略模式依次调用`DataAccessHandler.isSupport(DataAccessConfig)`方法，判断`DataAccessHandler`是否支持，如果支持，则调用`DataAccessHandler.handle`方法完成控制。

### 配置方式

已经提供了2种权限的配置方式：

1. 配置文件方式

   {% code-tabs %}
   {% code-tabs-item title="application.yml" %}
   ```yaml
   hsweb:
    users:
        admin:
          name: 超级管理员
          username: admin
          password: admin
          permissions:  # 用户的权限
            - id: test #权限标识,可以理解为资源
              actions: query,get,update,delete #用户持有对该资源的操作
              dataAccesses:   # 数据权限配置,此配置表示在对test进行query操作的时候,不能查询password和salt字段
                - action: query  # DataAccessConfig.action
                  type: DENY_FIELDS # DataAccessConfig.type
                  fields:     # 由对应的DataAccessConfigConvert解析生成对应的DataAccessConfig实现
                    - password
                    - salt
   ```
   {% endcode-tabs-item %}
   {% endcode-tabs %}

2. 数据库配置方式，[查看文档](../ye-wu-gong-neng/yong-hu-quan-xian.md)

## 自定义权限控制

上文中已经接受了数据权限配置的逻辑，在需要自定义数据权限控制的时候，只需要: 1. 创建自定义数据权限控制的配置,并实现`DataAccessConfig`接口。 2. 创建`DataAccessConfigConvert`实现类实现对应方法并注入到Spring。 3. 创建`DataAccessHandler`实现类实现对应方法并注入到Spring，

完成。

## 默认提供的数据权限



