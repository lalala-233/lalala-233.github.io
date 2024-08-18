# 一种在 Typst 中使用任务列表的方法（大概）

> 本文用 Markdown 编写，没找到 Typst 生成到 Markdown 的方法，doscify 也不支持 Typst

## 前言

我一直以为 Typst 是 Markdown 的上位替代，但后来发现 Typst 主要目的是取代 Latex，所以有些 Markdown 语法它并不支持。比如 Markdown 的任务列表语法（据说这是 Markdown 的语法扩展）。

```markdown
- [ ] 任务1
- [x] 任务2

1. [] 任务3
2. [x] 任务4
```

这将会渲染为：

- [ ] 任务1
- [x] 任务2

1. [] 任务3
2. [x] 任务4

注意，有的平台可能会不支持该语法，不同平台渲染效果可能不同。

## 解决方案

目前好像没有什么特别优雅的解决办法，big-todo 和 dash-todo 的效果也不怎么令人满意。网络上讨论的人好像也不多，拿 Emoji 凑合着用吧。

```typst
#let todo = emoji.square.medium
#let done = emoji.checkmark.heavy
#let cancel = emoji.crossmark

1. #todo 还未完成
2. #done 已完成
3. #cancel 已取消
```

渲染效果大致长这样（用 Markdown 模拟）：

1. ◼还未完成
2. ✔已完成
3. ❌已取消
