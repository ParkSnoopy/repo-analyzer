# Repo Analyzer

一个 AI 编程 Agent 技能，用于对开源项目进行深度架构分析，生成专业级架构报告——包含设计洞察、权衡分析和 Mermaid 架构图。

兼容 [Claude Code](https://claude.ai/claude-code)、[Codex](https://github.com/openai/codex)、[OpenClaw](https://github.com/anthropics/openclaw) 等所有支持 Skills 格式的 AI 编程 Agent。

**[English Documentation](README.md)**

## 相关文章

- [一键生成专业级分析报告：迭代20多轮的深度架构分析Skill，我决定开源了](https://mp.weixin.qq.com/s/sOPHFaNS8pIkhB4F-FysTQ)
- [Claude Code 源码深度架构分析](https://mp.weixin.qq.com/s/GjZ-tFBVwfJwK11F1lP5TQ)

## 快速安装

**npx（推荐）**

```bash
npx skills add yzddmr6/repo-analyzer
```

**Plugin Marketplace**

```
/plugin marketplace add yzddmr6/repo-analyzer
/plugin install repo-analyzer@repo-analyzer
```

**手动安装（Git Clone）**

```bash
# macOS / Linux
git clone https://github.com/yzddmr6/repo-analyzer.git ~/.claude/skills/repo-analyzer

# Windows
git clone https://github.com/yzddmr6/repo-analyzer.git %USERPROFILE%\.claude\skills\repo-analyzer
```

## 特性

- **架构级分析** — 聚焦"为什么这样设计"，而不是"这个文件里有什么函数"
- **自适应报告结构** — 没有固定模板，章节结构根据每个项目的特点动态生成
- **并行 Subagent 分析** — 为每个核心模块启动独立 Agent 并行分析，高效处理大型代码库
- **竞品定位** — 对比设计哲学和技术路线差异，而非功能清单
- **Mermaid 架构图** — 贯穿报告的系统架构图、数据流图和模块时序图
- **交互式工作流** — 根据项目特征生成针对性问题，在深入分析前与你对齐方向

## 使用方式

直接对 Claude Code 说：

```
分析项目 https://github.com/astral-sh/ruff
```

```
分析一下 ollama/ollama 这个仓库的架构
```

```
对比分析 express 和 fastify
```

支持 `owner/repo` 简写、GitHub/GitLab/Gitee 完整 URL、本地路径。

### 触发关键词

提到以下任意关键词时技能自动激活：

`分析项目` `分析仓库` `分析 GitHub` `项目分析` `源码分析` `架构分析` `代码分析` `学习这个项目` `研究这个框架` `看看这个库怎么实现的` `对比两个项目` `项目评测` `框架评测`

> **说明：** 默认输出中文报告。如果用其他语言提问，会跟随你的语言。

## 分析模式

扫描代码库后，技能会让你选择分析深度：

| 模式 | 核心模块覆盖率 | 次要模块覆盖率 | 适用场景 |
|------|--------------|--------------|---------|
| **快速分析** | >= 30% | >= 10% | 快速了解项目全貌 |
| **标准分析**（默认） | >= 60% | >= 30% | 常规架构分析 |
| **深度分析** | >= 90% | >= 60% | 深入研究每个设计决策 |

## 工作流程

1. **克隆与扫描** — 克隆仓库（或使用本地路径），按模块统计有效代码行数
2. **外部调研** — 搜索项目评价、竞品对比、架构讨论；遍历官网关键页面
3. **自适应提问** — 根据项目特征生成针对性问题，不是固定问题清单
4. **动态报告结构** — 根据你的回答和项目特点设计章节布局
5. **并行深度分析** — 为每个核心模块启动 Subagent 并行分析
6. **交叉验证** — 跨模块验证结论，检查代码阅读覆盖率
7. **多源融合** — 将调研、模块分析、洞察融合为连贯叙事
8. **最终报告** — 输出包含 Mermaid 图表的单一 Markdown 文件

## 报告输出

最终报告保存为单一 Markdown 文件：

```
~/repo-analyses/{项目名}-{日期}/ANALYSIS_REPORT.md
```

每份报告包含（根据项目特点调整）：

- **问题引入** — 用具体场景讲清楚项目解决什么问题、现有方案的不足
- **竞品定位** — 与同类项目的设计哲学和技术路线差异
- **项目全景** — 一览架构全貌
- **深度模块分析** — Why > What 的分析，包含权衡和业界对比
- **评价与启发** — 诚实的优缺点评估和可借鉴的设计经验
- **架构图表** — 系统全局图和各模块流程图（Mermaid）

## 可选依赖

技能开箱即用，使用 Claude Code 内置工具。以下 MCP 服务器为可选增强：

- [Tavily MCP](https://github.com/tavily-ai/tavily-mcp) — 通过 `tavily_crawl` 遍历网站
- [Brave Search MCP](https://github.com/anthropics/anthropic-quickstarts/tree/main/brave-search-mcp) — 替代搜索引擎

## 文件结构

```
repo-analyzer/
├── .claude-plugin/
│   └── plugin.json                         # 插件元数据
├── package.json                            # 包清单
├── skills/
│   └── repo-analyzer/
│       ├── SKILL.md                        # 技能主定义
│       └── references/
│           ├── analysis-guide.md           # 分析哲学与评价框架
│           └── module-analysis-guide.md    # 模块分析指南与 Subagent 模板
├── README.md                               # 英文文档
├── README.zh.md                            # 中文文档
└── LICENSE                                 # MIT 许可证
```

## 贡献

欢迎提交 Issue 和 Pull Request！

如果你想改进分析工作流，核心逻辑在 `skills/repo-analyzer/SKILL.md`。评价框架和 Subagent 模板在 `references/` 目录下。

## 许可证

[MIT](LICENSE)
