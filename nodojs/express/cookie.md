

## 1. 问题根源：HTTP 无状态

- **无状态**：服务器不保存任何请求历史，每次请求都是"陌生人"
- **后果**：登录后刷新页面，服务器又"不认识"你了
- **比喻**：像患有严重脸盲症的柜员，每次都要重新验证身份

---

## 2. 解决方案：Cookie（浏览器"卡包"）

Cookie 是浏览器内置的**身份卡包**，自动管理各网站的"出入证"

### 卡包四大功能
| 功能 | 说明 |
|------|------|
| 存放多证 | 支持多个网站、多个路径的证件 |
| 自动出示 | 访问对应网站时自动附带 |
| 精准匹配 | 不会把肯德基的证发给麦当劳 |
| 过期清理 | 自动淘汰过期证件 |

---

## 3. Cookie 的"身份证"结构



---

## 4. 发送规则（四条件同时满足）

1. **未过期** — 证件在有效期内
2. **域匹配** — cookie.domain 包含请求域名（如 `yuanjin.tech` 匹配 `www.yuanjin.tech`）
3. **路径匹配** — cookie.path 包含请求路径（如 `/news` 匹配 `/news/detail`）
4. **安全验证** — secure=true 时仅 HTTPS 发送

> 满足条件的 cookie 自动加入请求头：`Cookie: token=xxx; key2=val2`

---

## 5. 设置方式

### 服务器设置（响应头）
```http
Set-Cookie: token=123456; path=/; max-age=3600; httponly

```



### 客户端设置
```js
document.cookie = "key=value; path=/; domain=yuanjin.tech"
```


## 6. 关键属性速查


| 属性         | 作用              | 默认值                      |
| :--------- | :-------------- | :----------------------- |
| `path`     | 有效路径范围          | 当前请求路径（服务端）/ 当前页面路径（客户端） |
| `domain`   | 所属域名            | 当前请求域                    |
| `max-age`  | 相对有效期（秒）        | 会话结束即过期                  |
| `expire`   | 绝对过期时间（GMT）     | —                        |
| `secure`   | 仅 HTTPS 传输      | false                    |
| `httponly` | 禁止 JS 读取（防 XSS） | false                    |

---

## 7. 删除 Cookie

**本质：修改一个已过期的同名 cookie**


```http
Set-Cookie: token=; domain=yuanjin.tech; path=/; max-age=-1
```

---

## 8. 登录场景完整流程


```plain
【登录阶段】
浏览器 → POST /login (账号+密码)
服务器 ← 验证通过 → Set-Cookie: token=xxx
浏览器   自动保存到卡包

【后续请求】
浏览器 → GET /add-admin (自动附带 Cookie: token=xxx)
服务器 ← 验证 token 有效 → 执行操作
```

---

## 9. 核心安全警示

> **Cookie 包含身份信息，切勿泄露！** 他人获取后可冒用你的身份操作。

表格

| 防护手段       | 作用              |
| :--------- | :-------------- |
| `httponly` | 防止 XSS 脚本窃取     |
| `secure`   | 防止 HTTP 明文传输被截获 |
| `SameSite` | 防止 CSRF 跨站伪造请求  |
