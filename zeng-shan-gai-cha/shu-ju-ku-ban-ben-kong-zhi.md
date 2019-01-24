# 数据库版本控制

## 说明

hsweb使用nashorn来定义模块的初始化，更新脚本\(`hsweb-starter.js`\)。在模块首次引入,或者发生版本更新时，会执行对应的代码，利用此功能，可以实现对数据库的版本控制。

## 定义脚本

在模块的 resources下创建hsweb-starter.js. 内容如下: 

```text
//组件信息
var info = {
    groupId: "org.hswebframework",
    artifactId: "hsweb-payment-pay",
    version: "1.0.2", //当前版本
    website: "http://payment.hsweb.io",
    author: "zhouhao",
    comment: "支付模块"
};

//版本更新信息
var versions = [
    {
        version: "1.0.1",//版本升级到1.0.1时执行
        upgrade: function (context) {
            var database = context.database;
            //增加渠道供应商
            database.createOrAlter("pay_order")
                .addColumn().name("channel_provider").varchar(128).notNull().comment("渠道供应商").commit()
                .addColumn().name("channel_provider_name").notNull().varchar(128).comment("渠道供应商名称").commit()
                .comment("支付订单表").commit();
        }
    },
    {
        version: "1.0.2", //版本升级到1.0.2时执行
        upgrade: function (context) {
            var database = context.database;
            database.createOrAlter("pay_order")
                .addColumn().name("comment").varchar(2048).comment("说明").commit()
                .comment("支付订单表").commit();
        }
    }
];
var JDBCType = java.sql.JDBCType;

//首次启动此模块的时候执行
function install(context) {
    var database = context.database;
    database.createOrAlter("pay_order")
        .addColumn().name("id").varchar(32).notNull().primaryKey().comment("ID").commit()
        .addColumn().name("order_id").varchar(128).notNull().comment("商户订单号").commit()
        .addColumn().name("merchant_id").varchar(128).notNull().comment("商户ID").commit()
        //.....省略了更多字段设置
        .addColumn().name("status").varchar(16).notNull().comment("订单状态").commit()
        .index().name("idx_order_id").column("order_id").unique().commit()//订单号唯一索引
        .index().name("idx_order_merchant_id").column("merchant_id").commit()
        .comment("支付订单表").commit();
}

function initialize(){
    //首次使用此模块，在全部依赖安装完成后执行
}

//以下为固定的代码,无需修改
dependency.setup(info)
    .onInstall(install)
    .onUpgrade(function (context) { //更新时执行
        var upgrader = context.upgrader;
        upgrader.filter(versions)
            .upgrade(function (newVer) {
                newVer.upgrade(context);
            });
    })
    .onUninstall(function (context) { //卸载时执行

    }).onInitialize(initialize); //初始化
```

## 执行流程

1. 在应用启动完成后,会获取所有的hsweb-starter.js文件。
2. 执行js并获取到模块信息。
3. 根据模块信息判断模块是否安装或者更新。
4. 执行对应的安装或者更新操作。
5. 如果模块是首次执行，将会在全部模块安装完成后执行初始化操作。

