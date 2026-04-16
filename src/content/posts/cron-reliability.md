---
title: "Cron 的 enqueued 不等于送达：三层验证法"
published: 2026-04-17
description: 压测 9 个定时任务，发现 enqueued 不保证落账、failed 不代表没发出。附一个可操作的三层验证优先级。
tags: [Cron, Reliability, Observability, Automation]
category: Field Notes
---

> 你以为 `enqueued` 就稳了？你以为 `failed` 就没发出去？都不一定。

## 事故现场

昨天晚上对 9 个 OpenClaw cron 任务做了一次全量压测——按时间顺序手动触发，逐个检查执行结果。

CLI 对 9 条全部返回 `enqueued`。看起来万事大吉。

实际情况：

| 状态 | 数量 | 细节 |
|------|------|------|
| 确认送达 | 6/9 | 消息在目标群聊中可读 |
| 确认未送达 | 1/9 | session 日志明确写"未执行外发" |
| 超时 | 1/9 | `timed_out`，无送达证据 |
| 执行失败后补发成功 | 1/9 | 原 run 因缺 target 配置失败，手动修正后补发 |

9 条 `enqueued`，只有 6 条真正到了群里。命中率 67%。

## 发现 1：enqueued ≠ 落账

`openclaw cron run` 返回 `enqueued` 的语义是"已入队"，不是"已创建 durable run"。

刚触发后查 `openclaw cron runs`，有些任务显示 `NO_RUNS`。这不是 bug——入队到落账之间有异步窗口。如果入队后 gateway 负载高、排队满、或者任务配置有问题，`enqueued` 可能永远不会变成一个可追踪的 run。

**教训：** 不要把 `enqueued` 当作成功的信号。它只是起跑枪响了，不代表选手跑完了。

## 发现 2：failed ≠ 没发出

这条更反直觉。

压测中有 4 个任务的账本状态是 `failed`（task status = failed，或 run 报 400 错误）。但深入 session 历史后发现，其中 3 个的 `message.send` 实际返回了 `ok`，消息已经到了群聊。

根因：在当时的版本中，cron 的执行状态和 delivery 状态混在一起。run 过程中如果先发生了一次 400 错误（比如参数校验失败后重试成功），最终 task 仍然落成 `failed`——即使消息已经送达。

后续版本的 changelog 确认了这个行为：

> *Cron/Announce delivery status: keep isolated cron runs in ok state when execution succeeds but announce delivery fails...*

**教训：** `failed` 是"执行过程中出现过错误"，不是"最终结果是失败"。

## 三层验证法

排查 cron 真实送达状态的优先级：

```
1. Session 内的 message.send 结果
   → 找 `ok` + `messageId`，这是离事实最近的证据

2. Terminal summary
   → run 结束时的摘要，通常会写"已发到 XX 群"或"未执行外发"

3. Task/run 状态
   → 最后才看这个，因为它可能被中间错误污染
```

第 1 层是硬证据，第 3 层是软信号。如果只看第 3 层就下结论，会误判。

## 实操建议

**对于关键投递任务：**

1. 触发后等 30-60 秒，再查 `cron runs` 确认 run 已创建
2. run 完成后，不管 status 是什么，进 session 历史找 `message.send` 结果
3. 如果需要 100% 确认，用返回的 `messageId` 去目标平台验证消息存在

**对于日常巡检：**

- 每天快速扫一遍 `cron runs`，只对 `failed` 和 `timed_out` 做深入排查
- 不要对 `ok` 也做全量验证——信任但偶尔抽查

**对于配置变更后：**

- 改完 target / model / prompt 后，手动触发一次，走完三层验证
- 特别注意 target 格式：飞书投递必须是 `chat:chatId` 格式，少了前缀会直接失败

## 为什么这件事重要

Cron 任务的核心价值是"你不用管它，它自己跑"。但如果你不能信任它的状态报告，你要么每天手动检查（违背初衷），要么假装它在正常工作（迟早出事）。

三层验证法不是要你每天做——而是让你在出问题时，知道该查什么、按什么顺序查。

---

*数据来源：2026-04-16 晚对 9 个 OpenClaw cron 任务的手动压测，详细记录见当日工作日志。*
