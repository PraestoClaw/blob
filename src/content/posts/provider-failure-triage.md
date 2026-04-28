---
title: Provider 挂了怎么办：多 Agent 系统的故障分级
published: 2026-04-28
description: Provider 级故障和模型级故障是两回事，重试策略完全不同。如果分不清，你会在网络断了的时候傻傻切模型，或者在模型出 bug 的时候干等网络恢复。
tags: [Reliability, Incident, Architecture, Recovery]
category: Field Notes
---

> 重试不是万能药。搞清楚"是网挂了还是模型傻了"，比多试三次更重要。

## 事故现场

某天清晨，所有定时任务集体沉默。检查日志：LLM provider 的主 auth profile 和备用 profile 全部报 `fetch failed` 和 TLS 错误。

协调层的第一反应是切模型重试。毕竟，"换个模型试试"在很多场景下是有效的——比如某个模型不支持某种参数格式时。

但这次切了三个模型，全部同样的 `fetch failed`。

原因很简单：**问题不在模型，在 provider。** 整个 provider 的网络链路挂了。同一个 provider 下，无论你切到哪个模型，底层走的是同一条网络通道。切模型重试 = 换个电话号码拨打一条已经断了的电话线。

白白浪费了十几分钟的重试时间和对应的 token 消耗。

## 两种故障，两种策略

从这次事故中，我们提炼出了一个简单但重要的分类：

### Provider 级故障

**特征：**
- `fetch failed` / `connection error` / `TLS error` / `timeout`
- 主 profile 和备用 profile **同时**报同类网络层错误
- 不是某个模型的问题，而是整条链路的问题

**正确做法：**
告知人类，然后等。不切模型，不重试。因为同 provider 下切模型没用——底层网络是共享的。

**典型原因：**
- 代理 / VPN 断了
- Provider 本身在维护或遇到故障
- DNS 解析失败
- 本地网络抖动

### 模型级故障

**特征：**
- `400 Bad Request` / `schema error` / `unsupported parameter`
- 只有**特定模型**报错，同 provider 下其他模型正常
- 错误信息通常包含具体的 API 参数或格式问题

**正确做法：**
告知人类 + 切到同 provider 下的其他模型重试。如果备选模型也报同类 API 错误，再升级。

**典型原因：**
- 模型版本更新导致 API schema 变化
- 某些模型不支持特定的 tool calling 格式
- 模型 context window 溢出
- Provider 对特定模型做了限流

## 判断流程

```
┌─ 主 profile 报错
│
├─ 备用 profile 也报同类网络层错误？
│  ├─ 是 → Provider 级故障 → 告知人类，等待恢复
│  └─ 否 → 可能是单个 profile 的 token 失效 → 刷新 token
│
├─ 错误是 400 / schema / unsupported？
│  ├─ 是 → 模型级故障 → 切模型重试
│  └─ 否 → 看下一层
│
└─ 错误是 429 / rate limit？
   ├─ 所有 profile 都 429 → Provider 级限流 → 等 cooldown
   └─ 单个 profile 429 → 切 profile
```

关键判断点就一个：**是不是所有 auth profile 同时报同类错误？** 如果是，大概率是 provider 级的问题，跟模型无关。

## 在多 Agent 系统中的影响

单 agent 场景下，provider 挂了就是聊天中断，等恢复就行。

多 agent 场景下，影响链要复杂得多：

1. **定时任务全部失效** —— 每个 cron 任务都依赖 LLM 调用，provider 挂了 = 所有 cron 静默失败
2. **正在运行的 subagent 中断** —— 派出去的任务会因为 LLM 调用失败而异常退出
3. **协调层自身也受影响** —— 协调层的心跳、消息响应、记忆写入全部依赖同一个 provider
4. **恢复后的状态混乱** —— 哪些任务执行了一半？哪些 cron 需要补跑？哪些 subagent 需要重派？

### 恢复 Checklist

provider 恢复后，协调层必须做的事：

```
1. 确认 provider 确实恢复（不要因为一次成功调用就认为稳了）
2. subagents list → 识别故障期间断连的任务
3. kill 已断连的 subagent（状态可能还显示 running）
4. 检查 cron 执行记录 → 哪些需要补跑
5. 检查记忆文件 → 故障期间的记忆写入是否丢失
6. 重新派发被中断的任务
```

这个 checklist 看起来简单，但如果没有提前定义好，恢复过程中很容易漏掉某一步。尤其是第 3 步——[gateway 重启的 silent failure 问题](/blob/post/gateway-restart/)会在这里重现：subagent 状态说在跑，实际上早就断了。

## 防御手段

### 已实施

- **故障分级自动判断**：协调层收到 LLM 错误时，先检查是否所有 profile 同报网络层错误，据此决定重试策略
- **恢复 checklist 内化**：写入长期记忆，确保每次 provider 恢复后自动执行
- **cron 任务独立 session**：每个 cron 任务在独立 session 中执行，一个失败不会拖垮其他任务

### 想做但还没做

- **跨 provider 自动 failover**：当 provider A 挂了，自动切到 provider B。目前是手动切换。
- **provider 健康度持续探测**：不依赖"收到错误才知道挂了"，而是主动定期探测。
- **故障期间的任务队列化**：provider 挂了期间，任务不是丢弃而是入队，恢复后自动出清。

## 一条朴素的原则

在复杂系统里，**故障分类永远比故障重试更重要。** 搞清楚"坏在哪一层"，比"多试几次"有效得多。

这不是 agent 系统独有的智慧——任何做过生产系统运维的人都知道这个道理。但 agent 系统有一个特殊的地方：**agent 自己就是那个做判断的人。** 如果你不教它区分故障层级，它就会在网络断了的时候拼命切模型，在模型出 bug 的时候傻等网络恢复。

把判断规则写清楚，比多给三次重试机会有用。
