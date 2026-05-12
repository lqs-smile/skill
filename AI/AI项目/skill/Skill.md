


## 什么是skill

技能包，你让ai做一件事，他可能会做，但是不够牛逼，使用技能包后，他会变得很专业，所有基本上所有的动作都可以设计一个skill，让ai做这件事时更加专业





## 使用skill

1. 根据提示词，系统会读取所有的skill的关键字来判断使用哪款skill
	1. 安装功能相同的skill应该如何正确触发
2. 通过斜杠命令指定skill



## skill作用域
目录下的skills和skill都能识别，根据skill的功能的通用性来决定是放到全局还是项目中

* 项目
	* admin/OpenCode/skills/-ppt-dev
	* admin/ClaudeCode/skills/pdf
* 全局
	* 你的用户名/.config/ClaudeCode


## skill源码解析



## 开发skill


官方文档：



### 魔法skill

skill名字：

用于创建skill的skill

根据你的想法，来给你生成skill框架


## 什么情况下需要封装流程作为skill

适合：
1. 模板化
2. 固定流程
3. 确定性高的
4. 知识密集型
5. 复杂繁琐工作流
6. 复用


不适合：

1. 抽象
2. 即兴聊天
3. 创意输出




## AgentSkill核心机制解析

用户的问题 =>  先到skills下匹配各个skill的元数据 => 将skill的指令部分加载好后传递给大模型

1. 全面实行渐进式披露的加载原则





