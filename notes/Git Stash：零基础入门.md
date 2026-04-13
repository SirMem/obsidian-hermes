---
createTime: 2026-04-13 20:25:40 CST
updateTime: 2026-04-13 20:25:40 CST
tags:
  - git
  - stash
  - version-control
  - workflow
  - beginner
belongCanvas:
  - "[[Git.canvas]]"
  - "[[Git Stash.canvas]]"
status: growing
aliases:
  - git stash
  - Git stash
  - 暂存未提交修改
---

# Git Stash：零基础入门

> [!abstract] 一句话摘要
> `git stash` 的本质是：把当前“还没提交、但又不想丢掉”的修改临时收起来，等处理完别的事情后再恢复；它适合短期切任务，不适合作为长期管理改动的主流程。

---

## 一、为什么会需要 stash？

有时候项目你正改到一半，突然需要：
1. 切到别的分支
2. 先去修一个紧急 bug
3. 拉最新代码
4. 临时试别的方案

但此时你的工作还没完成：
- 不想提交，因为改了一半
- 不想丢掉，因为之后还要继续

这时，`git stash` 就像：
“把你桌上摊开的草稿，先临时收进抽屉里，等会儿再拿出来继续。”

一句话理解：
`git stash` 是 Git 提供的“临时收起未完成修改”的功能。

---

## 二、stash 的机制

默认情况下，`git stash` 主要会把这两类东西收起来：
1. 已跟踪文件中的已修改但未提交内容
2. 暂存区（stage/index）里的内容

执行后，工作区通常会变回接近“干净”的状态。

可以先把 3 个区域粗略理解成：
- 工作区：你正在编辑的文件
- 暂存区：你准备下次提交的文件
- 提交历史：已经正式提交的版本

而 stash 作用在前两者上：
- 把“工作区 + 暂存区”的修改临时存起来
- 让你先去做别的事

核心直觉：
- 不是“旧修改被忽略”
- 而是“旧修改先从当前工作区被拿走了”
- 等你执行 `pop` / `apply` 时，Git 再尝试把它合并回来

---

## 三、最小使用案例

把 stash 记成这句话就够了：
“先藏起来，之后再恢复。”

最常见流程只有 3 步：
1. 正在改代码，改到一半
2. `git stash`
3. 事情处理完后，`git stash pop`

最小命令组合：

```bash
git stash
git stash pop
```

---

## 四、最常见的使用场景

### 1. 改到一半，突然要切去修 bug

你正在做功能 A，改了一半。
这时老板说：先修一下线上 bug。

如果你现在直接切分支，Git 可能会：
- 不让你切
- 或者切过去后把当前修改也带过去，造成污染

这时你可以：
1. `git stash`
2. 切到 bug 分支处理问题
3. 修完回来
4. `git stash pop`

这样你原来的“半成品现场”就能回来。

### 2. 想拉最新代码，但当前改动不适合提交

你本地有一堆没做完的改动，但想先同步远程。

你可以：
1. `git stash`
2. `git pull`
3. `git stash pop`

### 3. 想快速试一个方向，但不确定要不要保留

你想试试一个临时方案，不想马上 commit。
可以先 stash 当前现场，再试别的东西，之后再恢复。

---

## 五、最基础命令

### 1. 临时收起当前修改

```bash
git stash
```

作用：
- 临时保存当前未提交修改
- 让工作区尽量恢复干净

### 2. 查看 stash 列表

```bash
git stash list
```

你可能会看到类似：

```bash
stash@{0}: WIP on main: abc1234 add login page
stash@{1}: WIP on feat/search: def5678 fix ui
```

意思是：
- 你不止能 stash 一次
- Git 会把它们排成一个栈
- `stash@{0}` 是最新的一次

### 3. 恢复最近一次 stash，并把它从列表里删除

```bash
git stash pop
```

作用：
- 把最近一次 stash 恢复到当前工作区
- 恢复成功后，从 stash 列表删掉这条记录

可以理解成：
“拿出来并顺手把抽屉里的记录清掉。”

### 4. 恢复最近一次 stash，但不删除记录

```bash
git stash apply
```

作用：
- 恢复 stash
- 但保留 stash 记录

一句话区分：
- `pop` = 恢复并删除
- `apply` = 恢复但保留

### 5. 删除某条 stash

```bash
git stash drop stash@{0}
```

### 6. 清空所有 stash

```bash
git stash clear
```

注意：
这通常不可逆，谨慎使用。

---

## 六、一个基础流程例子

假设你正在 `main` 上改代码：

```bash
git status
```

看到：

```bash
modified: app.js
modified: styles.css
```

此时突然要去处理别的事。

### 第一步：stash 当前修改

```bash
git stash
```

### 第二步：查看是否已经存进去

```bash
git stash list
```

### 第三步：去做别的事

比如切分支：

```bash
git switch fix/login-bug
```

### 第四步：回来后恢复

```bash
git stash pop
```

然后你原来改到一半的 `app.js` 和 `styles.css` 就会回来。

---

## 七、一个更直观的代码例子

下面这个例子专门回答一个常见疑问：
执行 `git stash` 之后，后面做新任务时，之前修改过的位置是不是会被“忽略”？

答案是：
不是“被忽略”，而是“之前的修改被临时拿走了”；你后续是在一个较干净的文件状态上继续改。

### 1. stash 前

假设原始文件 `app.js` 是：

```js
console.log("hello");
```

你正在做功能 A，把它改成：

```js
console.log("hello");
console.log("feature A");
```

这时候，这一行：

```js
console.log("feature A");
```

还只是你本地未提交的修改。

### 2. 执行 stash 后

你执行：

```bash
git stash
```

这时工作区里的 `app.js` 会变回近似这样：

```js
console.log("hello");
```

注意这里的关键点：
- 不是 Git 记住“这里以后不要改”
- 不是 Git 在“忽略 feature A 那一行”
- 而是 `feature A` 这行已经被 stash 临时收起来了，所以当前文件里暂时看不到它

### 3. 开始新任务修改

这时你开始做新任务 B，把文件改成：

```js
console.log("hello");
console.log("bugfix B");
```

这说明你现在是在“没有 feature A 那行的状态”上继续写新修改。

### 4. 执行 pop 恢复

之后你执行：

```bash
git stash pop
```

Git 会尝试把之前 stash 进去的：

```js
console.log("feature A");
```

重新套回当前文件。

如果两边改动不冲突，可能变成：

```js
console.log("hello");
console.log("bugfix B");
console.log("feature A");
```

如果新任务 B 也改了和 feature A 相同的位置，Git 可能就无法自动合并，这时就会出现冲突，需要你手动处理。

---

## 八、stash 前 → stash 后 → 新任务修改 → pop 恢复 的超直观文字图

```text
[初始状态]
app.js
└─ console.log("hello");

[你在做功能 A]
app.js
├─ console.log("hello");
└─ console.log("feature A");   ← 这是未提交修改

[执行 git stash]
feature A 这行被临时收进 stash
当前工作区里的 app.js 变回：
└─ console.log("hello");

[你开始做新任务 B]
app.js
├─ console.log("hello");
└─ console.log("bugfix B");

[执行 git stash pop]
Git 尝试把刚才收起来的 feature A 再放回来

[如果能自动合并]
app.js
├─ console.log("hello");
├─ console.log("bugfix B");
└─ console.log("feature A");

[如果不能自动合并]
Git 报冲突
你需要手动决定：
- 保留哪一部分
- 还是把两边内容重新整理后同时保留
```

这个图最重要的结论是：
- stash 后，旧修改不是“被忽略”
- 而是“暂时不在当前工作区”
- 之后 `pop` / `apply` 时，Git 才会尝试把它重新合并回来

---

## 九、stash 和 commit 的区别

### commit 是什么

`commit` 是正式保存进项目历史。

特点：
- 是项目版本历史的一部分
- 通常应该有清晰含义
- 会参与协作、review、回滚

### stash 是什么

`stash` 是临时藏起来，不是正式历史。

特点：
- 更像临时收纳
- 主要给自己中途切任务用
- 不应该替代正常提交

一句话：
- `commit` = 正式归档
- `stash` = 临时寄存

---

## 十、为什么 stash 不是长期方案

stash 很有用，但它不是长期主流程。

因为：
1. 它不如 commit 清晰
2. 放久了你可能忘了里面是什么
3. 多个 stash 叠起来后容易混乱
4. 恢复时可能冲突

所以更好的原则是：
- 短暂切任务：可以 stash
- 已经形成一个明确阶段：更适合 commit

---

## 十一、stash、branch、worktree 的关系

在更成熟的 Git 工作流里，stash 不是主角。

因为：
- stash 解决的是“临时收起现场”
- branch 解决的是“历史边界”
- worktree 解决的是“并行工作区隔离”

所以 stash 更像：
- 临时工具
- 过渡工具
- 应急工具

而不是默认开发主流程。

如果你开始习惯：
- 每个任务一个分支
- 并行任务用 worktree

你就会越来越少依赖 stash。

---

## 十二、给零基础用户的最简记忆版

你现在先只背这三句就够了：

1. `git stash` = 临时保存未提交修改
2. `git stash pop` = 恢复刚才藏起来的修改
3. stash 不是正式提交，commit 才是正式历史

---

## 十三、延伸链接

- [[Git.canvas]]
- [[Git移除已跟踪文件与多设备Pull同步问题]]
- [[Git删除已跟踪隐藏目录后，如何保留其他设备本地文件]]
