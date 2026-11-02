# Learning & Recording 插件架构 v0.1

> 当前方案：一个只包含 Learning 与 Recording 两个 Skill 的轻量插件。v0.1.0 清单和 ZIP 已准备，尚未保存到 ChatGPT 账户或完成安装验证。

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

## Learning 与 Recording 如何衔接

两个 Skill 不需要额外的结构化交接报告。

- Learning 负责当前的共同学习过程，不需要为了将来记录而在每次结束时生成固定总结。
- 用户在同一对话中主动调用 Recording 时，当前对话本身就是交接材料。Recording 回看这段讨论，选择真正影响理解的内容，再整理成记录。
- 如果用户在另一个对话中调用 Recording，原来的学习过程不一定处于当前上下文。此时只能使用实际可访问的历史资料；缺少关键内容时请用户提供原对话或相关片段，不要假设能够读取全部聊天历史。
- Notion 中的旧日记主要用于参考个人文风和保存位置，不能替代本次学习对话，作为本次思维变化的事实来源。

这样既能让两个 Skill 自然配合，也不会让 Learning 多出一项固定的收尾任务。
 
## 先不加入的组件

- **不自建 MCP Server**：当前核心工作是指导学习和记录，已有模型能力足够完成主要任务。
- **不重新实现 Memory**：用户偏好和跨会话上下文使用 ChatGPT 已有能力。
- **不立即建设 RAG**：具体历史资料先使用现有 Notion / GitHub 检索能力。只有普通检索反复找不到需要的内容，且检索质量确实影响任务时，再评估语义检索或 RAG。
- **不自动修改 Skill**：真实使用后发现有代表性的问题或成功案例时，用户可以主动提出复盘；讨论确认后再更新规则或正反例，并通过 Git 记录版本。

## 调用与外部工具

- 两个 Skill 都设置为显式调用，避免普通对话自动介入。
- Notion 用于检索真实日记样本和按用户要求保存记录；优先复用用户已经连接的 Notion 工具。
- Writing Style 不作为初版必要依赖。用户目前可用的主要文风样本来自 Notion 日记和可访问的对话；如果以后连接了有效的个人写作样本，再评估是否需要调用该工具。
- 若插件打包时需要声明某个现有 App 依赖，必须使用已验证的 App ID，并先检查授权与平台限制，不猜测标识。
- 如果某项外部能力在当前 ChatGPT 界面不可用，Skill 应说明限制并提供可行的草稿或替代流程，不声称已完成外部写入。

## 打包与验证

- 根目录使用 Agent Plugins 1.0 的 `plugin.json`；插件名为 `learning-recording`，版本为 `0.1.0`，分类为 `Education & Research`。
- 一个插件包含两个 Skill，目录名和 Skill frontmatter 保持一致。
- 核心指令放在各自的 `SKILL.md`；案例放在对应 `references/`，按需读取。
- 两个 Skill 的 `agents/openai.yaml` 都设置 `allow_implicit_invocation: false`，由用户主动选择。
- 初版不包含 MCP Server、外部 App 绑定或自建服务；Notion 等工具由用户在 ChatGPT 中另行连接。
- 已生成 ZIP 并检查清单、目录层级、Skill frontmatter、引用文件和压缩包完整性。
- 当前环境没有提供将该 ZIP 直接保存为账户插件的创建入口，因此账户保存与安装仍待完成；不能把 ZIP 生成视为插件已经安装。
- 安装后分别测试 Learning 与 Recording，并检查实际调用、外部工具权限和写入结果。测试通过前不称为正式版本；GitHub 是源文件和版本历史的唯一事实来源。

## 版本推进

1. 完成两个 Skill 的行为映射、核心指令和按需案例。
2. 整体检查两个 Skill 的职责边界、目录、元数据和插件清单。
3. 打包为一个 skills-only plugin 并安装。
4. 在 ChatGPT Chat 中测试两个 Skill 的显式调用及 Notion 工具协作。
5. 根据真实表现讨论修改，不做每次使用后的强制评估。

## 职责与独立使用审查

- [职责边界审查](skill-boundary-review-v0.1.md)：确认两个 Skill 的分工，没有必要增加固定交接报告。
- [独立使用审查](independent-use-review-v0.1.md)：检查只使用 Learning、只使用 Recording、同一对话先后调用，以及只保存局部资料四种场景。

从设计和配置来看，两个 Skill 可以独立使用，也可以在同一对话中先后配合。两个 Skill 都设置为显式调用。ChatGPT 安装后的实际显示、切换和外部工具权限仍待打包后验证。
