# 至简行易 · An Anti-Overengineering Skill for Claude

> Stop guessing. Start with facts. Delete before adding.
> 别猜。先列事实。能删就别加。

A 50-line Claude skill that systematically prevents the most common AI coding pitfalls. Fits on one screen. Tested with real benchmarks. Self-documenting evolution.

**Current version**: `v0.4` · **Trigger F1**: 100% (majority vote, 4 runs × 16 queries) · **Zero false triggers** across 32/32 negative cases.

---

## The problem this skill solves

When you ask Claude (or any LLM) to help with code, the default behavior is to be helpful by **adding stuff** — error handling, abstractions, alternatives, "future-proofing". This optimizes for looking thorough, not for shipping.

This skill enforces a different default: **before adding anything, prove you can't delete it.**

### Concrete patterns it stops

| Default LLM behavior | What this skill does instead |
|---------------------|------------------------------|
| 🚨 Adds `try/except` "just in case" | Refuses unless there's a specific failure on record |
| 🚨 Lists 3 parallel solutions for you to pick | Commits to one main path, fallback at the end |
| 🚨 Speculates about code without reading it | Refuses to describe behavior until source is read |
| 🚨 Says "should work / probably / usually" | Replaces vague language with verifiable commands |
| 🚨 Suggests abstraction for "future flexibility" | Blocks abstraction until ≥2 real call sites exist |
| 🚨 Sprays optimization suggestions on debug requests | Demands the original error stack first |

---

## What's inside

```
50 lines · one screen · self-describing
```

- **3 principles** (`先删后加 / 事实胜类比 / 识位而后动`)
- **2 entry questions** that force atomic-fact listing before any action
- **5 stances** (动 / 等 / 变 / 止 / 复) for situation-aware action
- **6 prohibitions** with rationale (`why` is the boundary, not the rule)
- **A skip condition** for trivial tasks: write minimum viable first, ask later
- **A self-evolution mechanism** (`text_evolution/`) — every project tracks its own usage data

See [`SKILL.md`](./SKILL.md) for the full skill content.

---

## Why this is different from "prompt engineering tips"

Most "AI coding rules" are static. This skill includes its own **regression test framework**:

- [`benchmarks.md`](./benchmarks.md) — Real subagent runs comparing with-skill vs baseline on 18 scenarios
- [`CHANGELOG.md`](./CHANGELOG.md) — Honest evolution log: where v0.1 contradicted itself, how v0.2 fixed it, what trade-offs each version made
- [`text_evolution/`](./text_evolution/) — A folder structure that you copy into your own project, where you log real usage data so you can later refactor the skill based on **what actually fails**, not vibes

The skill literally has a clause: *"This is the only credible data source for evolving this skill; without this folder, every modification is speculation."*

---

## Install (3 ways)

### Option A: Project-level (recommended)

In your project root, edit/create `CLAUDE.md`:

```markdown
## Engineering discipline
This project follows 《至简行易》. Read before acting:
`<absolute-path-to-cloned-repo>/SKILL.md`
```

Claude reads `CLAUDE.md` automatically when entering a project.

### Option B: User-level (global)

Add the same reference to your global `~/.claude/CLAUDE.md`. The skill will be active in every project. Trade-off: simple chat tasks become more verbose.

### Option C: One-off (per conversation)

Paste at the start of any conversation:

```
Please Read <path>/SKILL.md and follow its principles for the rest of this conversation.
```

---

## Benchmarks (the receipts)

| Metric | With skill | Baseline | Δ |
|--------|-----------|----------|---|
| Trigger precision (no false fires) | **100%** | n/a | n/a |
| Trigger recall (majority of 4 runs) | **100%** | n/a | n/a |
| Token overhead per task | +5.6% | — | small |
| Time overhead per task | +41% (mid-project) ~ +157% (simple 0→1) | — | grows with complexity, **inverse to value** |

Critical finding: **the skill's relative cost decreases as task complexity grows.** Simple "write me a script" tasks see the biggest time hit. Complex inheritance / debugging tasks barely notice.

Full numbers in [`benchmarks.md`](./benchmarks.md).

---

## When NOT to use this skill

This is honest dogfooding of the skill's own prohibition #5 (one main path, alternatives at the end):

- ❌ Pure conversation / asking factual questions
- ❌ Translation, summarization, formatting tasks
- ❌ Algorithm interview questions
- ❌ "Explain how X works" knowledge queries
- ❌ Anything that doesn't involve writing or modifying code

The trigger description filters these out automatically.

---

## Origin story

Built and tested in a single afternoon (2026-04-28) inside Anthropic's Cowork mode. Every iteration was driven by a real subagent benchmark, not by intuition.

Read [`CHANGELOG.md`](./CHANGELOG.md) for the full v0.1 → v0.4 evolution — including the moment v0.1's prohibition #5 directly contradicted its own "止" (halt) stance, and how v0.2 resolved it via scope separation.

---

## License

[MIT](./LICENSE) — use it, fork it, modify it, ship it. If you find a real failure case in your project's `text_evolution/`, send a PR.

---

## 中文简介

《至简行易》是一份给 Claude 用的工程心法 skill，专治 AI 编程时最常见的过度工程化倾向：

- 没源码就猜行为
- "以防万一"加 try/except
- 给三个方案让你挑
- 用"应该会 / 可能 / 通常"等模糊词

50 行、一屏可见、带"为什么"的禁令系统、能记录自己进化的 `text_evolution/` 文件夹。完整内容见 [`SKILL.md`](./SKILL.md)。
