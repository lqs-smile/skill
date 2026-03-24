

![](file:///C:/Users/ROG/Documents/WXWork/1688855351514530/Cache/Image/2026-01/企业微信截图_17690491331068.png)![](file:///C:/Users/ROG/Documents/WXWork/1688855351514530/Cache/Image/2026-01/企业微信截图_17690491815023.png)， 聚合项目遇到双滚动条的，检查下高度和挂载容器id就好，  悬浮滚动条丢失的是挂载容器没配置


1. 双滚动条1：人工滚动条，它不知道什么时候该隐藏，需要给他传入container，以及高度
2. 双滚动条2：给table组件设置了overflow：auto
3. 悬浮条丢失：挂在容器没设置