---
title: AgentOps 学习笔记
date: 2026-03-31
tags:
  - AgentOps
  - Claude-Code
  - git工作流
  - AI工程化
category: 编程/工具
difficulty: 进阶
status: 完成
---

# AgentOps 学习笔记

> 核心思想：Agent 会话是一次性的，环境才是永久的。

## 一、专业 Git 工作流

没有工程经验时的做法：写完代码 → commit → push，把 GitHub 当网盘用。

专业工作流循环：

```
Issue（确定要做什么）
  ↓
切出新分支
  ↓
小步 commit（每完成一个逻辑单元就存档）
  ↓
push 分支
  ↓
CI 自动跑测试
  ↓
开 PR + Review
  ↓
合并，Issue 关闭
```

- **Issue**：任务卡片，描述要做什么，还没开始
- **Branch**：隔离工作空间，不在 main 上直接改
- **Commit**：小粒度存档，方便回滚，不是「做完了才提交」
- **PR**：申请合并 + 代码 Review 入口
- **CI/CD**：push 后自动跑测试，不通过不让合并
- **Worktree**：进阶技巧，同时在多个分支工作

---

## 二、AgentOps 是什么

**一句话**：让 AI 编程助手不再「失忆」，每次会话都能越来越聪明。

**解决的根本问题**：Claude Code 等 AI 工具每次开新会话就完全忘记上次的一切。

### 三个核心缺口

| 缺口 | 问题 | 解决方案 |
|------|------|----------|
| 判断力验证缺失 | AI 写完代码就直接提交，没有风险评估 | `/vibe`、`/council` 提交前审查 |
| 知识不能积累 | 同一个坑踩了又踩 | `.agents/` 知识库 + `/forge` 锻造经验 |
| 循环没有闭合 | 做完的工作没产生更好的下一步 | `/post-mortem` 总结 + 自动生成后续建议 |

### 棘轮模型

```
多个 AI 并行干活（混乱）
  ↓
多模型委员会审查（过滤）
  ↓
通过结果锁进 main（棘轮锁死，永不倒退）
```

---

## 三、.agents/ 知识飞轮

```
你的项目/
├── src/
└── .agents/
    ├── learnings/   # 历史教训和经验
    ├── goals/       # 当前目标
    ├── findings/    # 发现的问题
    ├── context/     # 项目背景
    └── artifacts/   # 各阶段产出
```

**飞轮运转方式**：

```
会话开始 → ao lookup 检索相关知识注入上下文
  ↓
Claude 干活（已知历史教训）
  ↓
会话结束 → /post-mortem 总结
  ↓
/forge 锻造成结构化知识写入 .agents/
  ↓
下次会话，飞轮继续转，Claude 越来越懂项目
```

**`ao lookup`**：按需检索，不是全量注入，只取最相关的 3-5 条，精准补充上下文。

---

## 四、Skills 系统

Skills 是给 Claude 扩展的「技能包」，每个 Skill 是带有明确契约的工作流命令。

### 三大类 Skills

**探索类（想清楚再动手）**
- `/brainstorm` → 发散想法
- `/research` → 理解代码库
- `/plan` → 拆成 Issue 清单
- `/pre-mortem` → 提前预测风险

**执行类（真正写代码）**
- `/implement` → 做单个 Issue
- `/crank` → 批量做多个 Issue（自动波次）
- `/swarm` → 多 Agent 并行
- `/rpi "目标"` → 一条命令跑完全流程（懒人模式）

**验证类（保证质量）**
- `/vibe` → 提交前 8 维度全面检查（安全/质量/架构/测试...）
- `/council` → 多模型 PR Review
- `/post-mortem` → 任务结束后总结
- `/forge` / `/retro` → 把经验写入知识库

### Skill 的本质：SKILL.md 契约

每个 Skill 是一个目录，`SKILL.md` 定义了：
- 输入是什么
- 输出写到哪个文件
- 执行步骤顺序

Claude 严格按契约执行，不乱发挥。

### 两种运行模式

| 类型 | 代表 Skills | 行为 |
|------|------------|------|
| 编排者（NO-FORK） | `/rpi` `/crank` `/vibe` | 主会话运行，全程可见可干预 |
| 工人（FORK） | `/council` 的 workers | 分叉子 Agent，结果汇总回来 |

### 快速记忆

```
不知道做什么  →  /brainstorm
理解代码库   →  /research
拆任务       →  /plan
写代码       →  /implement 或 /crank
提交前检查   →  /vibe
任务收尾     →  /post-mortem
懒人一键     →  /rpi "目标"
```

---

## 五、Hooks 机制

Hook = 在特定事件发生时自动触发脚本，无需手动调用。

### 两层 Hooks

- **Claude Code 运行时 Hooks**（hooks.json）：控制 Claude 干活前后的行为
- **Git Hooks**（install-dev-hooks.sh）：控制 commit/push 前后的行为

### 运行时 Hook 事件表

| 事件 | 触发时机 | 做什么 |
|------|---------|--------|
| `SessionStart` | 会话开始 | 读取 .agents/ 注入历史知识，检查未完成任务 |
| `UserPromptSubmit` | 每次发消息 | 意图识别，推荐合适的 Skill |
| `PreToolUse` | Claude 用工具前 | 安全门禁：风险评估通过了吗？分支对吗？ |
| `PostToolUse` | Claude 用完工具后 | 质量检查：代码复杂度、静态分析、死循环检测 |
| `TaskCompleted` | 任务完成 | 验收：编译通过？测试通过？Issue 关闭了？ |
| `SessionEnd` | 会话结束 | 知识归档，整理 .agents/ |
| `Stop` | Claude 被停止 | 安全关闭飞轮，保存进度 |

### PreToolUse 关键检查（最重要）

- `pre-mortem-gate.sh`：没做风险评估？阻止执行
- `commit-review-gate.sh`：没跑 /vibe？阻止 git commit
- `git-worker-guard.sh`：防止 Claude 误操作 main 分支
- `edit-knowledge-surface.sh`：编辑 SKILL.md 等核心文件时强制提醒

### 关于 Windows 兼容性

Hooks 调用的是 `.sh` shell 脚本，**Claude Code 本身运行在 WSL 或 macOS/Linux 环境中**，不是 Windows 原生。所以：
- 在 WSL 里用：完全兼容，.sh 正常跑
- Windows 原生 PowerShell：不兼容，需要用 WSL
- 结论：AgentOps 的设计目标平台是 Unix 环境，Windows 用户需要通过 WSL 使用

---

## 六、完整工作流对应关系

```
/plan 生成 Issue
  ↓
/implement 或 /crank 写代码
  ↓
/vibe 提交前自查（PreToolUse hook 自动触发）
  ↓
push → CI 跑验证（.github/workflows/validate.yml）
  ↓
/council 多模型 PR Review
  ↓
合并进 main（棘轮锁死）
  ↓
/post-mortem → /forge 锁入知识库
  ↓
下次会话，知识自动注入，Claude 越来越懂项目
```

---

## 相关链接

- 项目地址：https://github.com/boshu2/agentops
- 新人指南：docs/newcomer-guide.md
- 架构文档：docs/ARCHITECTURE.md
- Skills 参考：docs/SKILLS.md
