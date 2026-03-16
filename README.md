# code-explorer — Claude Code 源码解读 Skill

一个为 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 设计的源码探索 Skill，帮助你快速读懂陌生代码库。

## 功能特点

- **三种分析模式**：快速解答（单函数）/ 标准分析（单模块）/ 深度探索（整项目）
- **多语言专项策略**：Go、Python、JavaScript/TypeScript 各有专属分析套路
- **参考脚本自动化**：自动检测语言、查找入口点、获取 Git 历史上下文
- **Mermaid 可视化**：自动生成架构图、时序图、状态机图
- **Git 洞察**：结合提交历史解释代码演变背景
- **Claude Code Hooks**：智能交互增强，运行时追踪与保护

## 安装

### 方式一：一键脚本（推荐）

```bash
git clone https://github.com/googs1025/claude-code-explorer-skill.git
cd claude-code-explorer-skill
bash install.sh
```

安装脚本会自动完成以下步骤：
1. 将 Skill 文件复制到 `~/.claude/skills/code-explorer/`
2. 安装 Git Hooks（仅当前仓库开发用）
3. 安装 Claude Code Hooks 到 `~/.claude/hooks/code-explorer/`
4. 注册 Hooks 到 `~/.claude/settings.json`

### 方式二：手动安装

```bash
# 安装 Skill 文件
mkdir -p ~/.claude/skills/code-explorer/lang ~/.claude/skills/code-explorer/scripts
cp SKILL.md ~/.claude/skills/code-explorer/
cp lang/*.md ~/.claude/skills/code-explorer/lang/
cp scripts/*.sh ~/.claude/skills/code-explorer/scripts/
chmod +x ~/.claude/skills/code-explorer/scripts/*.sh
```

> 手动安装不包含 Claude Code Hooks，如需 Hooks 功能请使用一键脚本。

## 使用方式

安装后，在 Claude Code 中用以下任意方式触发：

```bash
# 显式调用
/code-explorer src/main.go
/code-explorer handleRequest

# 自然语言触发（自动识别）
帮我理解 handleRequest 的执行流程
解释这个项目的架构
这段代码的核心逻辑是什么
为什么这里要用 goroutine？
这个接口是怎么用的
帮我看懂这个项目
```

## 分析模式

| 模式 | 触发条件 | 输出 |
|------|---------|------|
| **快速模式** | 单函数、单变量、单行疑问 | 简洁解释 + 注意点 |
| **标准模式** | 单个文件、单个模块 | 完整四阶段分析 + Mermaid 图 |
| **深度模式** | 整个项目、架构梳理 | 宏观扫描 → 确认范围 → 深入分析 |

### 四阶段分析流程（标准/深度模式）

```
Phase 1: 宏观扫描 → 识别技术栈、目录结构、依赖、架构分层
Phase 2: 关键路径追踪 → 定位核心函数、追溯调用链、数据流追踪
Phase 3: 抽象与可视化 → 伪代码重构、Mermaid 图表、设计模式识别
Phase 4: 综合解释 → 一句话总结、设计意图、潜在风险、Git 洞察
```

### 输出示例

**标准/深度模式输出结构：**

```markdown
## 核心功能摘要
[一句话，不超过 50 字]

## 架构/流程图
[Mermaid 图表]

## 关键逻辑拆解
入口点 → 核心流程 → 数据流向

## 深度洞察
设计意图 / 关键依赖 / Git 背景 / 注意事项

## 建议深入探索
[引导性问题]
```

## 支持语言

| 语言 | 策略文件 | 特色检查 |
|------|---------|---------|
| **Go** | `lang/go.md` | goroutine 泄漏、channel 死锁、interface 边界 |
| **Python** | `lang/python.md` | 框架识别、循环导入、装饰器追踪 |
| **JavaScript/TypeScript** | `lang/javascript.md` | XSS 风险、类型安全、框架路由 |
| 其他语言 | 通用策略 | 基础架构分析 |

## 参考脚本

Skill 内置三个辅助脚本，分析时自动执行：

| 脚本 | 功能 | 示例输出 |
|------|------|---------|
| `scripts/detect_lang.sh` | 检测项目语言、版本、框架 | `lang=go`, `framework=gin` |
| `scripts/find_entry.sh <lang>` | 按语言查找入口点和路由注册 | `./cmd/server/main.go` |
| `scripts/git_context.sh [file]` | 获取 Git 历史、活跃文件、贡献者 | 最近 20 条提交、Top 5 活跃文件 |

## Claude Code Hooks

安装脚本会自动配置以下 [Claude Code Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks)，增强分析体验：

| Hook | 触发时机 | 功能 |
|------|---------|------|
| **pre-prompt.sh** | `UserPromptSubmit` — 用户提交 prompt 时 | 检测深度/探索分析意图，交互式确认分析范围、关注重点和详细度，注入配置到上下文 |
| **post-bash.sh** | `PostToolUse(Bash)` — Bash 工具执行后 | 验证 `detect_lang` / `find_entry` / `git_context` 脚本输出，创建会话标记 |
| **post-read.sh** | `PostToolUse(Read)` — Read 工具执行后 | 追踪文件读取数量，Phase 1 上限(5个)和总上限(10个)时提醒 |
| **on-stop.sh** | `Stop` — 会话结束时 | 输出本次会话读取统计，清理临时文件 |

### Hooks 工作流程

```
用户输入 prompt
    │
    ▼
pre-prompt.sh 检测关键词
    │
    ├─ 匹配「架构/整体/overview...」→ 深度分析配置交互
    ├─ 匹配「解释/理解/how/why...」→ 探索分析配置交互
    └─ 无匹配 → 直接通过
    │
    ▼
Claude 执行分析（调用 Bash/Read 工具）
    │
    ├─ post-bash.sh → 验证脚本输出，创建会话标记
    └─ post-read.sh → 计数读取文件数，超限提醒
    │
    ▼
会话结束
    │
    ▼
on-stop.sh → 输出统计，清理临时文件
```

> **注意**：`pre-prompt.sh` 内置超时保护（10 秒），TTY 不可用时自动使用默认配置，不会阻塞 Claude Code。

## Git Hooks（开发者用）

项目还包含用于开发本 Skill 时的 Git Hooks：

| Hook | 功能 |
|------|------|
| `git-hooks/pre-commit` | 对暂存的 `.sh` 文件执行 shellcheck 检查 |
| `git-hooks/commit-msg` | 验证 commit message 符合 Conventional Commits 格式 |

> 这些 Git Hooks 仅在本项目开发时使用，不影响用户项目。

## 项目结构

```
claude-code-explorer-skill/
├── SKILL.md                    # Claude Code Skill 主文件（分析流程与策略）
├── install.sh                  # 一键安装脚本
├── uninstall.sh                # 一键卸载脚本
├── README.md                   # 项目说明
├── lang/                       # 语言专项分析策略
│   ├── go.md                   #   Go 专项策略
│   ├── python.md               #   Python 专项策略
│   └── javascript.md           #   JS/TS 专项策略
├── scripts/                    # 辅助分析脚本
│   ├── detect_lang.sh          #   语言检测
│   ├── find_entry.sh           #   入口点查找
│   └── git_context.sh          #   Git 上下文获取
├── claude-hooks/               # Claude Code Hooks（运行时增强）
│   ├── pre-prompt.sh           #   分析意图检测与配置注入
│   ├── post-bash.sh            #   脚本输出验证与会话标记
│   ├── post-read.sh            #   文件读取追踪
│   └── on-stop.sh              #   会话统计与清理
└── git-hooks/                  # Git Hooks（开发用）
    ├── pre-commit              #   shellcheck 检查
    └── commit-msg              #   Conventional Commits 验证
```

## 卸载

```bash
cd claude-code-explorer-skill
bash uninstall.sh
```

卸载脚本会自动：
1. 删除 `~/.claude/skills/code-explorer/` 及备份
2. 删除 `~/.claude/hooks/code-explorer/`
3. 从 `~/.claude/settings.json` 中移除 code-explorer 相关 hooks（保留其他配置）
4. 删除本项目的 Git Hooks
5. 清理 `/tmp/code-explorer-*` 临时文件

## License

MIT
