# 权限管理

权限控制不依赖任何权限管理实现，hsweb默认提供了2种权限管理的方式。

## 配置文件方式

{% code-tabs %}
{% code-tabs-item title="application.yml" %}
```yaml
hsweb:
    users:
        admin:
          name: 超级管理员
          username: admin
          password: admin
          roles:
            - id: admin
              name: 管理员
          permissions:  # 用户的权限
            - id: test #权限标识,可以理解为资源
              actions: query,get,update,delete #用户持有对该权限的操作
              dataAccesses:   # 数据权限配置,此配置表示在对test进行query操作的时候,不能查询password和salt字段
                - action: query  # DataAccessConfig.action
                  type: DENY_FIELDS # DataAccessConfig.type
                  fields:     # 由对应的DataAccessConfigConvert解析生成对应的DataAccessConfig实现
                    - password
                    - salt
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 数据库方式

见 [业务功能-用户权限](../ye-wu-gong-neng/yong-hu-quan-xian.md)

## 自定义

在某些场景默认的权限管理可能不能完全满足需求，比如：一个系统有多个用户体系，有独立用户管理系统。

### 替换默认的权限管理

方式一。实现AuthenticationManager接口，注入到Spring。

方式二。实现AuthenticationSupplier接口，注入到Spring。

### 多用户体系权限管理

hsweb提供了根据token来区分用户类型的权限信息。实现接口，ThirdPartAuthenticationManager。 与UserTokenParser中得到的tokenType一致即可。



