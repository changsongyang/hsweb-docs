# 多数据源管理

多数据源的配置由DynamicDataSourceConfigRepository进行管理，默认实现为InSpringDynamicDataSourceConfigRepository，使用多数据源的时候，直接在spring容器中声明多个数据源即可。

## 使用数据库来管理多数据源

参照[数据源配置文档](../ye-wu-gong-neng/shu-ju-yuan-pei-zhi.md)

## 自定义配置管理

实现DynamicDataSourceConfigRepository接口，如果使用了jta模块，则实现JtaDataSourceRepository接口。

