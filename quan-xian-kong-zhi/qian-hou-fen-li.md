# 前后分离

用户认证信息使用`UserTokenManager`进行管理，hsweb默认提供了基于`sessionId`的`token`实现，如果在需要前后分离，可以自行拓展`UserTokenGenerator`和`UserTokenParser`。

## 自定义Token策略

1.  自定义UserTokenGenerator

{% code-tabs %}
{% code-tabs-item title="MyUserTokenGenerator.java" %}
```java
@Component
public class MyUserTokenGenerator implements UserTokenGenerator {

    private int timeout = 30 * 60 * 1000;

    @Override
    public String getSupportTokenType() {
       // 登录时,请求参数为 token_type = token 时,此配置生效
        return "token";
    }

    @Override
    public GeneratedToken generate(Authentication authentication) {
        String token = IDGenerator.MD5.generate();

        return new GeneratedToken() {
            @Override
            public Map<String, Object> getResponse() {
                //返回给前端的参数,前端拿到此参数后自行处理
                return Collections.singletonMap("token", token);
            }

            @Override
            public String getToken() {
                return token;
            }

            @Override
            public int getTimeout() {
                // token 超时时间,超过时间没有请求时,自动失效
                return timeout;
            }
        };
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

2. 自定义UserTokenParser

{% code-tabs %}
{% code-tabs-item title="MyUserTokenParser.java" %}
```java
@Component
public class MyUserTokenParser implements UserTokenParser {
    @Override
    public ParsedToken parseToken(HttpServletRequest request) {
        //获取参数中的token
        String token = request.getParameter("token");

        if(StringUtils.isEmpty(token))return null;

        return () -> token;
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

3. 在登录时，多传入一个参数: token\_type=token



