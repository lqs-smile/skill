




## 跨域

问题：
1. 浏览器的同源策略导致无法访问跨源访问接口
2. 域名写错：`https://test-api.kbjcn.com//kbj-rx/prescriptionManagement/maintenance/prescriptionList`:多了一个/

解决：
* 前端：使用本地服务器正向代理
* 后端：设置cors

原理：
1. 通过本地服务器使用域名去请求外源接口，
2. 再通过正则将前端项目的域名替换成本地，也就是本地域名请求本地服务器，本地服务器去请求外源域名