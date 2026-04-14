---
title: "Prefix‑Cache 友好的 SubAgent 派生：一种 fork() 式设计"
date: 2026-04-14T00:08:01+08:00
slug: 2026-04-14-my-site
type: posts
draft: false
categories:
  - Computer
markup:
  goldmark:
    extensions:
      passthrough:
        delimiters:
          block:
          - - \[
            - \]
          - - $$
            - $$
          inline:
          - - $
            - $
        enable: true
params:
  math: true
---

这里记录着我个人对 LLM Agent 的 Ideas

## 最近的工作

最近，我自己编写了一个给自己用的 CLI 工具 (OMG CLI)

设计这个 CLI 的时候大范围借鉴了 `kimi-cli`

而关于 CLI 的设计上来说，业界也会有诸多的人如我一般，都会思考两件事

1. 上下文如何节省
2. 注意力如何不分散

于是我们今天就来聊一聊，我对这两件事的不同看法和实践

我们首先必须得知道的一个前提是上下文很**昂贵**

这个**昂贵**有两层含义，一个是经济上的，一个是对于 Agent 来说的

前者不必提，后者其实就是当上下文接近 80% 的时候 AI 就会被记忆给拖累以至于无法再有新产出

基于这个前提，我们自然会有许多手段来防止**上下文**溢出

先说三个业界内比较普遍的手段。

### 1. 上下文压缩

让 LLM 自己把整个上下文压缩，去掉冗余信息 `kimi-cli` 在这点上做的非常顶，它们的 [`COMPACT.md`](https://raw.githubusercontent.com/MoonshotAI/kimi-cli/7869e8cc2a1db7cf968ad60b5b844d0afa392774/src/kimi_cli/prompts/compact.md) 非常棒。

### 2. 动态裁剪

上次跟朋友讨论这个的时候，朋友分享了一个 [OpenCode](https://github.com/Opencode-DCP/opencode-dynamic-context-pruning) 的插件

就是把一些不必要的 `ToolCall` 和 `ToolResult` 让 LLM 自己压缩起来

这种方案缺点是会导致模型提供商围绕请求做的缓存失效。


### 3. 子代理 (SubAgent)

当前的 SubAgent 的方案其实就是多花 Token 外包某项任务给其它 Agent，而后 Main Agent 就可以通过 SubAgent 的汇报来保证自己注意力和上下文不分散，其实就是砸钱。


### 4. 我个人的方案 (Fork)

我个人的方案是操作系统那个异常经典且令人拍案叫绝的 `fork()` 函数

其核心思想是：SubAgent 继承 MainAgent 在分叉时刻的完整上下文。

为什么是这个方案？

1. 模型提供商的缓存大多数基于**前缀**匹配

`DeepSeek`, `Anthropic` or `MoonShot` 等提供商对缓存的判断其实是**前缀**是否相同。

**前缀**

对当前 SubAgent 方案上下文的建模

$$
\begin{array}{c}
\text{Main Agent:} \quad \underbrace{\rule{4cm}{0.4pt}}_{\text{Thinking Round}} \quad \blacktriangleright \\
\text{SubAgent:}  \quad \underbrace{\rule{4cm}{0.4pt}}_{\text{Thinking Round}} \quad \blacktriangleright
\end{array}
$$

如果是 `Fork` 的上下文应该如下

$$
\begin{array}{ccccc}
               &                & \stackrel{\text{Round 1}}{\text{MainAgent}} &                & \\
               & \swarrow       & \downarrow                                 & \searrow       & \\
               & \text{Sub1}    & \text{Main}                                & \text{Sub2}    & \\
               & \downarrow     & \downarrow                                 & \downarrow     & \\
               & \text{Round 2} & \text{Round 2}                             & \text{Round 2} & \\
               & \text{(并行)}  & \text{(并行)}                              & \text{(并行)}  &
\end{array}
$$

如果我们计入 `缓存命中` 的话，那么 `fork` 的方案就会有很明显的优势。

1. 缓存复用率很高，缓存命中之后成本节省接近 `90%` 
2. 完整的项目记忆，对项目的记忆无损的转发到 SubAgent 上，只要这个 SubAgent 承担是有关项目的工作，那么就不必再去**自己探索**或依赖其它有**信息损耗**的设计方案(譬如 `AGENTS.md`)



### 其它

我个人最近在找工作，有任何工作机会欢迎联系我
Email: `smallorangeqwq@gmail.com`