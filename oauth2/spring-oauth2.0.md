#### 授权服务器配置

使用@EnableAuthorizationServer，继承AuthorizationServerConfigurerAdapter 来配置OAuth2.0授权服务器

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    // ...
}
```

AuthorizationServerConfigurerAdapter要求配置以下几个类，它们由Spring创建的独立的配置对象，会被Spring传入AuthorizationServerConfigurer中进行配置

```java
public class AuthorizationServerConfigurerAdapter implements AuthorizationServerConfigurer {
    public AuthorizationServerConfigurerAdapter() {}
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {}
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {}
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {}
}

```

