## 核心原理

浏览器允许 `<script>`、`<img>`、`<link>` 等标签加载跨域资源，JSONP 就是利用这一点：


1. 前端创建 `<script src="...?callback=handleData">`
    
2. 浏览器向跨域服务器发起 GET 请求
    
3. 服务器返回 `handleData({"key":"value"})`
    
4. 浏览器执行这段 JS，自动调用全局 `handleData` 函数，拿到数据
    
    

## 实现方式 

前端通过jsonp发送请求

```javascript
// 1. 定义回调函数
function handleResponse(data) {
    console.log('收到数据:', data);
}

// 2. 动态创建 script 标签
function jsonp(url, callbackName) {
    const script = document.createElement('script');
    // 将回调函数名附加到 URL
    script.src = `${url}?callback=${callbackName}`;
    document.head.appendChild(script);
    
    // 加载完成后移除 script 标签
    script.onload = function() {
        document.head.removeChild(script);
    };
}

// 调用
jsonp('https://api.example.com/data', 'handleResponse');
```

后端处理 - node/express

```js

app.get('/data', (req, res) => {
    const callback = req.query.callback;  // "handleResponse"
    const data = { id: 1, name: '张三' };
    
    // 返回函数调用字符串
    res.send(`${callback}(${JSON.stringify(data)})`);
    // 实际输出: handleResponse({"id":1,"name":"张三"}) 
    // handleResponse会被调用,所以需要实现在前端声明
});
```


## 关键点

| 对比               | JSONP 返回                 | 普通 JSON 返回         |
| :--------------- | :----------------------- | :----------------- |
| **Content-Type** | `application/javascript` | `application/json` |
| **内容**           | `函数名(数据)`                | `{"key": "value"}` |
| **执行**           | 浏览器自动执行                  | 需要 `JSON.parse()`  |
| **跨域**           | ✅ `<script>` 天然支持        | ❌ 受同源策略限制          |

所以 JSONP 的"P"（Padding）指的就是**用函数名包裹 JSON 数据**，服务端返回的是"填充"后的函数调用语句。



## JSONP 的局限性（重要）

| 限制               | 说明                                                |
| ---------------- | ------------------------------------------------- |
| **仅支持 GET 请求**   | 只能通过 URL 传参，无法处理 POST/PUT/DELETE                  |
| **需要后端配合**       | 打乱后端响应支持 JSONP 协议（解析 callback 参数并返回 JS 代码）        |
| **存在安全风险**       | 请求的脚本会以当前页面的权限执行，若第三方服务被劫持，可能注入恶意代码               |
| **错误处理不完善**      | `<script>` 加载失败难以精确捕获（只能靠 onerror，且无法区分 HTTP 状态码） |
| **只能处理 JSON 数据** | 返回内容必须是可执行的 JavaScript                            |
