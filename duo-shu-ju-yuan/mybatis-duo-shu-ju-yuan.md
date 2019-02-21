# Mybatis 多数据源

hsweb拓展了mybatis配置，支持在同一个Mapper中切换数据源。只需要在配置文件中加入:

```yaml
mybatis:
  dynamic-datasource: true  #开启动态数据源支持
```

或者创建多个SqlSession管理不同的Mapper。具体方法请自行百度。

如果要使用事务，请参照[Jta实现](jta-duo-shu-ju-yuan-shi-wu.md).

