---
title: Hermes Memory System 学习笔记
date: 2026-04-13
tags:
  - Hermes
  - Memory
  - Agent
  - Obsidian
  - 学习笔记
category: 工具/Hermes
status: 整理中
source:
  - https://hermes-agent.nousresearch.com/docs/user-guide/features/memory
---

# Hermes Memory System 学习笔记

## 我先用一句话概括

Hermes 的 memory system 不是“无限长期记忆”，而是一套“有容量上限、人工筛选、跨会话持久化”的记忆机制，用来把最关键、最稳定、最值得长期保留的信息，持续注入到之后的会话里。

它的核心不是“记得越多越好”，而是“只记那些能减少未来重复沟通成本的高价值信息”。

## 1. Hermes 的记忆由什么组成

官方文档里，Hermes 的内建持久记忆主要由两部分组成：

1. MEMORY.md
   - 存 agent 自己的工作笔记
   - 偏环境事实、项目约定、工具坑点、经验教训

2. USER.md
   - 存用户画像
   - 偏用户偏好、沟通风格、工作习惯、身份信息

文档明确写到：
- 这两个文件存放在 ~/.hermes/memories/
- 每次新会话开始时，会被读取并注入 system prompt
- 注入进去的是“冻结快照”而不是实时联动视图

也就是说：
- 改 memory 是立即持久化到磁盘的
- 但本次会话里 system prompt 不会动态刷新
- 要到下一次会话启动时，新的记忆才会作为 prompt 前缀重新出现

这是一个非常重要的设计点。

## 2. 为什么要做“冻结快照”

文档给出的原因是：

- 保持 prefix cache 稳定
- 提升性能
- 避免 system prompt 在会话中途不断变化

我的理解是：

如果 memory 一改，system prompt 就立刻变，那模型上下文前缀就会不断失效，影响效率和稳定性。所以 Hermes 选择：

- 会话开始时一次性装载记忆
- 会话过程中允许 memory 工具继续增删改
- 但当前轮的“系统前缀”不跟着热更新

因此要区分两个层面：

1. 存储层已经更新
2. 当前会话可见的 prompt 快照还没更新

这也解释了为什么文档说：
- system prompt 里看到的是 frozen snapshot
- tool response 显示的才是 live state

## 3. memory 工具能做什么

Hermes 用 memory 工具管理持久记忆，只有 3 个动作：

- add：新增一条记忆
- replace：替换已有记忆
- remove：删除已有记忆

注意：
- 没有 read 动作
- 因为“读取记忆”这件事默认通过 system prompt 注入完成
- agent 在会话开头天然就能看到这些记忆

### replace / remove 的关键机制：substring matching

不是必须拿整条旧内容去改，而是用 old_text 提供一个“能唯一定位该条记忆的短子串”。

如果这个子串：
- 只匹配到一条 → 成功
- 匹配到多条 → 会报错，需要给更具体的 old_text

这个设计很实用，因为真实使用时，agent 很少会机械地复制整段记忆再覆盖，而更可能按关键词更新。

## 4. 两类 target 的边界

### target = memory

这是 agent 的个人工作记忆，更适合存：

- 环境事实
  - 例如系统版本、安装了什么工具、目录结构
- 项目约定
  - 例如代码风格、测试命令、分支规范
- 工具 quirks
  - 例如某命令在这个环境里会失败、某服务要走特殊端口
- 经验教训
  - 例如某 bug 的稳定规避方式

### target = user

这是用户画像，更适合存：

- 用户是谁
- 用户偏好什么表达风格
- 用户讨厌什么做法
- 用户习惯如何协作
- 用户技术背景与熟练度

### 我对这两个 target 的理解

可以粗暴地记成：

- memory = 关于“世界/环境/工作对象”的稳定事实
- user = 关于“这个人”的稳定事实

这个边界很重要，因为如果混着记：
- 用户偏好会被环境事实淹没
- 环境约定会被个体画像污染
- 后续压缩和替换会更难做

## 5. Hermes 认为“应该主动保存”的信息

文档强调：agent 应主动保存，而不是等用户说“帮我记住”。

推荐主动保存的主要有：

1. 用户偏好
   - 例如“我更喜欢 TypeScript 而不是 JavaScript”

2. 环境事实
   - 例如“这台机器是 Debian 12，数据库是 PostgreSQL 16”

3. 用户纠正过 agent 的地方
   - 例如“不要用 sudo 跑 docker，用户已经在 docker group 里”

4. 项目惯例
   - 例如“本项目用 tabs、120 列、Google 风格 docstring”

5. 已完成的重要工作
   - 例如“数据库已在某日从 MySQL 迁移到 PostgreSQL”

6. 用户显式要求长期记住的事情

### 我觉得这里最值得注意的一点

Hermes 官方文档把“纠正”和“约定”看得非常重。

这很合理，因为真正能减少未来摩擦的，往往不是大而空的信息，而是：

- 下次别再犯这个错
- 下次直接按这个约定来
- 下次不要再问我同样的问题

## 6. Hermes 明确建议不要存什么

文档列出的“不该存”的内容包括：

- 太琐碎、太显然的信息
- 很容易重新查到的事实
- 原始数据大块复制
- 临时性的会话上下文
- 已经存在于 context files 里的内容（例如 SOUL.md、AGENTS.md）

### 这背后的原则

记忆位很贵，所以不能把它当日志仓库，也不能当百科缓存。

如果一个信息：
- 未来高概率会再次用到
- 且用户不想每次重复解释
- 且不容易在当前上下文即时获得

那才值得进入 memory。

反过来，如果只是：
- 本轮调试的临时路径
- 一大段日志
- 一次性报错堆栈
- 通用知识

那就不该塞进 memory。

## 7. 容量管理：这套系统是强约束的

文档给出的默认上限：

- memory：2200 chars
- user：1375 chars

这不是“建议值”，而是硬约束式设计思路。

官方给出的典型容量大致是：
- memory：8-15 条
- user：5-10 条

所以 Hermes 的 memory system 本质上是：

- 小容量
- 高密度
- 强筛选
- 重压缩

### 满了怎么办

如果新增会超限，工具会报错，并返回：
- 当前使用量
- 当前已有条目
- 新增条目会超多少

此时 agent 应该：

1. 检查现有条目
2. 找能删掉的
3. 找能合并压缩的
4. 用 replace 做整合
5. 再 add 新内容

### 官方推荐的最佳实践

当 memory 超过 80% 时，就应该开始主动压缩和合并，而不是等爆掉再处理。

这个思路很像：
- 不是“仓库越堆越大”
- 而是“持续整理成高信息密度摘要”

## 8. 什么算“好的 memory entry”

文档特别强调：

好记忆应该是 compact, information-dense。

也就是：
- 短
- 但信息量大
- 且可操作性强

例如好的条目会把：
- 系统环境
- 工具栈
- shell
- 编辑器偏好

压在一条里，而不是拆成很多零散小句子。

### 我总结出的好条目标准

1. 稳定
   - 不容易明天就失效

2. 可行动
   - 下次看到能直接指导行为

3. 高复用
   - 多轮协作里都会受益

4. 高密度
   - 尽量一条里打包相关事实

5. 不写流水账
   - 不要写“某天用户让我看了某项目，然后我发现……”这种长叙事

## 9. 重复项与安全扫描

文档还提到两个机制：

### 9.1 Duplicate Prevention

如果完全重复添加已有内容，系统会拒绝重复写入，并返回类似“no duplicate added”的成功信息。

这说明 Hermes 不希望 memory 膨胀成同义重复集合。

### 9.2 Security Scanning

memory 条目在写入前会做安全扫描，因为这些内容最终会进入 system prompt。

会被拦截的内容包括：
- prompt injection 模式
- credential exfiltration 模式
- SSH backdoor 等危险内容
- invisible Unicode 字符

这一点很关键，因为 memory 一旦被污染，相当于把污染源持续注入每个后续会话。

## 10. session_search 和 memory 不是一回事

文档特别区分了两套系统：

### memory

特点：
- 小而精
- 永远在 system prompt 里
- 每轮开局就带着
- 适合关键事实

### session_search

特点：
- 搜历史对话
- 容量近似无限
- 按需查询
- 更像“长期档案馆”而不是“工作记忆”

文档里还明确写到：
- 所有 CLI 和 messaging sessions 存在 SQLite：~/.hermes/state.db
- 通过 FTS5 做全文搜索
- 搜到后会返回总结结果

### 我的理解

可以把两者类比成：

- memory = 大脑里随手就能调出的少量关键事实
- session_search = 去档案室翻旧案卷

因此：
- 高频、稳定、关键的信息 → memory
- 偶尔要回忆“之前我们聊过什么” → session_search

这也是为什么 Hermes 文档强调：
memory 是 curated 的，session history 是 automatic 的。

## 11. 配置项

文档给出了 ~/.hermes/config.yaml 中的 memory 配置示例：

```yaml
memory:
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 2200
  user_char_limit: 1375
```

说明可以控制：
- 是否启用 memory
- 是否启用 user profile
- 两边字符上限

## 12. 外部 memory providers 是“并行增强层”

官方文档最后提到，Hermes 还有外部 memory provider 插件，例如：

- Honcho
- OpenViking
- Mem0
- Hindsight
- Holographic
- RetainDB
- ByteRover
- Supermemory

关键点不是“替代内建 memory”，而是：

- alongside built-in memory
- 也就是与内建 memory 并行存在

它们提供更深层的能力，例如：
- 语义搜索
- 自动事实抽取
- 知识图谱
- 更强的跨会话用户建模

我的理解是：
内建 memory 负责“短小稳定的核心提示层”，外部 provider 负责“更深更广的长期记忆层”。

## 13. 这篇文档回答了我最关心的几个问题

### Q1. Hermes 的记忆是不是一个真实文件？

是。官方文档明确说：
- MEMORY.md
- USER.md
- 都保存在 ~/.hermes/memories/

### Q2. 为什么我改了记忆，本轮 prompt 里没立刻变化？

因为 system prompt 用的是 frozen snapshot。

### Q3. memory 工具为什么没有 read？

因为默认读取是靠会话启动时注入 prompt 完成的。

### Q4. session_search 和 memory 的差别是什么？

- memory：少量关键事实，常驻上下文
- session_search：所有历史对话，按需回忆

### Q5. memory system 的设计哲学是什么？

不是尽可能多记，而是：
- 只记高价值
- 记稳定事实
- 记可行动约束
- 持续压缩
- 防止污染

## 14. 我自己给出的使用准则

如果以后我要判断“某信息值不值得进入 Hermes memory”，我会用下面这个四问法：

1. 这是不是稳定信息？
2. 这会不会在未来显著减少重复沟通？
3. 这是不是比临时上下文更值得长期保留？
4. 这条信息能不能压缩成高密度、可执行的一句话？

如果四个里大多数都满足，就值得存。

## 15. 一句话总结

Hermes 的 memory system 本质上不是“自动帮你记住一切”，而是“帮 agent 维护一个小而精、跨会话复用、可持续整理的核心认知层”。

它像一个受预算约束的长期工作记忆，而不是无限扩容的资料库。
