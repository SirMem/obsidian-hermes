# 单人开发最顺手的 Git branch + worktree 工作流

创建时间：2026-04-13 15:51:36 CST
状态：未整理

## 核心结论
推荐默认采用这套规则：
1. 主仓库目录固定放 main
2. main 只负责同步、查看主线状态、合并已完成分支
3. 每个 feature / bugfix / spike 都新开分支
4. 如果同一时间只做一个任务，可以直接在当前仓库切分支
5. 如果同一时间并行两个及以上任务，每个任务一个 worktree
6. feature 完成后，自测、整理 commit，再 merge 或 squash merge 回 main
7. cherry-pick 只用于补救，不作为默认主流程

一句话总结：
main 是“稳定基线目录”，branch 是“任务边界”，worktree 是“并行上下文容器”。

## 推荐的目录组织
假设主仓库名为 `project`：
- `~/code/project` -> 主 worktree，固定在 `main`
- `~/code/project-feat-auth` -> `feat/auth`
- `~/code/project-fix-cache` -> `fix/cache-bug`
- `~/code/project-spike-xxx` -> `spike/xxx`

建议：主目录永远保留给 `main`，不要让它承担具体开发任务。

这样做的好处：
1. 一进主目录就知道这里是干净基线
2. 其他目录名已经表达当前任务，无需额外判断分支
3. 删除已完成任务目录更直观
4. 心智负担更低

## 默认工作流

### 场景 1：开始一个新 feature
步骤：
1. 先更新 `main`
2. 从 `main` 创建 feature 分支
3. 如果这是当前唯一任务，可以直接在当前仓库切分支
4. 如果已经有别的任务在做，则新建 worktree

#### 情况 A：当前只做这一个任务
适合：
- 当前没有别的并行任务
- 任务比较短
- 不需要保留 `main` 的独立工作区

流程：
- `git switch main`
- `git pull`
- `git switch -c feat/xxx`

#### 情况 B：已有其他任务，或想始终保持 main 可用
更推荐长期采用。

流程：
- `git switch main`
- `git pull`
- `git worktree add ../project-feat-xxx -b feat/xxx main`

效果：
- 创建一个新目录
- 创建一个新分支
- 基于 `main` 生成独立工作区

## 为什么“主目录固定 main”很省脑子
它建立了非常稳定的心智模型：
1. 想看最新主线状态 -> 去主目录
2. 想开始新任务 -> 从主目录开新 branch / worktree
3. 想删除已完成任务目录 -> 删除对应 worktree
4. 想知道某目录在做什么 -> 看目录名即可

这比在一个目录里反复 checkout 不同分支更不容易混乱。

## 建议的分支命名规则
尽量简单一致：
- `feat/xxx`
- `fix/xxx`
- `refactor/xxx`
- `chore/xxx`
- `spike/xxx`

例如：
- `feat/user-login`
- `fix/token-refresh`
- `refactor/api-client`
- `spike/rag-memory-layout`

好处：
1. 一眼能看出目的
2. worktree 目录容易对应命名
3. 提交历史和 PR 标题更清晰

## 并行开发时为什么 worktree 很值钱
比如你正在做：
- `feat/editor`

突然插入一个线上 bug：
- `fix/login-timeout`

不推荐做法：
- stash
- 切 `main`
- 再切 bugfix 分支
- 修完后再切回来
- 恢复现场

推荐做法：
1. 主目录保持 `main`
2. `project-feat-editor` 保持原样
3. 新开一个目录 `project-fix-login-timeout`

结果：
- 一个 feature 的完整现场保留着
- 一个 bugfix 的完整现场独立处理
- 不用 stash
- 不会互相污染
- 不容易把提交打错分支

## feature 完成后的标准收尾 checklist
建议固定执行：
1. 跑测试
2. 看 diff
3. 整理 commit
4. 合回 `main`
5. 删除分支 / 删除 worktree

## merge 风格建议
单人开发通常有两种常用风格：

### 方式 A：保留分支提交历史
适合：
- 分支提交本身已经很整洁
- 你希望保留开发过程

### 方式 B：squash merge
适合：
- 中间有很多 `wip / fix / cleanup`
- 你希望 `main` 历史更干净

对单人开发而言，更推荐：
“分支内自由提交，合并到 `main` 时偏向 squash”。

这样可以同时满足：
- 开发时不用压抑提交频率
- 主线历史依然清爽

## 我推荐的实际节奏
1. 平时保持一个主目录：`project`
   - 永远在 `main`
   - 主要负责同步主线
2. 来新任务时：
   - 小任务且当前空闲：可以直接在当前仓库切分支
   - 只要有并行任务趋势：直接开 worktree
3. 对任何超过半小时的改动：
   - 一律不要直接在 `main` 写
4. 分支上可以高频 commit：
   - `wip`
   - `checkpoint`
   - `fix`
   - `refactor`
5. 合并前再统一整理：
   - `rebase -i`
   - 或直接 `squash merge`
6. 合并完成后：
   - 回主目录更新 `main`
   - 删除完成任务的 worktree 和分支

## 什么时候可以不用 worktree
以下情况可以不强制使用 worktree：
1. 当前只有一个任务
2. 任务持续时间很短
3. 不需要同时保留 `main` 工作现场
4. 不介意在一个目录里切分支

也就是说：
- branch 是强建议
- worktree 是并行开发增强器

如果当前只有一个任务，完全可以直接：
- `git switch main`
- `git pull`
- `git switch -c feat/xxx`

## 什么时候强烈建议用 worktree
以下情况强烈建议直接使用 worktree：
1. 同时做两个任务以上
2. feature 过程中插入 hotfix
3. 需要保留另一个分支的现场
4. 项目启动成本高，不想反复切分支后重新准备环境
5. 容易 stash 忘记恢复，或者误提交

## cherry-pick 在这套流里的定位
在这套工作流里，`cherry-pick` 不是主角，而是补救工具。

适合场景：
1. 某个修复误提交到了错误分支
2. 某个 hotfix 需要进入多个分支
3. 某个独立提交值得单独摘出

不适合：
- 整个 feature 靠 cherry-pick 搬运
- 先在 `main` 上开发，再靠 cherry-pick 整理历史

如果经常依赖 cherry-pick 来整理 feature，通常意味着前面的分支习惯有问题。

## 默认判断器
以后可以直接用这套判断规则：
1. 这是 feature / bugfix / refactor 吗？
   - 是：开新分支
2. 当前有别的任务正在进行吗？
   - 有：开新 worktree
   - 没有：可直接切分支，或也开 worktree
3. 这个任务会有多个提交吗？
   - 会：绝不直接在 `main` 写
4. 合并回 `main` 时历史要不要干净？
   - 要：优先 `squash merge`

因此默认策略就是：
- `main` 不开发功能
- 功能在 branch 上开发
- 并行任务用 worktree 隔离
- 回 `main` 时优先 squash / merge
- `cherry-pick` 仅补救

## 我认为“最顺手”的最终版本
最省脑子的个人标准流：
1. 主目录永远是 `main`
2. 新任务默认：
   - `git worktree add ../project-feat-xxx -b feat/xxx main`
3. 在新目录里开发
4. 分支内随手 commit
5. 完成后整理并 squash 到 `main`
6. 删除该 worktree
7. 回到主目录继续下一件事

这是单人开发里最清晰、最不容易乱、同时保留灵活度的方式。

## 可复制思路（场景化）
后续可继续补充为命令清单，至少包括：
1. 开新 feature
2. 中途插入 hotfix
3. 合并完成后清理
4. 误提交到错误分支时如何补救
