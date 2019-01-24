# 说明

## 简介

`hsweb` 是一个基于`java8`，`spring-boot`构建的企业后台管理基础框架。用于快速搭建后台管理类项目。 `hsweb-framework` 是整个项目的核心，使用`maven`多模块进行管理。  `hsweb-framework` 只实现了后端的功能， 通过`restful`发布接口。 ui请查看: [miniui实现](https://github.com/hs-web/hsweb3-demo)， layui实现，antd实现。

{% hint style="warning" %}
我们推荐使用`maven`的方式来使用，而**不是**将整个项目`clone`到本地修改和添加业务。
{% endhint %}

hsweb 提供了一些开发和业务常用的功能,如:  [通用增删改查](zeng-shan-gai-cha/tong-yong-crud.md)，[细粒度\(按钮,行,列\)的权限控制](quan-xian-kong-zhi/)，[多数据源](duo-shu-ju-yuan/)，[数据字典](ye-wu-gong-neng/shu-ju-zi-dian.md)，[组织架构](ye-wu-gong-neng/zu-zhi-jia-gou.md)，[工作流](ye-wu-gong-neng/gong-zuo-liu.md)等，帮助项目快速开发。

{% hint style="info" %}
hsweb-framework的模块较多，是为了将功能细化后，可以根据需求选择适合自己项目的模块。需要什么就使用什么，无需引入一些无用的功能。在实际项目中，应该根据需求建立合适的项目结构。
{% endhint %}

在使用hsweb之前，我们假设您已经掌握或者能使用以下技术栈了

1. Java
2. Maven
3. Spring
4. Spring-Boot
5. Mybatis
6. IntelliJ IDEA

{% hint style="info" %}
如果已经熟悉以上技术可以让你快速上手hsweb。
{% endhint %}

一些基于hsweb的项目实践: 

| 项目 | 说明 |
| :---: | :--- |
| [hsweb-payment](http://payment.hsweb.pro) | 多商户，多渠道，多配置的开源聚合支付项目 |
| [hsweb-iot-cloud](http://iot-cloud.hsweb.pro) | 物联网基础平台 |



