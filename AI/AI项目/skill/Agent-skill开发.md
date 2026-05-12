


## skill结构&组成

* 元数据层：识别与触发
	* 对外暴露的特征信号，用于被Agent的意图识别系统扫描和匹配
	* 接收到用户表达意图时，agent并非搜索工具，而是领域识别，将所有skill的名片（名称-描述）载入记忆





一、创建目录
1. .opencode/skill


二、创建技能文件夹

1. 创建 【自动生成周报】的文件夹：weekly-report-generator

三、创建Skill.md文件
1. 在weekly-report-generator/skillmd

``` yaml
---​

name: weekly-report-generator​

description: 自动生成项目周报，从 git 提交、issue、todo 文件和用户输入中提取信息，生成专业的 HTML/PDF 周报​

---
```


四、指令编写
```markdown
​

代码块​

# 指令:​

你是一个项目周报生成助手。你的任务是从多种数据源收集本周的项目进展，并生成结构化的周报。​

- 风险与问题​

​

4. **生成报告**​

- 参考 references/template-filling.md 了解模板填充逻辑​

- 使用 assets/report-template.html 作为模板​

- 将结构化数据填充到模板中​

- 生成 HTML 格式的周报文件：weekly-report-YYYY-MM-DD.html​

- 使用 scripts/html-to-pdf.py 将 HTML 转换为 PDF​

- 生成 PDF 格式的周报文件：weekly-report-YYYY-MM-DD.pdf​

​

## 周报内容说明​

​

用户需要提供以下内容（支持三种方式）：​

​

**方式一：在项目中提供文件（推荐）**​

- **本周遇到的问题**：在项目根目录创建 `problems.md` 或 `problems.json` 文件​

- **本周个人成长**：在项目根目录创建 `growth.md` 或 `growth.json` 文件​

- **相关知识分享**：在项目根目录创建 `knowledge.md` 或 `knowledge.json` 文件​

​

JSON 格式需符合 user-input.json 中的结构定义。Markdown 格式需包含相应的标题和内容。​

​

**方式二：通过 user-input.json 文件提供**​

- 在 user-input.json 中定义 problems、growth、knowledge 字段​

​

**方式三：对话提供**​

- 如果项目中没有提供文件，按规则询问用户输入​

​

内容说明：​

- **本周遇到的问题**：开发过程中遇到的技术难题、阻塞问题等，包含问题描述、类型、解决方案、经验教训​

- **本周个人成长**：学到的技术、能力提升、经验总结，包含类别、内容、影响​

- **相关知识分享**：值得记录的技术知识点、最佳实践、学习资源，包含标题、内容、资源链接​

​

## 使用说明​

​

用户可以直接说："帮我生成本周的周报"，或者提供具体的时间范围。​

​

## 注意事项​

​

- 如果项目不是 git 仓库，跳过 git log 分析
- 如果没有 issue 或 todo 文件，提醒用户手动输入关键进展​
- 需要安装 Chrome 或使用浏览器手动生成 PDF​
- 同时输出 HTML 和 PDF 两种格式
```


五、为这个skill提供资源

* assets
* references
* scripts
* skill.md



六、测试skill

1. 人为测试，就是让这个skill跑一次