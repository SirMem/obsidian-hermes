# Git 开发流程：feature 分支 vs 直接在 main 开发

创建时间：2026-04-13 15:40:34 CST
状态：未整理

## 原问题
在一个项目的开发过程中，如果有 feature，是在 main 分支中将内容切到一个新分支中好，还是直接通过 worktree 的 main 开发，开发完成后通过 merge 或 cherry-pick 更好？

## 结论
更推荐：每个 feature 都从 main 拉一个新分支开发，而不是直接在 worktree 的 main 上开发，完成后再通过 merge 或 cherry-pick 补救整理。

一句话结论：
1. 默认方案：feature branch + 可选 worktree
2. 不推荐：在 main 上直接开发，再想办法摘提交

## 拆开看：这是两个维度
最好拆成两个问题：
1. 分支策略
2. 是否使用 worktree

它们不是二选一。通常最合理的组合是：
- main 保持干净
- feature 在新分支开发
- 如有需要，用 worktree 同时开多个工作目录

## 为什么不建议直接在 main 上开发

### 1. main 容易被污染
main 通常应该代表：
- 可发布
- 相对稳定
- 团队共享基线

如果直接在 main 开发，中间态提交往往会是：
- 不完整
- 不能跑
- 改到一半
- 带实验性代码

这样 main 的语义会变得混乱。

### 2. 后续整理成本更高
如果先在 main 上开发，完成后再考虑 merge 或 cherry-pick，通常会出现：
- 实验提交很多，不容易整理
- cherry-pick 需要挑提交、排顺序、补遗漏
- 如果提交混杂多个 feature，会更难拆

本质上是把前面的省事变成后面的额外成本。

### 3. 更容易误推远程 main
如果本地 main 跟踪远程 main：
- 一不小心就可能 git push
- CI 可能跑在半成品上
- 别人也可能基于被污染的 main 开发

团队环境里风险更高。

### 4. 不利于 code review
最舒服的 review 方式通常是：
- 一个分支
- 一组围绕单一 feature 的提交
- 最后发一个 PR/MR

如果先在 main 开发、再摘提交，review 上下文通常更碎。

## 为什么“feature branch + worktree”通常最好
worktree 解决的是“同时切多个上下文很麻烦”的问题；branch 解决的是“提交历史和协作边界”的问题。

所以它们不是替代关系，而是互补关系。

推荐姿势：
- main 分支：始终保持干净，只做同步
- 新 feature：从最新 main 切 feature branch
- 如果要并行做多个 feature / bugfix，就给每个分支开一个 worktree

例如：
- repo/ -> main
- ../repo-feature-a -> feature/a
- ../repo-bugfix-login -> fix/login-timeout

这样做的好处：
1. main 永远是干净基线
2. 每个目录对应一个任务，上下文清晰
3. 不需要频繁 stash / checkout
4. 分支天然隔离，提交历史干净
5. review、回滚、重做都容易

## merge 和 cherry-pick 应该怎么用

### merge
适合正常 feature 完成后并回主线。这是标准路径。

### cherry-pick
适合少量、明确、临时摘取提交，例如：
- 从另一个分支摘一个 hotfix
- 把某个独立提交移植到 release 分支
- 把误提交在错误分支上的少数提交捞出来

但不适合作为日常主流程。
如果经常靠 cherry-pick 来整理 feature，通常说明前面的分支习惯有问题。

## 什么时候直接在 main 上开发还能接受
只有少数个人场景可以勉强接受：
- 纯个人仓库
- 改动极小
- 不需要 review
- 明确知道自己不会误推
- main 本身不是稳定基线，只是个人试验场

例如：
- 改一个本地脚本
- 记一条小配置
- 5 分钟的一次性修复

即便如此，只要它已经算一个 feature，仍然更建议开分支。

## 默认决策规则

### 情况 A：正式 feature / 超过半小时 / 多个提交 / 可能回滚 / 可能 review
=> 一律新分支开发
=> 需要并行时再配 worktree

### 情况 B：很小的本地一次性改动
=> 可以直接在当前分支改
=> 但如果当前分支是 main，仍建议谨慎

### 情况 C：并行多个任务
=> 不要在一个 worktree 里来回切
=> 用多个 worktree，各自绑定不同分支

## 推荐结论
最佳实践：
1. main 只用来同步和保持稳定
2. 每个 feature 从 main 切新分支
3. 需要并行开发时，用 worktree 承载这些分支
4. 完成后优先通过正常 merge / rebase + merge / squash merge 回 main
5. cherry-pick 只作为补救或局部移植手段，不作为默认开发流

## 单人开发的最实用版本（简版）
最省脑子的流程：
- main 永远不直接写功能
- 来一个需求就先更新 main，然后切 feature 分支
- 如果手头已有别的任务在做，就用新的 worktree 承载新分支
- 做完后自测、整理提交，再 merge 回 main

一句话总结：
分支是“历史隔离”，worktree 是“工作区隔离”；单人开发里通常要优先保证的是“新分支开发”，而不是“先在 main 开发、之后再补救”。
