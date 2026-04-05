---
layout:     post
title:      CodeBuddy CLI 团队使用指南
subtitle:   记忆系统、规则管理与团队协作最佳实践
date:       2026-04-06
author:     hongfei
header-img: img/post-bg-debug.png
catalog: true
tags:
    - AI
    - CodeBuddy
    - 效率工具
    - 团队协作
---

# CodeBuddy CLI 团队使用指南

> 本文档面向团队成员，介绍 CodeBuddy CLI 的核心概念和团队协作最佳实践。

---

## 一、记忆系统详解

### 1.1 记忆类型与存储位置

CodeBuddy 提供分层的记忆结构，每层有不同的用途和共享范围：

| 记忆类型 | 存储位置 | 用途 | 是否提交 git |
|---------|---------|------|-------------|
| 用户记忆 | `~/.codebuddy/CODEBUDDY.md` | 个人全局偏好（适用所有项目） | 否 |
| 用户规则 | `~/.codebuddy/rules/*.md` | 个人模块化规则 | 否 |
| **项目记忆** | `./CODEBUDDY.md` | **团队共享的项目指令** | **是** |
| **项目规则** | `./.codebuddy/rules/*.md` | **团队共享的模块化规则** | **是** |
| 项目本地记忆 | `./CODEBUDDY.local.md` | 个人的项目特定偏好 | 否（自动 .gitignore） |
| Auto Memory | `~/.codebuddy/memories/` | AI 自动保存的持久记忆 | 否 |

### 1.2 使用 /init 初始化项目记忆

首次在项目中使用 CodeBuddy CLI 时，运行：

```bash
/init
```

该命令会：
- 扫描项目结构，检测构建系统、测试框架和代码模式
- 生成初始的 `CODEBUDDY.md` 文件，包含项目的关键信息
- 建立项目知识图谱，提升后续对话的准确性和速度

**效果**：减少约 30-50% 的 Token 重复消耗，AI 响应更精准。

**何时重新运行**：项目结构发生重大变化时（如新增模块、大规模重构），建议 `/clear` 后重新 `/init`。

### 1.3 记忆加载顺序

启动 CodeBuddy 时，记忆按以下顺序加载：

```
1. 用户级：~/.codebuddy/CODEBUDDY.md + ~/.codebuddy/rules/
      ↓
2. 项目级：从当前目录向上递归加载所有 CODEBUDDY.md 和 CODEBUDDY.local.md
      ↓
3. 项目规则：仅加载当前目录的 .codebuddy/rules/（不向上递归）
      ↓
4. 子目录记忆：操作子目录文件时动态按需加载
      ↓
5. Auto Memory：MEMORY.md 前 200 行自动注入上下文
```

### 1.4 多层目录查找规则（重点）

这是团队使用中最容易混淆的点，请务必理解：

#### CODEBUDDY.md：向上递归，全部加载

```
/monorepo/                    ← CODEBUDDY.md ✅ 被加载
├── packages/
│   ├── frontend/             ← CODEBUDDY.md ✅ 被加载
│   │   └── src/              ← 你在这里打开 CodeBuddy
│   └── backend/
│       └── CODEBUDDY.md      ← ❌ 不加载（不在向上路径上）
```

在 `/monorepo/packages/frontend/src/` 打开 CodeBuddy 时：
- ✅ 加载 `/monorepo/CODEBUDDY.md`
- ✅ 加载 `/monorepo/packages/frontend/CODEBUDDY.md`
- ❌ **不加载** `/monorepo/packages/backend/CODEBUDDY.md`（兄弟目录不加载）

#### .codebuddy/rules/：仅当前目录，不递归

```
/monorepo/
├── .codebuddy/rules/         ← ❌ 不加载（不在当前目录）
├── packages/
│   └── frontend/
│       ├── .codebuddy/rules/ ← ❌ 不加载（不在当前目录）
│       └── src/
│           └── .codebuddy/rules/ ← ✅ 仅加载这里的规则
```

**核心不对称性**：`CODEBUDDY.md` 向上递归；`.codebuddy/rules/` 仅当前目录。

#### .codebuddy/ 目录完整内容与加载方式

`.codebuddy/` 目录除了 `rules/` 之外，还包含多种配置和扩展内容，各自有不同的加载方式：

| 目录/文件 | 用途 | 加载方式 | 是否提交 git |
|-----------|------|----------|-------------|
| `CODEBUDDY.md` | 项目主指令文件（等同于根目录的 `CODEBUDDY.md`） | **向上递归加载** | 是 |
| `rules/*.md` | 模块化规则文件 | **仅当前目录加载，不向上递归** | 是 |
| `skills/` | 技能定义（可复用的工作流，用 `/skill-name` 调用） | 当前目录加载 | 是 |
| `agents/` | 子代理定义（专用 AI 代理，独立上下文运行） | 当前目录加载 | 是 |
| `commands/` | 自定义斜杠命令 | 当前目录加载 | 是 |
| `settings.json` | 本地配置（权限、语言、hooks 等） | 当前目录加载 | **否** |
| `memories/` | Auto Memory 自动记忆存储 | 由系统按项目 ID 管理 | **否** |

> **说明**：`.codebuddy/CODEBUDDY.md` 和项目根目录的 `CODEBUDDY.md` 是**等价的**，二选一即可。使用 `.codebuddy/CODEBUDDY.md` 的好处是将所有 CodeBuddy 相关文件收拢到一个目录下，保持项目根目录整洁。

#### 建议

对于 monorepo 项目，**建议在项目根目录打开 CodeBuddy**，这样根目录的 `CODEBUDDY.md` 和 `.codebuddy/rules/` 都能正确加载。

---

## 二、CODEBUDDY.md 编写指南

### 2.1 编写原则：精简至上

> 写每一行时问自己："不写这行，CodeBuddy 会犯错吗？" 如果不会，就删掉。

| 应该写 | 不应该写 |
|--------|----------|
| CodeBuddy 猜不到的命令 | 读代码能搞清楚的东西 |
| 与默认习惯不同的风格约定 | 标准的语言约定 |
| 测试说明和首选运行器 | 详细的 API 文档（放链接即可） |
| 仓库规范（分支命名、MR 约定） | 经常变化的信息 |
| 项目特有的架构决策 | 逐文件的代码描述 |
| 开发环境的坑 | 长篇大论的解释 |

### 2.2 编写示例

```markdown
# MyProject

## 常用命令
- 基础设施部署：`terraform apply -var-file=env/prod.tfvars`（生产环境需先 `terraform plan` 确认变更）
- 服务发布：`kubectl apply -k overlays/prod/` （使用 kustomize 管理多环境配置）
- 日志排查：`stern -n prod "api-server.*" --since 30m`（实时聚合多 Pod 日志，排查问题首选）

## 代码风格
- Terraform 资源命名用下划线分隔，如 `tencent_vpc_main`
- K8s manifest 文件按资源类型放在 `k8s/{namespace}/{resource-type}.yaml`
- IMPORTANT: Secret 和敏感配置必须使用 Vault 或 Sealed Secrets，禁止明文写入 manifest

## 架构约定
- IaC 代码在 infra/，按模块拆分（network、compute、database）
- K8s 部署配置在 k8s/，使用 kustomize base + overlays 管理多环境
- 查看 @docs/architecture.md 了解完整架构设计
```

### 2.3 导入功能

CODEBUDDY.md 支持 `@path` 语法导入外部文件：

```markdown
查看 @README.md 了解项目概述，@package.json 了解可用命令。

# 附加规范
- 架构设计 @docs/architecture.md
- Git 工作流 @docs/git-workflow.md
- 个人偏好 @~/.codebuddy/my-preferences.md
```

- 支持相对路径和绝对路径
- 导入可递归，最大深度 5 层
- 代码块内的 `@` 不会触发导入

### 2.4 使用强调词提高遵守度

```markdown
## 安全要求
- IMPORTANT: 所有用户输入必须经过验证
- 必须：API 密钥从环境变量读取，禁止硬编码
- NEVER: 不要在日志中输出敏感信息
```

---

## 三、模块化规则管理

### 3.1 推荐的目录结构

```
your-project/
├── CODEBUDDY.md                 # 主要项目指令（精简）【提交 git】
├── CODEBUDDY.local.md           # 个人偏好【不提交】
├── .codebuddy/
│   ├── CODEBUDDY.md             # 等同于根目录的 CODEBUDDY.md（二选一）
│   ├── rules/                   # 模块化规则【提交 git】
│   │   ├── code-style.md        # 代码风格
│   │   ├── testing.md           # 测试规范
│   │   ├── security.md          # 安全要求
│   │   ├── frontend/
│   │   │   ├── react.md         # React 组件规范
│   │   │   └── styles.md        # 样式规范
│   │   └── backend/
│   │       ├── api.md           # API 设计规范
│   │       └── database.md      # 数据库规范
│   ├── skills/                  # 团队共享技能【提交 git】
│   │   └── fix-issue/
│   │       └── SKILL.md
│   ├── agents/                  # 团队共享子代理【提交 git】
│   │   └── security-reviewer.md
│   ├── commands/                # 自定义斜杠命令【提交 git】
│   ├── settings.json            # 本地配置（权限、语言、hooks）【不提交】
│   └── memories/                # Auto Memory 自动记忆【不提交】
```

### 3.2 跨项目共享规则

使用符号链接在多个项目间共享通用规则：

```bash
# 链接共享规则目录
ln -s ~/shared-codebuddy-rules .codebuddy/rules/shared

# 链接单个规则文件
ln -s ~/company-standards/security.md .codebuddy/rules/security.md
```

---

## 四、团队协作配置

### 4.1 .gitignore 推荐配置

```gitignore
# ========================================
# CodeBuddy CLI - Git 忽略配置
# ========================================

# 个人本地记忆（每个人的私有偏好，不提交）
CODEBUDDY.local.md

# 本地配置文件（包含个人权限、语言等设置）
.codebuddy/settings.json

# 自动记忆目录（AI 自动生成的持久记忆，个人专属）
.codebuddy/memories/
```

### 4.2 应该提交到 git 的文件

| 文件/目录 | 用途 | 备注 |
|-----------|------|------|
| `CODEBUDDY.md` | 项目级共享指令 | 团队核心配置 |
| `.codebuddy/rules/*.md` | 模块化项目规则 | 按主题拆分 |
| `.codebuddy/skills/` | 团队共享技能 | 可复用的工作流 |
| `.codebuddy/agents/` | 团队共享子代理 | 专用 AI 代理 |
| `.codebuddy/commands/` | 自定义命令 | 团队快捷操作 |

### 4.3 不应该提交的文件

| 文件/目录 | 原因 |
|-----------|------|
| `CODEBUDDY.local.md` | 个人偏好，如沙箱 URL、测试数据路径 |
| `.codebuddy/settings.json` | 个人配置，如语言、权限设置 |
| `.codebuddy/memories/` | AI 自动生成的个人记忆 |

---

## 五、工作流最佳实践

### 5.1 四阶段工作流

```
探索 → 规划 → 实现 → 提交
```

1. **探索**：进入计划模式，让 CodeBuddy 只读取文件、回答问题，不做修改
2. **规划**：让 CodeBuddy 输出详细的实现计划，审核后再执行
3. **实现**：切换回普通模式，按计划编码，边做边验证
4. **提交**：让 CodeBuddy 写提交信息并创建 MR

### 5.2 上下文管理

| 命令 | 作用 | 使用时机 |
|------|------|----------|
| `/clear` | 清空上下文 | 任务切换时 |
| `/compact` | 压缩上下文 | 对话过长时 |
| `Esc` | 中断当前操作 | 发现跑偏时 |
| `Esc + Esc` 或 `/rewind` | 回退到之前状态 | 需要撤销时 |

**核心原则**：上下文窗口是最重要的资源。填满后性能会显著下降。

### 5.3 避免常见的坑

| 坑 | 症状 | 解决方法 |
|----|------|----------|
| 无关上下文干扰 | 中间穿插不相关的问题 | 任务切换时 `/clear` |
| 反复纠正 | 纠正两次还不对 | `/clear` 后重新写更好的提示 |
| CODEBUDDY.md 太长 | 重要规则被忽略 | 精简，无用的删掉或转成 hook |
| 只信任不验证 | 代码看似正确但有边缘问题 | 始终提供验证手段（测试、脚本） |
| 无边界探索 | 上下文被填满 | 缩小范围或用子代理 |

---

## 六、调试记忆加载

### 6.1 使用 /memory 命令

运行 `/memory` 可以：
- 查看已加载的记忆文件列表
- 打开 Auto Memory 目录
- 切换 Auto Memory 开关

### 6.2 缓存与重载

| 操作 | 是否重新加载记忆 |
|------|-----------------|
| 重启 CodeBuddy | ✅ 是 |
| 通过 /memory 编辑 | ✅ 是（自动清缓存） |
| `/clear` | ❌ 否（仅清消息历史） |
| 手动修改记忆文件 | ❌ 否（需重启） |
| 新增/删除规则文件 | ❌ 否（需重启） |

> **提示**：手动修改了 `CODEBUDDY.md` 或规则文件后，需要重启 CodeBuddy 才能生效。

---

## 七、团队上手清单

### 新成员入职

- [ ] 安装 CodeBuddy CLI
- [ ] 配置个人语言偏好：`/config` → Language → 简体中文
- [ ] 了解项目的 `CODEBUDDY.md` 内容
- [ ] （可选）创建个人的 `~/.codebuddy/CODEBUDDY.md`，配置全局偏好
- [ ] （可选）创建 `CODEBUDDY.local.md`，配置个人的项目特定偏好

### 项目初始化

- [ ] 在项目根目录运行 `/init`，生成初始 `CODEBUDDY.md`
- [ ] 审核并精简生成的内容
- [ ] 按需创建 `.codebuddy/rules/` 模块化规则
- [ ] 配置 `.gitignore`（参考第四节）
- [ ] 将 `CODEBUDDY.md` 和 `.codebuddy/rules/` 提交到 git

### 日常使用

- [ ] 定期审查和更新 `CODEBUDDY.md`（像对待代码一样）
- [ ] 遇到 CodeBuddy 反复犯的错误 → 加到 CODEBUDDY.md
- [ ] CodeBuddy 已经自动做对的事情 → 从 CODEBUDDY.md 删掉
- [ ] 团队成员共同维护项目记忆

---

## 附录：常用命令速查

| 命令 | 作用 |
|------|------|
| `/init` | 初始化项目，生成 CODEBUDDY.md |
| `/clear` | 清空对话上下文 |
| `/compact` | 压缩对话上下文 |
| `/memory` | 管理记忆（查看、编辑、切换） |
| `/config` | 管理配置（语言、权限等） |
| `/permissions` | 配置工具权限白名单 |
| `/hooks` | 配置自动化钩子 |
| `/rewind` | 回退到之前的对话状态 |
| `/rename` | 给当前会话命名 |
| `/resume` | 列出或恢复历史会话 |
| `Esc` | 中断当前操作 |
| `Esc + Esc` | 打开回退菜单 |

---

*本文档基于 CodeBuddy CLI 官方文档整理，建议团队成员结合实际使用体验持续更新。*
