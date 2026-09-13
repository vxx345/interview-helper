# interview-helper

大厂后端与 AI Agent 方向面试题的**标准化答题教练** Skill，可用于 WorkBuddy / CodeBuddy 等支持 Skill 的 AI 编程助手。

把任意一道面试题，加工成三件东西：

- **怎么想**（思路脚手架 / CoT 推理链）
- **说什么**（大厂标准答案，分层深答）
- **怎么说**（能直接开口念的口述稿）

## 核心能力

- **题型判定**：先判断要不要绑项目（客观题纯通用回答，禁止硬挂项目），再决定打多深。
- **深度分级**：`[简答] / [标准] / [深挖] / [深潜]` 四档，最高档对标大厂面委级深度（六维分层、论文级引用、量化对比、生产实践、升华总结）。
- **交互模式**：`练:`（教练模式，引导思考）、`mock:`（模拟面试）、`拷打:`（高压追问复盘）。
- **覆盖方向**：
  - 后端：Java / JVM / 并发 / MySQL / Redis / Kafka / Spring / 分布式 / 网络
  - Agent：RAG / MCP / Multi-Agent / 上下文工程 / Function Calling / 工具调用容错

## 目录结构

```
interview-helper/
├── SKILL.md                      # 技能主入口与执行流程
└── references/                   # 分主题参考资料
    ├── agent-backend-engineering.md
    ├── agent-interview-points.md
    ├── answer-frameworks.md
    ├── backend-interview-points.md
    ├── deep-dive-framework.md
    ├── reverse-qa.md
    └── spoken-script-craft.md
```

## 安装方式

将本仓库拷贝到 Skill 目录后，助手即可在对话中自动匹配并加载：

- **用户级**（所有项目可用）：`~/.workbuddy/skills/interview-helper/`
- **项目级**（仅当前项目）：`<你的项目>/.workbuddy/skills/interview-helper/`

```bash
# 以用户级为例
git clone https://github.com/vxx345/interview-helper.git ~/.workbuddy/skills/interview-helper
```

## 使用方式

在对话中直接抛出面试题即可，例如：

- `帮我答一下：MySQL 为什么用 B+ 树而不是 B 树？`
- `mock: 来一道 Redis 缓存击穿的题`
- `拷打: 我刚讲的是 RAG，你追问我`

技能会根据题型自动选择「纯通用 / 绑项目」的组织方式，并按你指定的深度档位输出。

## 许可证

[MIT](./LICENSE)
