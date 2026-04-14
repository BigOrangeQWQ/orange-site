---
title: "Prefix-Cache-Friendly SubAgent Derivation: A fork()-Style Design"
date: 2026-04-14T00:08:01+08:00
slug: 2026-04-14-llm-ideas-fork
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

This post records some of my personal ideas about LLM agents.

## Recent Work

Recently, I built a CLI tool for my own daily use (OMG CLI).

While designing it, I borrowed heavily from `kimi-cli`.

When it comes to CLI design, many people in the industry (including me) usually think about two things:

1. How to save context.
2. How to keep attention focused.

So today I want to share my different perspective and practice on these two points.

A key premise we must accept first: context is **expensive**.

This **expensive** has two meanings: economic cost, and cognitive cost for the agent.

The first one is obvious. The second means that when context usage gets close to 80%, the model is dragged down by memory pressure and struggles to produce fresh output.

Given this premise, we naturally need ways to prevent **context** overflow.

Let me first list three common approaches in the industry.

### 1. Context Compression

Let the LLM compress the whole context by itself and remove redundant information. `kimi-cli` does this extremely well. Its [`COMPACT.md`](https://raw.githubusercontent.com/MoonshotAI/kimi-cli/7869e8cc2a1db7cf968ad60b5b844d0afa392774/src/kimi_cli/prompts/compact.md) is excellent.

### 2. Dynamic Pruning

When I discussed this with a friend last time, they shared an [OpenCode](https://github.com/Opencode-DCP/opencode-dynamic-context-pruning) plugin.

It asks the LLM to compress unnecessary `ToolCall` and `ToolResult` items.

The downside is that this can invalidate the cache strategy model providers apply around requests.

### 3. Subagents

The current SubAgent approach is basically spending more tokens to outsource a task to other agents. Then the main agent uses reports from those subagents to keep its own attention and context from drifting. In short: pay more.

### 4. My Personal Approach (Fork)

My own approach is inspired by the classic and brilliant `fork()` function in operating systems.

The core idea: a SubAgent inherits the full context of the MainAgent at the branching moment.

Why this approach?

1. Most model-provider caching is based on **prefix** matching.

For providers like `DeepSeek`, `Anthropic`, or `MoonShot`, cache judgment is mostly whether the **prefix** is the same.

**Prefix**

A simple context model for current SubAgent-style setup:

$$
\begin{array}{c}
\text{Main Agent:} \quad \underbrace{\rule{4cm}{0.4pt}}_{\text{Thinking Round}} \quad \blacktriangleright \\
\text{SubAgent:}  \quad \underbrace{\rule{4cm}{0.4pt}}_{\text{Thinking Round}} \quad \blacktriangleright
\end{array}
$$

Under a `fork`-style context, it should look like this:

$$
\begin{array}{ccccc}
               &                & \stackrel{\text{Round 1}}{\text{MainAgent}} &                & \\
               & \swarrow       & \downarrow                                 & \searrow       & \\
               & \text{Sub1}    & \text{Main}                                & \text{Sub2}    & \\
               & \downarrow     & \downarrow                                 & \downarrow     & \\
               & \text{Round 2} & \text{Round 2}                             & \text{Round 2} & \\
               & \text{(parallel)}  & \text{(parallel)}                              & \text{(parallel)}  &
\end{array}
$$

If we include `cache hits`, the `fork` approach shows obvious advantages:

1. Very high cache reuse. After cache hits, cost savings can approach `90%`.
2. Full project memory is forwarded to SubAgents without loss. As long as a SubAgent is working on project-related tasks, it does not need to re-explore everything on its own or depend on other designs with **information loss** (for example `AGENTS.md`).

该文章使用GPT-5.3-Codex进行翻译。
