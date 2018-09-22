#### 背景介绍
  项目中使用了Spring Boot,由于是微服务架构,现在不想使用Session来存储用户登录状态 (session是基于cookie实现的)态,选择JWT来存储用户ID,作为身份认证的凭证.

#### JWT介绍(Json Web Token)
Json web token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准(RFC 7519).该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。
   
##### JWT构成   
    jwt由三部分组成:头部(header),载体(payload),签名(signature),如下所示
```    
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOiIxIiwibmJmIjoxNTE3OTEzMzY3LCJleHAiOjE1MTc5MTMzNzd9.9QTvNWuwJsma84AfeXK4JO9ozOy4owmZJws9IZ2DMAI
```

###### 头部(header)
    
```
{
    'typ':'JWT',
    'alg':'HS256',
}
```
    头部由两部分组成,
    typ:类型
    alg:加密方式
将头部进行Base64加密,得到
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```

###### 载荷(payload)
存储有效信息的地方,一般有三部分组成

1.标准中注册的声明(不强制使用)
```
iss: jwt签发者
sub: jwt所面向的用户
aud: 接收jwt的一方
exp: jwt的过期时间，这个过期时间必须要大于签发时间
nbf: 定义在什么时间之前，该jwt都是不可用的.
iat: jwt的签发时间
jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击
```

2.公共的声明

存放公共信息的地方

3.私有的声明

存放私有信息的地方

注意:载荷中不要存储敏感信息

如下为我使用的载荷
```
{
    'nbf':'Tue Feb 06 18:36:07 CST 2018',
    'exp':'Tue Feb 06 18:36:17 CST 2018',
    'userId':'1'
}
```
将载体进行Base64加密,得到
```
eyJ1c2VySWQiOiIxIiwibmJmIjoxNTE3OTEzMzY3LCJleHAiOjE1MTc5MTMzNzd9
```

###### 签名(signature)
将得到头部和载荷加密的字符串拼接起来,使用头部中alg指定的加密算法,加上secret盐加密,得到
```
9QTvNWuwJsma84AfeXK4JO9ozOy4owmZJws9IZ2DMAI
```
##### 最后将三部分使用'.'拼接起来即可

##### 使用方法
当用户登录通过验证,返回一个jwt给前端,前端在请求头中添加一个'Authorization'字段,每次请求
带上此请求头,后端根据userId判断用户身份

#### 具体实现
##### 项目使用的jjwt,在pom.xml中引入如下依赖
```
<!-- jwt -->
<dependency>
	<groupId>io.jsonwebtoken</groupId>
	<artifactId>jjwt</artifactId>
	<version>0.7.0</version>
	<scope>compile</scope>
</dependency>
```
##### 项目结构如下
```
jwt
 |_ excption
         |_JwtParameterEmptyException.java
         |_JwtParameterIllegalException.java
 |_jwtInfo.java
 |_JwtPayload.java
 |_JwtUtils.java

```

其中exception包下是自定义异常

JwtInfo.java代码如下
```
package com.liao.common.jwt;

/**
 * Jwt信息(生成Jwt时所需要的参数)
 *
 * @author hongyangliao
 * @ClassName: JwtInfo
 * @Date 17-12-28 下午3:42
 */
public class JwtInfo extends JwtPayload {
    /**
     * 签名的加密方式
     * 一般取值: HS256
     */
    private String alg;

    /**
     * 签名的盐
     */
    private String secret;

    /**
     * 过期时间 单位毫秒
     */
    private long expiresMillis;

    /**
     * 生成的token信息
     */
    private String token;

    public String getAlg() {
        return alg;
    }

    public void setAlg(String alg) {
        this.alg = alg;
    }

    public String getSecret() {
        return secret;
    }

    public void setSecret(String secret) {
        this.secret = secret;
    }

    public long getExpiresMillis() {
        return expiresMillis;
    }

    public void setExpiresMillis(long expiresMillis) {
        this.expiresMillis = expiresMillis;
    }

    public String getToken() {
        return token;
    }

    public void setToken(String token) {
        this.token = token;
    }

    @Override
    public String toString() {
        return "JwtInfo{" +
                "alg='" + alg + '\'' +
                ", secret='" + secret + '\'' +
                ", expiresMillis=" + expiresMillis +
                ", token='" + token + '\'' +
                '}';
    }
}

```
JwtPayload.java代码如下
```
package com.liao.common.jwt;

import java.util.Date;

/***
 * Jwt载荷
 *
 * @ClassName: JwtPayload
 * @author hongyangliao
 * @Date 17-12-28 下午2:34
 */
public class JwtPayload {
    /**
     * 用户id
     */
    private String userId;

    /**
     * 过期时间
     */
    private Date exp;

    /**
     * 在什么时间之前不能使用
     */
    private Date nbf;

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public Date getExp() {
        return exp;
    }

    public void setExp(Date exp) {
        this.exp = exp;
    }

    public Date getNbf() {
        return nbf;
    }

    public void setNbf(Date nbf) {
        this.nbf = nbf;
    }

    @Override
    public String toString() {
        return "JwtPayload{" +
                "userId='" + userId + '\'' +
                ", exp=" + exp +
                ", nbf=" + nbf +
                '}';
    }
}
```

JwtUtils.java代码如下
```
package com.liao.common.jwt;

import com.liao.common.jwt.exception.JwtParameterEmptyException;
import com.liao.common.jwt.exception.JwtParameterIllegalException;
import io.jsonwebtoken.*;
import org.apache.commons.lang3.StringUtils;

import java.util.Date;


/***
 * Jwt工具类(使用jjwt)
 *
 * @ClassName: JwtUtils
 * @author hongyangliao
 * @Date 17-12-28 下午6:16
 */
public class JwtUtils {
    /***
     * 生成Jwt
     *
     * @Title: generateJwt
     * @author: hongyangliao
     * @Date: 17-12-28 下午4:38
     * @param jwtInfo jwt信息 需要传入 alg,userId,secret,expiresMillis
     * @return java.lang.String
     * @throws
     */
    public static String generateJwt(JwtInfo jwtInfo) throws JwtParameterEmptyException, JwtParameterIllegalException, SignatureException {
        if (jwtInfo == null) {
            throw new JwtParameterEmptyException("Jwt传入参数为空");
        }

        //参数校验
        String alg = jwtInfo.getAlg();
        String userId = jwtInfo.getUserId();
        String secret = jwtInfo.getSecret();
        long expiresMillis = jwtInfo.getExpiresMillis();

        if (StringUtils.isBlank(alg) || StringUtils.isBlank(userId) || StringUtils.isBlank(secret)) {
            throw new JwtParameterEmptyException("Jwt传入参数为空");
        }

        if (expiresMillis < 0) {
            throw new JwtParameterIllegalException("过期时间不合法");
        }

        Date currentDate = new Date();
        long dateMillis = currentDate.getTime() + expiresMillis;
        Date date = new Date(dateMillis);

        //如果jwtInfo.getAlg()不正确会抛出SignatureException
        SignatureAlgorithm signatureAlgorithm = null;
        try {
            signatureAlgorithm = SignatureAlgorithm.forName(jwtInfo.getAlg());
        } catch (SignatureException e) {
            throw new SignatureException("(alg)签名加密方式不正确");
        }

        String token = null;
        if (signatureAlgorithm != null) {
            //设置头部,载体,签名
            token = Jwts.builder()
                    .setHeaderParam("typ", "JWT")
                    .setHeaderParam("alg", jwtInfo.getAlg())
                    .claim("userId", jwtInfo.getUserId())
                    .setNotBefore(currentDate)
                    .setExpiration(date)
                    .signWith(signatureAlgorithm, secret)
                    .compact();
        }
        return token;
    }

    /***
     *  根据信息获取Jwt载体
     *
     * @Title: getJwtPayload
     * @author: hongyangliao
     * @Date: 17-12-28 下午4:59
     * @param jwtInfo jwt信息, 需要传入token,secret
     * @return com.ducetech.common.jwt.JwtPayload Jwt载体
     * @throws
     */
    public static JwtPayload getJwtPayload(JwtInfo jwtInfo)
            throws JwtParameterEmptyException, ExpiredJwtException,
            UnsupportedJwtException, MalformedJwtException,
            SignatureException, IllegalArgumentException {
        //参数校验
        if (jwtInfo == null) {
            throw new JwtParameterEmptyException("Jwt传入参数为空");
        }

        String secret = jwtInfo.getSecret();
        String token = jwtInfo.getToken();

        if (StringUtils.isBlank(secret) || StringUtils.isBlank(token)) {
            throw new JwtParameterEmptyException("Jwt传入参数为空");
        }

        Jws<Claims> jws = null;
        try {
            jws = Jwts.parser().setSigningKey(jwtInfo.getSecret()).parseClaimsJws(jwtInfo.getToken());
        } catch (ExpiredJwtException ee) {
            //token已过期
            throw ee;
        } catch (UnsupportedJwtException ujw) {
            //签名算法不支持
            throw ujw;
        } catch (MalformedJwtException mje) {
            //失去载体
            throw mje;
        } catch (SignatureException se) {
            //jwt签名不匹配
            throw se;
        } catch (IllegalArgumentException iae) {
            //不能序列化载体
            throw iae;
        }

        JwtPayload jwtPayload = new JwtPayload();
        if (jws != null) {
            Claims claims = jws.getBody();
            String userId = claims.get("userId", String.class);
            jwtPayload.setUserId(userId);
            jwtPayload.setExp(claims.getExpiration());
            jwtPayload.setNbf(claims.getNotBefore());
        }
        return jwtPayload;
    }

    public static void main(String[] args) {
        JwtInfo jwtInfo = new JwtInfo();
        jwtInfo.setAlg("HS256");
        jwtInfo.setUserId("1");
        jwtInfo.setSecret("hongyangliao");
        jwtInfo.setExpiresMillis(10000L);
        String jwt = generateJwt(jwtInfo);
        System.out.println(jwt);
        jwtInfo.setToken(jwt);
        JwtPayload jwtPayload = getJwtPayload(jwtInfo);
        System.out.println(jwtPayload);
    }
}
```

#### 参考

 [什么是 JWT -- JSON WEB TOKEN](https://www.jianshu.com/p/576dbf44b2ae)



