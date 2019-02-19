# 登录授权

## 授权

使用`hsweb-authorization-api`提供的监听器,类`UserOnSignIn`监听用户授权事件`AuthorizationSuccessEvent` 当用户完成授权\(授权方式可自行实现或者使用框架默认的授权方式,主要触发该事件即可\).授权通过后会触发该事件.流程如下

![](../.gitbook/assets/autz-flow.png)

1. 完成授权,触发`AuthorizationSuccessEvent`
2. `UserOnSignIn` 收到`AuthorizationSuccessEvent`事件,获取参数`token_type`\(默认为`sessionId`\),以及授权信息
3. 根据`token_type` 调用对应的`UserTokenGenerator`生成token.
4. 将token和授权信息中的userId注册到`UserTokenManager`
5. 将token返回给授权接口

{% hint style="warning" %}
UserTokenGenerator 默认使用sessionId作为token,你可以实现UserTokenGenerator和UserTokenParser来进行自定义token实现 [前后分离](qian-hou-fen-li.md)
{% endhint %}

## 授权登录接口

http接口: `POST /authorize/login`, 登录接口支持2种`content-type`,`application/json`\(Json RequestBody方式\)和`application/x-www-form-urlencoded`\(表单方式\), 请在调用等时候指定对应等`content-type`.必要参数: `username` 和 `password`.

⚠️注意: 此接口只实现了简单的登录逻辑,不过会通过发布各种事件来实现自定义的逻辑处理.

1. `AuthorizationDecodeEvent` 在接收到登录请求之后触发,如果在登录前对用户名密码进行里加密,可以通过监听此事件实现对用户名密码的解密操作
2. `AuthorizationBeforeEvent` 在`AuthorizationDecodeEvent`事件完成后触发,可通过监听此事件并获取请求参数,实现验证码功能
3. `AuthorizationSuccessEvent` 在授权成功后触发.注意: 权限控制模块也是通过监听此事件来完成授权
4. `AuthorizationFailedEvent` 授权失败时触发.当发生过程中异常时触发此事件

## 注销

与授权同理,类`UserOnSignOut`监听`AuthorizationExitEvent` ,当触发事件后,调用`UserTokenManager`移除当前登录的token信息

