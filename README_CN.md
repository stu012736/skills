# 技能
技能是指包含指令、脚本和资源的文件夹，Claude 会动态加载这些内容以提高在专业任务上的表现。技能教会 Claude 如何以可重复的方式完成特定任务，无论是使用公司品牌指南创建文档、使用组织特定工作流程分析数据，还是自动化个人任务。

更多信息请查看：
- [什么是技能？](https://support.claude.com/en/articles/12512176-what-are-skills)
- [在 Claude 中使用技能](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [如何创建自定义技能](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [使用代理技能为现实世界装备代理](https://anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

# 关于此仓库

此仓库包含示例技能，展示了 Claude 技能系统的可能性。这些示例涵盖了从创意应用（艺术、音乐、设计）到技术任务（测试 Web 应用、MCP 服务器生成）再到企业工作流程（通信、品牌等）的各种场景。

每个技能都包含在自己的目录中，其中有一个 `SKILL.md` 文件，包含 Claude 使用的指令和元数据。浏览这些示例以获取创建自己技能的灵感，或了解不同的模式和方法。

此仓库中的示例技能是开源的（Apache 2.0）。我们还在 [`document-skills/`](./document-skills/) 文件夹中包含了支持 [Claude 文档能力](https://www.anthropic.com/news/create-files)的文档创建和编辑技能。这些是源代码可用的，但不是开源的，但我们希望与开发者分享这些作为参考，了解在生产 AI 应用中使用的更复杂技能。

**注意：** 这些是用于启发和学习的参考示例。它们展示的是通用能力，而不是组织特定的工作流程或敏感内容。

## 免责声明

**这些技能仅供演示和教育目的提供。** 虽然其中一些功能可能在 Claude 中可用，但您从 Claude 获得的实现和行为可能与这些示例中显示的不同。这些示例旨在说明模式和可能性。在依赖它们执行关键任务之前，请务必在您自己的环境中彻底测试技能。

# 示例技能

此仓库包含多样化的示例技能集合，展示了不同的能力：

## 创意与设计
- **algorithmic-art** - 使用 p5.js 创建生成艺术，包含种子随机性、流场和粒子系统
- **canvas-design** - 使用设计哲学在 .png 和 .pdf 格式中设计精美的视觉艺术
- **slack-gif-creator** - 创建针对 Slack 大小限制优化的动画 GIF

## 开发与技术
- **artifacts-builder** - 使用 React、Tailwind CSS 和 shadcn/ui 组件构建复杂的 claude.ai HTML 工件
- **mcp-server** - 创建高质量 MCP 服务器以集成外部 API 和服务的指南
- **webapp-testing** - 使用 Playwright 测试本地 Web 应用程序以进行 UI 验证和调试

## 企业与通信
- **brand-guidelines** - 将 Anthropic 的官方品牌颜色和排版应用于工件
- **internal-comms** - 编写内部通信，如状态报告、新闻通讯和常见问题解答
- **theme-factory** - 使用 10 个预设专业主题为工件设置样式，或即时生成自定义主题

## 元技能
- **skill-creator** - 创建有效扩展 Claude 能力的技能指南
- **template-skill** - 用作新技能起点的基本模板

# 文档技能

`document-skills/` 子目录包含 Anthropic 开发的技能，用于帮助 Claude 创建各种文档文件格式。这些技能展示了处理复杂文件格式和二进制数据的高级模式：

- **docx** - 创建、编辑和分析 Word 文档，支持跟踪更改、注释、格式保留和文本提取
- **pdf** - 全面的 PDF 操作工具包，用于提取文本和表格、创建新 PDF、合并/拆分文档以及处理表单
- **pptx** - 创建、编辑和分析 PowerPoint 演示文稿，支持布局、模板、图表和自动幻灯片生成
- **xlsx** - 创建、编辑和分析 Excel 电子表格，支持公式、格式设置、数据分析和可视化

**重要免责声明：** 这些文档技能是时间点快照，不积极维护或更新。这些技能的版本随 Claude 预装。它们主要用作参考示例，说明 Anthropic 如何开发处理二进制文件格式和文档结构的更复杂技能。

# 在 Claude Code、Claude.ai 和 API 中尝试

## Claude Code
您可以通过在 Claude Code 中运行以下命令将此仓库注册为 Claude Code 插件市场：
```
/plugin marketplace add anthropics/skills
```

然后，安装特定的技能集：
1. 选择 `浏览并安装插件`
2. 选择 `anthropic-agent-skills`
3. 选择 `document-skills` 或 `example-skills`
4. 选择 `立即安装`

或者，直接通过以下方式安装任一插件：
```
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
```

安装插件后，您只需提及技能即可使用它。例如，如果您从市场安装了 `document-skills` 插件，您可以要求 Claude Code 执行类似的操作："使用 PDF 技能从 path/to/some-file.pdf 提取表单字段"

## Claude.ai

这些示例技能在 Claude.ai 中已对所有付费计划可用。

要使用此仓库中的任何技能或上传自定义技能，请按照 [在 Claude 中使用技能](https://support.claude.com/en/articles/12512180-using-skills-in-claude#h_a4222fa77b) 中的说明操作。

## Claude API

您可以通过 Claude API 使用 Anthropic 的预构建技能并上传自定义技能。有关更多信息，请参阅 [技能 API 快速入门](https://docs.claude.com/en/articles/12512198-creating-custom-skills)。

# 创建基本技能

技能创建很简单 - 只需一个包含 YAML 前置数据和指令的 `SKILL.md` 文件的文件夹。您可以使用此仓库中的 **template-skill** 作为起点：

```markdown
---
name: my-skill-name
description: 技能的完整描述，说明它做什么以及何时使用
---

# 我的技能名称

[在此处添加 Claude 在技能激活时将遵循的指令]

## 示例
- 示例用法 1
- 示例用法 2

## 指南
- 指南 1
- 指南 2
```

前置数据只需要两个字段：
- `name` - 技能的唯-标识符（小写，用连字符代替空格）
- `description` - 技能的完整描述，说明它做什么以及何时使用

下面的 Markdown 内容包含 Claude 将遵循的指令、示例和指南。有关更多详细信息，请参阅 [如何创建自定义技能](https://support.claude.com/en/articles/12512198-creating-custom-skills)。

# 合作伙伴技能

技能是教 Claude 如何更好地使用特定软件的好方法。当我们看到合作伙伴的优秀示例技能时，我们可能会在这里重点介绍一些：

- **Notion** - [Notion 的 Claude 技能](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0)