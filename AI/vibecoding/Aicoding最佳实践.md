



> 这里面还缺少最重要的一条，所有模型写出来的code，全部都要review，全部都要理顺，既要让模型辅助也需要向他学习。

1. 清晰规划比盲目 “让 AI 自由发挥” 更重要。
2. “Planning is everything” ——不要让 AI 自己随意规划整个项目，否则代码会混乱。
3. 最开始要做一个 Game Design Document（GDD，或者如果是应用的话，就是产品需求文档 PRD），以 Markdown 格式写清你的构想。
4. 之后要让 AI 基于这个设计文档 +技术选型，生成一个 实现计划（implementation plan），而不是直接让 AI开始写代码。
5. 实现计划里的每一步都应该是小粒度，并且附带测试，这样每次 AI 写出的功能都能被验证。
6. 维持上下文一致性：用 Memory Bank（记忆库）
7. 建议创建一个 memory-bank 文件夹，把 GDD、tech-stack、implementation plan、progress、architecture 等重要文档都放进去。
8. AI 在生成代码时 “总是” 读取关键规则 /文档（例如 architecture.md, game-design-document.md），以保证它写出来的东西是基于你当前的整体结构，而不是零散乱写。
9. 你还应该在 progress.md 中记录每一步完成情况，在 architecture.md 中补充每个文件或者模块的架构解释。这样未来回顾或让 AI 继续开发时，会更清晰。
10. 迭代 + 验证 + 提交
    
    - 用 AI 写第一步（实现计划里 Step 1）之后，不要马上继续下一步，而是让你自己运行测试：确认 AI 写的代码是否满足预期。
    - 每完成一个 step，就 commit 一次。这样可以保留历史，也便于后退／修正。
    - 每一步都开启新的对话（新的 Chat /新上下文）让 AI “重新读 memory-bank + progress 再继续下一步”。这种方式能避免上下文混乱。
    
11. 为新特性写 feature-specific 文档
    
    - 在基础框架（base game / app）完成后，想加新功能（特效、声音、UI …）时，不要直接命令 AI 写代码，而是为每个大功能写一个 feature-implementation.md：列出小步骤 +测试。
    - 然后让 AI 逐步实现这些 feature，保持明确、模块化、可测试。
    
12. 错误处理 & 卡住时的方法
    
    - 如果 AI 生成功能出错，用 Claude Code 的 /rewind 回到上一步重新尝试。
    - 对于 JavaScript 错误，建议把控制台（console）日志／错误复制到 VSCode，让 AI 帮你分析。
    - 如果问题很复杂、卡住了，可以把整个 repo 做成一个大文件（用类似 RepoPrompt / uithub 的方式），然后请 AI 从整体视图帮你诊断。
    
13. 优化 AI 工具使用
    
    - 对于小改动（refactor /小调整等），建议使用较小 /中等能力的模型（如 GPT-5 medium）进行，以节省成本，同时保持响应质量。
    - 配合使用 CLI 和 VSCode：既可以在命令行里运行 Codex CLI / Claude Code 来看 diff，又可以通过 VSCode 插件维持开发节奏。
    - 为 Claude Code 或 Codex CLI 自定义命令，比如 /explain $arguments：先让模型理解某个模块 /变量 /逻辑，然后再让它基于理解做任务，这样能提升生成质量。
    - 频繁清除对话上下文（如 /clear 或 /compact），避免旧对话内容影响新的 prompt。
    
14. 风险意识与权衡
    
    - 虽然 vibe coding 鼓励快速产出，但这种方式有潜在风险：AI 写出的代码可能结构混乱、未来维护困难。社区里有人提到 “代码混乱到调试噩梦”。
    - 有人指出 AI 写出的逻辑有 bug（如并发问题、不正确的 API 调用等），这些 bug 很难被察觉，因为代码“看起来对”。
    - 如果项目到后期进入生产阶段（或用户较多时），最好考虑重构（vibe-refactor）：有人在社区里专门提供这种服务，把用 AI 快速写出的 “原型 / β 版本” 变得更健壮。
    - 保持适度的审查机制：虽然是 vibe coding，但定期审查代码、做重构、建立测试习惯非常重要。
    
15. 持续反馈与学习
    
    - 每次迭代完成后，不仅记录 progress，还记录 architecture 的变动和思考，这样下次生成代码时 AI 有 “记忆” 可用。
    - 如果你卡住了，或者某些 prompt /策略不成功，可以向社区求助（例如 Reddit 的 r/vibecoding）。很多人都在分享他们失败 +成功的经验。
    - 建议保持小步快跑 — 用 AI 快速原型验证想法，不要一次把所有功能堆进去。发现方向对了再慢慢加。
    
16. 综合心得
    
    - vibe coding 是一个强大的快速原型工具：它可以让你很迅速地把想法验证出来。但它不应该取代所有传统的软件工程流程，尤其是当你追求长期维护或扩大规模时。
    - 上下文管理非常关键：记忆库（memory-bank） + 明确规则（Always read architecture / GDD）是维持项目健康的重要支撑。
    - 测试不可省略：每一步有测试、每个 feature 都拆开实现并验证，是保证生成代码可用性的关键。
    - 灵活结合 AI 与人类判断：AI 写的东西非常有用，但人类需要持续审查、校正、重构。
    - 社区很有参考价值：阅读其他 vibe coder 的经验（比如他们卡住了什么、重构怎么做）对自己的实践非常有帮助。