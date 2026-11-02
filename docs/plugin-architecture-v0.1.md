# Learning & Recording 插件架构 v0.1

> 当前方案：先做一个只包含两个 Skill 的轻量插件，在 ChatGPT Chat 中验证实际调用。本文记录架构选择，不代表插件已经打包或安装。

## 为什么选择 Plugin

独立 Skill 的可用范围会受到 ChatGPT 账户资格和产品界面的影响。ChatGPT 官方文档区分独立 Skill 与插件内 Skill：插件可以把 Skill 打包为可安装工作流，内置 Skill 可以在支持的 ChatGPT Chat / Work 界面使用。

此前尝试直接上传 Learning Skill，未能确认新对话是否真正调用了它。因此那次对话只能作为普通回答样本，不能作为 Learning Skill 的有效回归测试。下一次测试应在完整插件安装后进行。

## 初期结构

```text
learning-recording/
├── plugin.json
└── skills/
    ├── learning/
    │   ├── SKILL.md
    │   ├── agents/
    │   │   └── openai.yaml
    │   └── references/
    │       └── examples.md
    └── recording/
        ├── SKILL.md
        ├── agents/
        │   └── openai.yaml
        └── references/
            └── examples.md
```

两个 Skill 分开触发、分别承担职责：

- **Learning**：与用户共同研究当前问题。
- **Recording**：根据真实学习过程撰写和保存记录。

两者可以配合，但 Recording 不会在每次 Learning 结束后自动运行。是否记录由用户主动决定。

## 先不加入的组件

- **不自建 MCP Server**：当前核心工作是指导学习和记录，已有模型能力足够完成主要任务。
- **不重新实现 Memory**：用户偏好和跨会话上下文使用 ChatGPT 已有能力。
- **不立即建设 RAG**：具体历史资料先使用现有 Notion / GitHub 检索能力。只有普通检索反复找不到需要的内容，且检索质量确实影响任务时，再评估语义检索或 RAG。
- **不自动修改 Skill**：真实使用后发现有代表性的问题或成功案例时，用户可以主动提出复盘；讨论确认后再更新规则或正反例，并通过 Git 记录版本。

## 调用与外部工具

- 两个 Skill 都设置为显式调用，避免普通对话自动介入。
- Notion、Write Like Me 等能力尽量复用用户已经连接或安装的现有工具；不在初版中重复搭建服务。
- 若插件打包时需要声明某个现有 App 依赖，必须使用已验证的 App ID，并先检查授权与平台限制，不猜测标识。
- 如果某项外部能力在当前 ChatGPT 界面不可用，Skill 应说明限制并提供可行的草稿或替代流程，不声称已完成外部写入。

## 打包与验证

- 一个插件包含两个 Skill，目录名和 Skill frontmatter 保持一致。
- 核心指令放在各自的 `SKILL.md`；较长的正反案例放在对应 `references/`，按需读取。
- 在插件配置中关闭隐式调用，两个 Skill 都由用户主动选择。
- 插件安装后分别测试 Learning 与 Recording，并检查实际调用、外部工具权限和写入结果。
- 测试通过前不称为正式版本；GitHub 是源文件和版本历史的唯一事实来源。

## 版本推进

1. 完成 Recording 的行为映射、Skill 和案例。
2. 检查两个 Skill 的目录、元数据和插件清单。
3. 打包为一个 skills-only plugin 并安装。
4. 在 ChatGPT Chat 中测试两个 Skill 的显式调用。
5. 根据真实表现讨论修改，不做每次使用后的强制评估。
