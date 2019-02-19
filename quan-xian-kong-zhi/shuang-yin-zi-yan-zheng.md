# 双因子验证

hsweb提供了双因子验证功能,默认提供了totp实现,可通过totp app\(比如:[Google Authenticator](https://github.com/google/google-authenticator)\)进行验证码获取。可以通过拓展实现接口短信验证等功能。

### 开启双因子验证

配置 application.yml

```text
hsweb:
    authorize:
        two-factor:
            enable: true
        totp:
          enable: true # 开启totp支持
          domain: hsweb.me
```

在需要验证的接口上注解:

```java
@PostMapping
@TwoFactor("update-password") //调用此接口时,需要进行双因子验证
public ResponseMessage<Boolean> updatePassword(String password){
    //业务逻辑
}
```

### TOTP

如果开启了TOTP 双因子验证,在创建用户时会生成用户的totp验证密钥,并发出事件:`TotpTwoFactorCreatedEvent` ，通过监听此事件进行自定义处理：比如发送邮件通知等。

### 验证流程

1. 调用接口,判断当前是否已经验证通过
2. 如果没有验证通过,返回{"status":403,"code":"need\_tow\_factor","message":"需要进行双因子验证","provider":"totp"}
3. 客户端根据返回码,引导用户输入验证码
4. 将验证码添加到第一步的接口参数中,默认参数名为: `verifyCode`
5. 验证通过后,在`@TwoFactor(timeout=600000L)`毫秒内再次调用无需重复输入,默认为`10分钟`.

### 自定义验证策略

hsweb默认只提供了TOTP方式的验证实现,，但是提供了拓展方式可以自定义验证策略，比如：短信验证

1. 继承抽象类: `DefaultTwoFactorValidatorProvider`并重写对应方法
2. 修改配置: `hsweb.authorize.two-factor.default-provider=自定义的provider`，或者在注解中指定provider: `@TwoFactor(value="验证接口标识",provider="自定义的provider")`

