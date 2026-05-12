越来越多的公司会创建一个中心服务器，服务于各种产品线，而cookie只能在web端起作用，移动端和桌面端不支持，所以就得整一个聚合验证

![305](assets/jwt/file-20260506101826378.png)


* 正常情况下，不同的生产线，都会有自己的服务器，产品内部的数据一般与自己的服务器交互
* 但数据中心服务器仍然有存在的必要，因为一家公司总会存在可以共享的数据，例如：用户数据


## 设备与中心服务器通讯

进行http通讯，中心服务器至少承担认证与授权的功能，例如登录，这种情况下传统的cookie还能继续使用吗，也是可以的，cookie无法就是会自动发送的请求头，但浏览器之外的设备需要手动模拟cookie的功能

JWT的出现就是为了解决这个问题



## JWT

JSON WEB TOKEN , 提供为多种多端设备提供统一的、安全的令牌格式

![360](assets/jwt/file-20260506102852658.png)


### 存储位置
 没有限制，可以是cookie或者localstorage


### 传输
放到消息头里,这个就是看和服务端的约定了

```http
HTTP/1.1 200 OK
...
set-cookie:token=jwt令牌
authorization:jwt令牌
...
{..., token:jwt令牌}
```


### 客户端拿到令牌后

存储它：
* 任何位置，手机文件、pc文件、localstorage、cookie
使用它
* 后续发送的每一个请求都要携带token,没有明确规定要怎么携带，但通常是这么放，这是OAUTH2附带token的一种规范
* ```http
  GET /api/resources HTTP/1.1
	...
	authorization: bearer jwt令牌
	...
  ```




## 令牌的组成

三部分组成
1. header：令牌头部：记录令牌的类型和算法
2. payload：令牌负荷：记录主体信息，jwt解析后的所有内容都可以放这里
3. signature：令牌签名：按照头部的算法类型，对整个令牌进行签名，目的是让这个令牌不得被伪造和篡改

他们的组成格式为：header.payload.signature

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJpYXQiOjE1ODc1NDgyMTV9.BCwUy3jnUQ_E6TqCayc7rCHkx-vxxdagUwPOWqwYCFc
```



### header

* 令牌的类型
* 签名算法

```json
{
  "alg":"HS256",
  "typ":"JWT"
}
```


该对象记录了：

- alg：signature部分使用的签名算法，通常可以取两个值
    
    - HS256：一种对称加密算法，使用同一个秘钥对signature加密解密
        
    - RS256：一种非对称加密算法，使用私钥加密，公钥解密
        
- typ：整个令牌的类型，固定写`JWT`即可
    

设置好了`header`之后，就可以生成`header`部分了


### 生成header部分

具体的生成方式及其简单，就是把`header`部分使用`base64 url`编码即可

> `base64 url`不是一个加密算法，而是一种编码方式，它是在`base64`算法的基础上对`+`、`=`、`/`三个字符做出特殊处理的算法
> 而`base64`是使用64个可打印字符来表示一个二进制数据，具体的做法参考[百度百科](https://baike.baidu.com/item/base64/8545775?fr=aladdin)



浏览器提供了`btoa`函数，可以完成这个操作：

```js
window.btoa(JSON.stringify({  
  "alg":"HS256",  
  "typ":"JWT"  
}))  
// 得到字符串：eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

同样的，浏览器也提供了`atob`函数，可以对其进行解码：

```js
window.atob("eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9")  
// 得到字符串：{"alg":"HS256","typ":"JWT"}
```

> nodejs中没有提供这两个函数，可以安装第三方库`atob`和`bota`搞定
> 或者，手动搞定

## payload

这部分是jwt的主体信息，它仍然是一个JSON对象，它可以包含以下内容：

```json
{  
  "ss"："发行者",  
  "iat"："发布时间",  
  "exp"："到期时间",  
  "sub"："主题",  
  "aud"："听众",  
  "nbf"："在此之前不可用",  
  "jti"："JWT ID"  
}
```

以上属性可以全写，也可以一个都不写，它只是一个规范，就算写了，也需要你在将来验证这个jwt令牌时手动处理才能发挥作用

上述属性表达的含义分别是：

- ss：发行该jwt的是谁，可以写公司名字，也可以写服务名称
    
- iat：该jwt的发放时间，通常写当前时间的时间戳
    
- exp：该jwt的到期时间，通常写时间戳
    
- sub：该jwt是用于干嘛的
    
- aud：该jwt是发放给哪个终端的，可以是终端类型，也可以是用户名称，随意一点
    
- nbf：一个时间点，在该时间点到达之前，这个令牌是不可用的
    
- jti：jwt的唯一编号，设置此项的目的，主要是为了防止重放攻击（重放攻击是在某些场景下，用户使用之前的令牌发送到服务器，被服务器正确的识别，从而导致不可预期的行为发生）
    

可是到现在，看了半天，没有出现我想要写入的数据啊😂

当用户登陆成功之后，我可能需要把用户的一些信息写入到jwt令牌中，比如用户id、账号等等（密码就算了😳）

其实很简单，payload这一部分只是一个json对象而已，你可以向对象中加入任何想要加入的信息

比如，下面的json对象仍然是一个有效的payload

```json
{  
  "foo":"bar",  
  "iat":1587548215  
}
```

`foo: bar`是我们自定义的信息，`iat: 1587548215`是jwt规范中的信息

最终，payload部分和header一样，需要通过`base64 url`编码得到：

```js
window.btoa(JSON.stringify({  
  "foo":"bar",  
  "iat":1587548215  
}))  
// 得到字符串：eyJmb28iOiJiYXIiLCJpYXQiOjE1ODc1NDgyMTV9
```



## signature

这是JWT的签名，保证jwt不被篡改

根据header要求的签名算法对header和payload的编码结果进行加密

例如：
* 头部要求算法：hs256
* 前两部分的编码结果：eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJpYXQiOjE1ODc1NDgyMTV9

加密：
```js
HS256(`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJpYXQiOjE1ODc1NDgyMTV9`, "keyStr")

// 得到：BCwUy3jnUQ_E6TqCayc7rCHkx-vxxdagUwPOWqwYCFc
```

讲它们三个组合在一起得到

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmb28iOiJiYXIiLCJpYXQiOjE1ODc1NDgyMTV9.BCwUy3jnUQ_E6TqCayc7rCHkx-vxxdagUwPOWqwYCFc
```

安全：
1. 由于密钥保存在服务器端，所以这个信息是无法被解开
2. 不要将类似密码之类的敏感信息放到里面，因为可以被明文看到



如何保证不被篡改？

获取到jwt后，讲payload部分base64Url解码后，修改数据再进行编码，传给服务器，如何得知消息体被修改过了？

令牌的验证？

将前两个部分使用加密算法加密后与第三部分进行对比，然后再进行后续的校验




## 注意

* jwt加密都是无法还原的 -- hash签名，而不是 Encryption(加密)

## 总结

- jwt本质上是一种令牌格式。它和终端设备无关，同样和服务器无关，甚至与如何传输无关，它只是规范了令牌的格式而已

- jwt由三部分组成：header、payload、signature。主体信息在payload

- jwt难以被篡改和伪造。这是因为有第三部分的签名存在。