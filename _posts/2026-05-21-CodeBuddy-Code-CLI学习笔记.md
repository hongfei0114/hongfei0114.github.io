---
layout:     post
title:      Claude Code CLI 学习笔记
subtitle:   从配置到自动化，8节课掌握AI编程助手核心用法
date:       2026-05-21
author:     hongfei
header-img: img/post-bg-debug.png
catalog: true
tags:
    - AI
    - Claude Code
    - CLI
    - 效率工具
---

# Claude Code CLI 学习笔记

> 记录每节课的核心要点，便于日后快速回顾。

---

## 第1课：项目配置基础

### 三层 CLAUDE.md

| 位置 | 作用域 | 优先级 |
|------|--------|--------|
| `~/.claude/CLAUDE.md` | 全局 | 低 |
| `项目根目录/CLAUDE.md` | 项目级 | 中 |
| `子目录/CLAUDE.md` | 目录级 | 高 |

### 关键文件

- **CLAUDE.md** — Claude 每次对话都会读，写清楚它就听话
- **settings.json** — 团队共享的权限和 hooks（提交 git）
- **settings.local.json** — 个人私有配置（不提交 git）
- **.gitignore** — 安全底线，`.env` 和 `settings.local.json` 绝不提交

### 要点

1. `/init` 可以自动生成初始 CLAUDE.md
2. CLAUDE.md 越清晰，AI 输出越精准
3. 新项目第一件事：init git + 写 CLAUDE.md + 配 settings.json

---

## 第2课：日常高效操作

### Prompt 五段位

| 段位 | 特征 | 例子 |
|------|------|------|
| 青铜 | 模糊 | "写个工具函数" |
| 白银 | 有目标缺上下文 | "写个 login 函数，用 TS" |
| 黄金 | 具体+有约束 | "在 xxx.ts 中写 formatDate，参数 Date，返回 YYYY-MM-DD" |
| 钻石 | 引用现有模式 | "参考 utils.ts 的风格，添加 formatTime" |
| 王者 | 分步+验证 | "1.写函数 2.写测试 3.运行测试确认通过" |

### 不确定时的三种方法

1. **追问法**："你觉得应该怎么设计？先给方案，不要直接写代码"
2. **选项法**："给我 2-3 个方案，我选一个你再写"
3. **场景法**："场景是 xxx，你来设计需要哪些函数，列出来我确认"

> 口诀：不确定就让 Claude 先出方案，你只负责选。

### 斜杠命令

| 命令 | 场景 |
|------|------|
| `/init` | 新项目起步（只用一次） |
| `/compact` | 对话太长 Claude 变慢时压缩 |
| `/clear` | 切换到完全不相关的任务 |
| `/review` | 提 PR 前审查代码 |
| `/help` | 查看所有可用命令 |

### 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Cmd+Enter` | 发送消息 |
| `Enter` | 换行 |
| `Esc` | 打断生成（跑偏时立刻按） |
| `Ctrl+C` | 强制中断（连工具调用都停） |
| `↑` | 翻出上一条 prompt |
| `Tab` | 自动补全 |

> 核心习惯：Claude 跑偏 → 立刻 Esc，不要等它写完再纠正。

---

## 第3课：权限管理

### 三层权限架构

```
全局 ~/.claude/settings.json        → 个人习惯，所有项目通用
项目 .claude/settings.json          → 团队共享，提交 git
本地 .claude/settings.local.json    → 私有配置，不提交
```

优先级：本地 > 项目 > 全局

### deny 永远赢

- 任何层的 deny 都不可被其他层的 allow 覆盖
- 这是安全设计的基石

### 匹配语法

```
Bash(git status)     ← 精确匹配
Bash(git log*)       ← 前缀通配
Bash(npm *)          ← 允许所有 npm 子命令
Read(**)             ← 允许读取任何文件
```

### 安全等级

```
🟢 安全（只读）     git status, git log*, ls*, find*, which*
🟡 中等（副作用）   npm test*, npm run build*
🔴 危险（别白名单） rm *, git push --force*, Edit(**)
```

### 什么放哪里

| 全局 | 项目 | 本地 |
|------|------|------|
| git status、ls、which | npm test、deny 危险命令 | 个人 MCP 工具 |

### 实用命令

- `/fewer-permission-prompts` — 扫描历史，自动添加常用命令到白名单（积累一段时间后再用）

---

## 第4课：Hooks 自动化

### 4个触发点

| 触发点 | 时机 | 典型用途 |
|--------|------|----------|
| `PreToolUse` | 工具执行前 | 命令重写、拦截危险操作 |
| `PostToolUse` | 工具执行后 | 输出过滤、自动格式化 |
| `Stop` | Claude 停止响应时 | 提示音、发通知 |
| `Notification` | Claude 发通知时 | 播放声音、发消息 |

### Hook 配置结构

```jsonc
{
  "hooks": {
    "触发点": [
      {
        "matcher": "工具名",    // 空 = 匹配所有
        "hooks": [
          {
            "type": "command",
            "command": "要执行的命令或脚本路径"
          }
        ]
      }
    ]
  }
}
```

### PreToolUse 返回值

| stdout 返回 | 效果 |
|-------------|------|
| 空 / 无输出 | 放行，正常执行 |
| `{"decision":"allow"}` | 明确放行 |
| `{"decision":"block","reason":"..."}` | 阻止执行 |
| `{"decision":"replace","content":{...}}` | 替换工具输入 |

### matcher 值

| 值 | 含义 |
|----|------|
| `""` | 匹配所有 |
| `"Bash"` | 只匹配 Bash 工具 |
| `"Read"` | 只匹配文件读取 |
| `"Write"` | 只匹配文件写入 |
| `"Edit"` | 只匹配文件编辑 |

### 执行顺序

- 同触发点：全局 Hook 先执行，项目 Hook 后执行
- 任何一个返回 block 就停止后续执行

### 双重保护设计

```
第1道：permissions.deny     ← 静态字符串匹配
第2道：PreToolUse Hook     ← 灵活脚本检测（grep/正则/外部工具）
```

### 注意事项

- Hook 脚本必须 `chmod +x` 加执行权限
- 脚本通过 stdin 接收 JSON，stdout 返回 JSON
- Hook 应快速执行（<5秒），否则影响交互体验

---

## 第5课：MCP 进阶

### MCP 是什么

MCP Server 是独立进程，Claude 通过标准协议调用它提供的"工具"，扩展能力边界。

```
Claude Code ←→ MCP 协议 ←→ MCP Server ←→ 外部服务
```

### 配置结构

```jsonc
{
  "mcpServers": {
    "服务名": {
      "command": "启动命令",
      "args": ["参数"],
      "env": { "KEY": "value" }
    }
  }
}
```

### 配置位置

| 层级 | 场景 | 例子 |
|------|------|------|
| 全局 | 个人常用 | GitHub MCP |
| 项目 | 团队共享 | 项目专用数据库 |
| 本地 | 私有工具 | 个人 token 相关 |

### 常用 MCP Server

| 名称 | 能力 |
|------|------|
| filesystem | 文件系统沙盒访问 |
| github | Issue/PR/代码搜索 |
| brave-search | 网页搜索 |
| sqlite | 本地数据库查询 |
| puppeteer | 浏览器自动化 |
| memory | 跨对话持久记忆 |

### 安全三问

1. 这个 MCP server 能访问什么？（沙盒范围）
2. 我信任这个 server 的代码吗？（官方 vs 第三方）
3. Token/Key 放哪里？（永远放 local，不放项目级）

### 注意事项

- 修改 MCP 配置后必须**重启 Claude Code** 才生效
- MCP server 在会话启动时连接，中途修改不会自动重载

---

## 第6课：Sub-agents 子代理

### 核心价值

- **保护上下文** — 探索/搜索结果不污染主对话
- **专业分工** — 每个子代理有专属 prompt 和工具集
- **控制成本** — 可用更便宜的模型（如 Haiku）
- **安全隔离** — 限制子代理只能读不能写

### 定义方式

文件格式：Markdown + YAML frontmatter，存放在：
- `.claude/agents/` — 项目级（提交 git，团队共享）
- `~/.claude/agents/` — 用户级（所有项目可用）

### 关键 Frontmatter 字段

| 字段 | 说明 |
|------|------|
| `name` | 唯一标识，小写+连字符 |
| `description` | Claude 据此判断何时委派（写清楚！） |
| `tools` | 允许的工具白名单 |
| `model` | haiku/sonnet/opus/inherit |
| `isolation` | 设为 `worktree` 隔离文件系统 |
| `background` | 设为 `true` 后台运行 |
| `maxTurns` | 防死循环 |

### 调用方式

1. **自动** — Claude 根据 description 自动委派（推荐）
2. **`/agents`** — 管理面板，查看/创建/编辑
3. **CLI** — `claude --agents '{...}'` 动态定义

### 工具权限设计

```
只读搜索 → tools: Read, Grep, Glob
能跑命令 → tools: Read, Grep, Glob, Bash + disallowedTools: Write, Edit
完全信任 → 不写 tools（继承全部，慎用）
```

### 内置子代理

| 名称 | 模型 | 用途 |
|------|------|------|
| Explore | Haiku | 搜索/探索代码库 |
| Plan | 继承 | Plan Mode 调研 |
| General-purpose | 继承 | 复杂多步任务 |

### 与其他并行方式的区别

| 方式 | 场景 |
|------|------|
| Sub-agents | 单会话内侧任务，返回摘要 |
| Agent View | 多独立任务，多窗口监控 |
| Agent Teams | 需要互相协作通信 |
| Worktrees | 编辑同一仓库需隔离 |
| /batch | 大规模重构，并行 PR |

---

## 第7课：并行工作流

### 三种并行方式

| 方式 | 启动命令 | 适合场景 |
|------|----------|----------|
| Agent View | `claude agents` | 多个独立任务，你来监控 |
| Worktrees | `claude -w name` | 手动隔离，精确控制分支 |
| /batch | 对话中 `/batch` | 仓库级批量重构 |

### Agent View 核心操作

```bash
claude agents                  # 打开监控台
claude --bg "任务描述"          # 派发后台任务
claude attach <id>             # 附加到会话介入
claude logs <id>               # 查看日志
claude stop <id>               # 停止
claude rm <id>                 # 移除
```

- 后台会话自动使用 worktree 隔离
- 关闭终端后会话仍在运行（daemon）
- 需要人工介入时 attach 进去

### Worktree 工作流

```bash
claude -w fix-auth                # 创建 worktree + 进入
claude -w #123                    # 从 PR 创建
claude -w name --tmux             # 配合 tmux 多窗口
git worktree list                 # 查看所有
git worktree remove <path>        # 删除
```

路径：`项目/.claude/worktrees/<name>/`

隔离原理：独立 git checkout，文件互不可见。

#### `-w` 背后的 git 操作详解

执行 `claude -w fix-auth-bug` 等价于以下 git 命令序列：

```bash
# 1. 创建 worktree（独立目录 + 新分支）
git worktree add .claude/worktrees/fix-auth-bug -b fix-auth-bug
#   → 在 .claude/worktrees/fix-auth-bug/ 生成项目完整副本
#   → 自动创建并切换到 fix-auth-bug 分支（基于当前 HEAD）

# 2. Claude Code 将工作目录设为 worktree 路径
#   → 所有 Read/Write/Edit/Bash 操作在 worktree 内执行
#   → 主目录完全不受影响

# 3. 退出会话时提示：keep 或 remove
```

#### 完整生命周期

```bash
# 创建 + 进入
claude -w fix-auth-bug

# --- 在 worktree 会话中 ---
# 修改文件、提交
git add examples/auth.ts
git commit -m "fix: 修复 null 检查"

# --- 退出会话后 ---
# 合并回主分支
git merge fix-auth-bug

# 清理
git worktree remove .claude/worktrees/fix-auth-bug
git branch -d fix-auth-bug
```

#### 为什么不用 git stash + 切分支？

| 传统方式 | Worktree |
|----------|----------|
| stash → 切分支 → 改 → 切回 → pop | 两个目录同时存在，互不干扰 |
| stash 冲突风险 | 无冲突 |
| 只能在一个分支工作 | 可以同时在多个分支工作 |
| 切换时编辑器状态丢失 | 每个 worktree 独立打开编辑器 |

### /batch 批量重构

```
/batch "把所有 console.log 替换为 logger.info"
→ 自动拆成 5-30 个子代理
→ 每个在独立 worktree 中工作
→ 每个完成后开 PR
```

适用：import 迁移、API 升级、风格统一、依赖替换。

### 选择指南

```
同时修多个独立 bug     → Agent View
实验性方案不影响 main  → 单个 Worktree
200 个文件批量改       → /batch
任务中间搜索大量代码   → Sub-agents（第6课）
```

### 注意事项

- 并行 = token 消耗翻倍
- 多个 worktree 改同一文件会冲突
- 先想清楚谁改哪些文件再拆分
- 并行的前提是任务之间无依赖

---

## 第8课：Headless & CLI 自动化

### `-p` 打印模式

非交互执行，结果输出到 stdout 后退出。可以当普通命令行工具用。

```bash
# 基本用法
claude -p "分析这个项目的架构"

# 管道输入
cat error.log | claude -p "解释这些错误"

# 输出为 JSON（便于脚本解析）
claude -p "列出安全问题" --output-format json

# 限制预算
claude -p --max-budget-usd 1.00 "重构 auth 模块"

# 限制轮次
claude -p --max-turns 5 "写单元测试"
```

### 管道组合（Unix 哲学）

```bash
# 分析最近的日志
tail -200 app.log | claude -p "有异常吗？"

# 审查变更的文件
git diff main --name-only | claude -p "这些文件有安全风险吗"

# 批量翻译
claude -p "把新增的字符串翻译成法语，提一个 PR"
```

### 写进脚本

```bash
#!/bin/bash
# check-security.sh — 一键安全扫描
result=$(claude -p "扫描 examples/ 的安全漏洞，输出JSON格式" --output-format json)
echo "$result" | jq '.vulnerabilities'
```

### CI/CD 集成

```yaml
# GitHub Actions 示例
- name: Security scan
  run: |
    claude -p "审查这个 PR 的安全风险" \
      --output-format json \
      --max-turns 3 \
      --allowedTools "Read" "Grep" "Glob"
```

### 关键 flags

| Flag | 作用 |
|------|------|
| `-p "prompt"` | 非交互执行 |
| `--output-format json` | 输出 JSON 便于解析 |
| `--output-format stream-json` | 流式 JSON |
| `--max-turns N` | 限制执行轮次 |
| `--max-budget-usd N` | 限制花费 |
| `--allowedTools "..."` | 限制可用工具 |
| `--model sonnet` | 指定模型 |
| `--bare` | 最小模式，跳过 hooks/MCP 加载更快 |
