---
title: 0分布式认证和授权方案
date: 2022-02-25 21:27:40
permalink: /pages/43f9c7/
categories:
  - 框架
  - Spring
tags:
  - 
---
# Spring Security OAuth 2.0 认证和授权方案

## 1 OAuth 2.0

### 1.1 OAuth 2.0介绍

[OAuth 2.0](https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html) 是目前最流行的授权机制，用来授权第三方应用，获取用户数据。

- OAuth （Open Authorization 开放授权）是一个开放标准，允许用户授权第三方应用以便访问他们存储在其他服务提供者上的信息，而不需要用户将用户名和密码提供给第三方应用或分享用户数据的所有内容。

- OAuth 2.0是OAuth 协议的延续版本，但是不兼容OAuth 1.0即完全废除了OAuth 1.0。很多大公司如Google、Microsoft等都提供了OAuth 认证服务，这些都足以说明OAuth 标准逐渐成为开放资源授权的标准。

- 大家最熟悉的就是几乎每个人都用过的，比如用微信登录、用 QQ 登录、用微博登录、用 Google 账号登录、用 github 授权登录等等，这些都是典型的 OAuth2 使用场景。

- 下边分析一个OAuth 2.0认证的例子，通过这个例子可以很好的理解OAuth 2.0协议的认证过程，本例子是借助知乎网站使用QQ认证的过程，过程简要描述如下：

- 用户借助QQ认证登录知乎网站，用户就不需要在知乎网站上单独注册用户，那么怎么才算认证成功呢？知乎网站需要成功从QQ获取用户的身份信息则认为用户认证成功，那如何从QQ获取用户的身份信息？用户信息的拥有者是用户本人，QQ需要经过用户的同意方可为知乎网站生成令牌，知乎拿到此令牌才可以从QQ获取用户的信息。

- 1️⃣客户端请求第三方授权：用户进入知乎的登录页面，点击QQ的图标以QQ的账号和密码登录系统，用户是自己在QQ里信息的资源拥有者。

   ![oauth2-7](https://gitee.com/linbingxing/image/raw/master/oauth2-7.png)

- 2️⃣资源拥有者同意给客户端授权：资源拥有者输入QQ的账号和密码表示资源拥有者同意给客户端授权，QQ会对资源拥有者的身份进行验证，验证通过后，QQ会询问用户是否给ProcessOn访问自己的QQ数据，用户打开QQ手机版，点击“确认登录”表示同意授权，QQ认证服务器会颁发一个授权码，并重定向到ProcessOn的网站。

![oauth2-8](https://gitee.com/linbingxing/image/raw/master/oauth2-8.png)

- 3️⃣客户端获取到授权码，请求认证服务器申请令牌：此过程用户看不到，客户端应用程序请求认证服务器，请求携带授权码。
- 4️⃣认证服务器向客户端响应令牌：QQ认证服务器验证了客户端请求的授权码，如果合法则给客户端颁发令牌，令牌是客户端访问资源的通行证。此交互过程用户是看不到的，当客户端拿到令牌后，用户在ProcessOn网站上看到已经登录成功。
- 5️⃣客户端请求资源服务器的资源：客户端携带令牌请求访问QQ服务器获取用户的基本信息。
- 6️⃣资源服务器返回受保护资源：资源服务器校验令牌的合法性，如果合法则向用户响应资源信息内容。

### 1.2 OAuth 2.0角色

OAuth2 协议中定义了四个核心的角色：**资源、客户端、授权服务器和资源服务器**。

![oauth2_1](https://gitee.com/linbingxing/image/raw/master/oauth2_1.png)

- 1️⃣客户端：本身不存储资源，需要通过资源拥有者的授权去请求资源服务器的资源，比如：Android客户端、Web客户端、微信客户端等。
- 2️⃣资源拥有者：通常为用户，也可以是应用程序，即该资源的拥有者。
- 3️⃣授权服务器（也称为认证服务器）：用于服务提供商对资源拥有的身份进行认证，对访问资源进行授权，认证成功后会给客户端发放令牌(access_token)，作为客户端访问资源服务器的凭证。
- 4️⃣资源服务器：存储资源的服务器，比如QQ服务器。

> 服务提供商不会随便允许任意一个客户端接入到它的授权服务器的，服务提供商会给准入的接入方一个身份，用于接入的凭证：client_id（客户端标识）和client_secret（客户端密钥）。

### 1.3 OAuth 2.0四种授权模式

#### 1.3.1 概述

- OAuth 2.0规定了四种授权模式。
- 1️⃣授权码（authorization code）。
- 2️⃣隐藏式（implicit）。
- 3️⃣密码（password）。
- 4️⃣客户端凭证（client credentials）。

#### 1.3.2  授权码（**Authorization Code**）

- 授权码方式，指的是第三方应用先申请一个授权码，然后再用这个授权码去获取令牌。

- 授权码方式是最常用的方式，安全性也最高，它适用于那些有后端的Web应用。授权码通过前端发送，令牌是存储到后端的，而且所有和资源服务器的通信都是在后端完成。这样的前后端分离，可以避免令牌泄露。

第一步，A网站提供一个链接，用户点击后就会跳转到B网站，授权用户数据给A网站使用。下面就是A网站跳转到B网站的一个示例连接。

```shell
https://b.com/oauth/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

上面 URL 中，`response_type`参数表示要求返回授权码（`code`），`client_id`参数让 B 知道是谁在请求，`redirect_uri`参数是 B 接受或拒绝请求后的跳转网址，`scope`参数表示要求的授权范围（这里是只读）。

![oauth2-2](https://gitee.com/linbingxing/image/raw/master/oauth2-2.jpg)

第二步，用户点击后，B网站会要求用户登录，然后询问是否同意给A网站授权。用户表示同意，这时B网站就会调回redirect_uri参数指定的网址。跳转的时候，会传一个授权码。

```shell
https://a.com/callback?code=AUTHORIZATION_CODE
```

上面的URL中，`code`参数就是授权码。

![oauth2-3](https://gitee.com/linbingxing/image/raw/master/oauth2-3.jpg)

第三步，A 网站拿到授权码以后，就可以在后端，向 B 网站请求令牌。

```
https://b.com/oauth/token?
 client_id=CLIENT_ID&
 client_secret=CLIENT_SECRET&
 grant_type=authorization_code&
 code=AUTHORIZATION_CODE&
 redirect_uri=CALLBACK_URL
```

上面 URL 中，`client_id`参数和`client_secret`参数用来让 B 确认 A 的身份（`client_secret`参数是保密的，因此只能在后端发请求），`grant_type`参数的值是`AUTHORIZATION_CODE`，表示采用的授权方式是授权码，`code`参数是上一步拿到的授权码，`redirect_uri`参数是令牌颁发后的回调网址。

![oauth2-4](https://gitee.com/linbingxing/image/raw/master/oauth2-4.jpg)

第四步，B 网站收到请求以后，就会颁发令牌。具体做法是向redirect_uri指定的网址，发送一段 JSON 数据。

```json
{    
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101,
  "info":{...}
}
```

上面 JSON 数据中，`access_token`字段就是令牌，A 网站在后端拿到了。

总结一下，整个授权码工作流程如下：

![授权码工作流程](https://gitee.com/linbingxing/image/raw/master/oauth2-10.png)

#### 1.3.3 **密码模式（Password Credentials）**

- 如果你高度信任某个应用，OAuth 2.0允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为密码式。
  第一步，A网站要求用户提供B网站的用户名和密码。拿到以后，A直接向B请求令牌。

```
https://oauth.b.com/token?
  grant_type=password&
  username=USERNAME&
  password=PASSWORD&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET
```

上面的URL中，`grant_type`参数是授权方式，这里的`password`表示密码式，`username`和`password`是B的用户名和密码。

第二步，B网站验证身份通过后，直接给出令牌。注意，此时不需要跳转，而是把令牌放在JSON里面，作为HTTP回应，A网站因此拿到令牌。

> 这种方式需要用户给出自己的用户名和密码，风险非常大，因此只适用于其他授权都无法采用的情况，而且必须是用户高度信任的应用。

其授权流程如下图所示：

![oauth2-11](https://gitee.com/linbingxing/image/raw/master/oauth2-11.png)

#### 1.3.4 ****简化模式（Implicit）****

 - 有些web应用是纯前端应用，没有后端。这时就不能使用上面的方式了，必须将令牌存储在前端。这种方式没有授权码这个步骤，所以称为授权码的隐藏式。
第一步，A网站提供一个连接，要求用户跳转到B网站，授权用户数据给A网站使用。

```
https://b.com/oauth/authorize?
  response_type=token&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

上面的URL中，`response_type`参数为`token`，表示要求直接返回令牌。

第二步，用户点击后，跳转到B网站，登录后同意给A网站授权。这时，B网站就会跳转到`redirect_uri`参数指定的跳转网址，并且把令牌作为URL参数，传给A网站。

```
https://a.com/callback#token=ACCESS_TOKEN
```

上面的URL中，`token`的参数就是令牌，A网站因此直接在前端拿到令牌。

注意：令牌的位置是URL锚点，而不是查询字符串，这是因为OAuth 2.0允许跳转网址是HTTP协议，因为存在“中间人攻击”的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄露令牌的风险。

![oauth2-6](https://gitee.com/linbingxing/image/raw/master/oauth2-6.jpg)
隐藏式是把令牌直接传给前端，很不安全。因此，只能用于一些安全性要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间有效，浏览器关闭，令牌就失效了。

####  1.3.5 **客户端模式（Client Credentials）**

- 这种方式适用于没有前端的命令行应用，即在命令行下请求令牌。
第一步，A应用在命令行向B发出请求。

```
https://oauth.b.com/token?
  grant_type=client_credentials&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET
```

> 上面的URL中，`grant_type`参数是授权方式，这里的`client_credentials`表示客户端凭证式，`client_id`和`client_secret`用来让B确认A的身份。

第二步，B网站验证通过后，直接返回令牌。

> 这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。

### 1.4 OAuth 2.0令牌使用

访问令牌是什么？令牌是 OAuth2 协议中非常重要的一个概念，本质上也是一种代表用户身份的授权凭证，但与普通的用户名和密码信息不同，令牌具有针对资源的访问权限范围和有效期。

- A网站拿到令牌以后，就可以向B网站的API请求数据了。
- 每个发送的API请求，都必须携带令牌。具体的做法就是在请求的头信息，加上Authorization字段，令牌就放在这个字段里面。

```shell
curl -H "Authorization: Bearer ACCESS_TOKEN" \
"https://api.b.com"
```

> 上面命令中，`Authorization`就是拿到的令牌。

如下所示就是一种常见的令牌信息：

```json
{

    "access_token": "0efa61be-32ab-4351-9dga-8ab668ababae",

    "token_type": "bearer",

    "refresh_token": "738c42f6-79a6-457d-8d5a-f9eab0c7cc5e",

    "expires_in": 43199,

    "scope": "webclient"

}
```

- access_token：代表 OAuth2 的令牌，当访问每个受保护的资源时，用户都需要携带这个令牌以便进行验证。
- token_type：代表令牌类型，OAuth2 协议中有多种可选的令牌类型，包括 Bearer 类型、MAC 类型等，这里指定的 Bearer 类型是最常见的一种类型。
- expires_in：用于指定 access_token 的有效时间，当超过这个有效时间，access_token 将会自动失效。
- refresh_token：其作用在于当 access_token 过期后，重新下发一个新的 access_token。
- scope：指定了可访问的权限范围，这里指定的是访问 Web 资源的“webclient”。

### 1.5 OAuth 2.0更新令牌

- 令牌的有效期到了，如果让用户重新走一遍上面的流程，再申请一个新的令牌，可能体验不好，而且也没有必要。OAuth 2.0允许用户自动更新令牌。
- B网站颁发令牌的时候，一次性颁发两个令牌，一个用于获取数据，另一个用于获取新的令牌（refresh token）。令牌到期后，用户使用refresh token发送一个请求，去更新令牌。

```http
https://b.com/oauth/token?
  grant_type=refresh_token&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET&
  refresh_token=REFRESH_TOKEN
```

> 上面的URL中，`grant_type`参数为`refresh_token`表示要求更新令牌，`client_id`参数和`client_secret`参数用于确认身份，`refresh_token`参数就是用于更新令牌的令牌。

- B网站验证通过后，就会颁发新的令牌了。

基于 OAuth2 协议的授权工作流程。整个流程如下图所示：

![oauth2-9](https://gitee.com/linbingxing/image/raw/master/oauth2-9.png)

我们可以把上述流程进一步展开梳理。

- 首先，客户端向用户请求授权，请求中一般包含资源的访问路径、对资源的操作类型等信息。如果用户同意授权，就会将这个授权返回给客户端。
- 现在，客户端已经获取了用户的授权信息，可以向授权服务器请求访问令牌。
- 接下来，授权服务器向客户端发放访问令牌，这样客户端就能携带访问令牌访问资源服务器上的资源。
- 最后，资源服务器获取访问令牌后会验证令牌的有效性和过期时间，并向客户端开放其需要访问的资源。

##  2 OAuth2 协议与微服务架构

对应到微服务系统中，**服务提供者充当的角色就是资源服务器，而服务消费者就是客户端**。所以每个服务本身既可以是客户端，也可以作为资源服务器，或者两者兼之。当客户端拿到 Token 之后，该 Token 就能在各个服务之间进行传递。如下图所示：

![oauth2-12](https://gitee.com/linbingxing/image/raw/master/oauth2-12.png)

在整个 OAuth2 协议中，最关键的问题就是如何获取客户端授权。就目前主流的微服架构来说，当我们发起 HTTP 请求时，关注的是如何通过 HTTP 协议透明而高效地传递令牌，此时授权码模式下通过回调地址进行授权管理的方式就不是很实用，密码模式反而更加简洁高效。

## 3 Spring Security OAuth2

### 3.1 介绍

- Spring Security OAuth2是对OAuth2的一种实现。
- OAuth2的服务提供涵盖了两种服务：授权服务（Authorization Server，认证和授权服务）和资源服务（Resource Server）。使用Spring Security OAuth2的时候可以将授权服务和资源服务放在同一个应用程序中，也可以选择建立使用同一个授权服务的多个资源服务。
- `授权服务`（Authorization Server）：包含对接入端以及登入用户的合法性进行验证并颁发token等功能，对令牌的请求端点由Spring MVC或Spring Webflux的控制器进行实现，下面是配置一个认证服务要实现的endpoints（端点）：
  - `AuthorizationEndpoint`用于认证请求。默认的URL是/oauth/authorize。
  - `TokenEndpoint`用于访问令牌的请求。默认的URL是/oauth/token。
- `资源服务`（Resource Server）:包含对资源的保护功能，对非法请求进行拦截，对请求中token进行解析鉴权等，下面的过滤器用于实现OAuth2的资源服务。
  - OAuth2AuthenticationProcessingFilter用来对请求给出的身份令牌解析鉴权。

### 3.2 构建授权服务器

#### 3.2.1 配置注解 @EnableAuthorizationServer

这个注解的作用在于**为微服务运行环境提供一个基于 OAuth2 协议的授权服务**，该授权服务会暴露一系列基于 RESTful 风格的端点（例如 /oauth/authorize 和 /oauth/token）供 OAuth2 授权流程使用。

```java
@Configuration
@EnableAuthorizationServer // 开启认证服务器
public class Oauth2ServerConfig extends AuthorizationServerConfigurerAdapter {

}
```

#### 3.2.2 继承AuthorizationServerConfigurerAdapter

```java
public class AuthorizationServerConfigurerAdapter implements AuthorizationServerConfigurer {
    public AuthorizationServerConfigurerAdapter() {
    }

    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        //配置令牌端点的安全性约束
    }

    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        //配置客户端详情服务
    }

    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        //配置令牌（token）的访问端点和令牌服务（token services）
    }
}
```

####  3.2.2 配置客户端详情服务

设置客户端时，用到的配置类是 ClientDetailsServiceConfigurer，该配置类用来配置客户端详情服务 ClientDetailsService。用于描述客户端详情的 ClientDetails 接口则包含了与安全性控制相关的多个重要方法，该接口中的部分方法定义如下：

```java
public interface ClientDetails extends Serializable {
    	//客户端唯一性 Id
    	String getClientId();

    	//客户端安全码,需要是信任的客户端。
    	String getClientSecret();

    	//客户端的访问范围,用来限制客户端的访问范围，如果为空（默认），那么客户端将拥有全部的访问范围
    	Set<String> getScope();

    	//客户端可以使用的授权模式
    	Set<String> getAuthorizedGrantTypes();
    
        //此客户端可以使用的权限（基于Spring Security authorities）
        Collection<GrantedAuthority> getAuthorities();
    	…
}
```

在授权服务器中存储客户端信息有两种方式

1. **基于内存级别的存储**
2. **通过 JDBC 在数据库中存储详情信息，使用JdbcClientDetailsService**
3. **实现ClientRegistrationService接口**

为了简单起见，这里使用了内存级别的存储方式。

```java
@Override
public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    //使用内存方式存储客户端详情
    clients.inMemory()
        .withClient("client_id") //client_id
        .secret(passwordEncoder.encode("secret")) //client_secret
        .resourceIds("product-id") //资源标识
        .authorizedGrantTypes("authorization_code", "password", "client_credentials", "implicit", "refresh_token") //该client允许的授权类型
        .scopes("read,write")//允许的授权范围
        .autoApprove(false)
        .resourceIds("https://www.baidu.com"); //回调地址
    super.configure(clients);
}
```

#### 3.3.3 令牌服务



## 参考资料

[OAuth 2.0 的四种方式](https://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)





