# 至简行易 · An Anti-Overengineering Skill for Claude

> Stop guessing. Start with facts. Delete before adding.
> 别猜。先列事实。能删就别加。

A 50-line skill that pushes Claude against three common AI coding anti-patterns: over-defensive `try/except`, dumping parallel solutions for the user to pick, and speculating about code without reading it.

**Status**: v0.4, dated 2026-04-28. Single-author, same-day validation only. Read [Limitations](#limitations) before treating the numbers below as proof of anything.

---

## What it changes about Claude's defaults

| Default LLM behavior | What this skill does instead |
|---------------------|------------------------------|
| Adds `try/except` "just in case" | Requires a specific failure on record before adding any defense |
| Lists 3 parallel solutions for you to pick | Commits to one main path, fallback at the end |
| Speculates about code without reading it | Refuses to describe behavior until source is read |
| Says "should work / probably / usually" | Replaces vague language with verifiable commands |
| Suggests abstraction for "future flexibility" | Blocks abstraction until ≥2 real call sites exist |
| Sprays optimization suggestions on debug requests | Demands the original error stack first |

---

## What's inside

50 lines · one screen.

- **3 principles** (`先删后加 / 事实胜类比 / 识位而后动`)
- **2 entry questions** that force atomic-fact listing before any action
- **5 stances** (动 / 等 / 变 / 止 / 复) for situation-aware action
- **6 prohibitions**, each with its `*因*` — the rationale defines the boundary
- **A skip condition** for trivial tasks: write minimum viable first, ask later
- **A `text_evolution/` folder** for logging real usage, so future revisions can be driven by data rather than the author's intuition

Full content: [`SKILL.md`](./SKILL.md).

---

## Install

### Option A: Project-level (recommended)

In your project root, edit/create `CLAUDE.md`:

```markdown
## Engineering discipline
This project follows 《至简行易》. Read before acting:
`<absolute-path-to-cloned-repo>/SKILL.md`
```

### Option B: User-level (global)

Add the same reference to your global `~/.claude/CLAUDE.md`. Trade-off: simple chat tasks become more verbose.

### Option C: One-off (per conversation)

Paste at the start of any conversation:

```
Please Read <path>/SKILL.md and follow its principles for the rest of this conversation.
```

---

## Numbers

All numbers come from subagent runs on 2026-04-28. Useful as a sanity check, not as proof. See [Limitations](#limitations).

| Metric | Value | What it actually measures |
|--------|-------|---------------------------|
| Trigger accuracy (majority vote, 4 runs × 16 prompts) | 16/16 | Whether an evaluator agent reading only the `description` fires on the right prompts. Does **not** measure whether the loaded skill changes Claude's output. |
| Single-run trigger variance | 3/16 prompts split 2-1 across runs | Borderline cases the majority-vote headline hides. |
| Token overhead per task | +5.6% (avg) | Output length cost. |
| Time overhead | +41% mid-project, +157% trivial 0→1 | Inverse to task complexity — trivial tasks pay the largest relative cost. |

Execution quality is reported as 6 paired before/after cases, not aggregate scores. See [`benchmarks.md`](./benchmarks.md) for the actual outputs.

**Practical implication**: don't load this skill for one-shot scripts, chat, lookups, or trivial tasks — the time tax is steepest there.

---

## Limitations

- **Single author, same-day validation.** v0.1 → v0.4 were all iterated and benchmarked on 2026-04-28 by the author. No third-party replication.
- **Trigger F1 = 100% measures the description, not the skill.** The evaluator agent only sees the `description` field. A well-tuned description and a well-tuned skill body are different things; the numbers conflate them.
- **`text_evolution/` is unused.** SKILL.md says: *"This is the only credible data source for evolving this skill; without this folder, every modification is speculation."* As of v0.4 there is exactly one entry — the author's initial-eval. By the skill's own standard, any revision past v0.4 is speculation until external entries accumulate.
- **Baseline is "no skill loaded", not "modern Claude system prompt".** Recent Claude versions already discourage several of these anti-patterns. The incremental value over a current system prompt is **not** measured.
- **Prohibition #1 over-fires.** When a user explains behavior verbally without sharing source, the skill refuses to engage. Known issue; not fixed in v0.4.
- **The skill makes outputs longer, not shorter.** "先删后加" applies to thinking; the printed response adds verification commands, fact lists, and risk callouts.
- **6 execution-quality cases is not a benchmark, it is a demonstration.** N=6, single author, no blind scoring.

---

## When NOT to use this skill

- Pure conversation / factual questions
- Translation, summarization, formatting tasks
- Algorithm interview questions
- "Explain how X works" knowledge queries
- Anything that doesn't involve writing or modifying code

The trigger description filters these out in testing — but see the variance note above.

---

## Origin

Built and iterated in one afternoon (2026-04-28) inside Anthropic's Cowork mode. Every "version bump" in [`CHANGELOG.md`](./CHANGELOG.md) is a same-day edit, not a field-tested release. The four-version history reads like a release sequence; treat it as a single design session with annotated revisions.

---

## License

[MIT](./LICENSE) — use it, fork it, modify it, ship it. If you run it on real work and find a failure case, log it in `text_evolution/` and open a PR.

---

## 中文简介

《至简行易》是一份给 Claude 用的工程心法 skill，针对 AI 编程时最常见的几类反模式：

- 没读源码就猜行为
- "以防万一"加 try/except
- 给三个方案让用户挑
- 用"应该会 / 可能 / 通常"等模糊词

50 行、一屏可见、每条禁令带 `*因*`。完整内容见 [`SKILL.md`](./SKILL.md)。

**当前状态**：v0.4，作者本人在 2026-04-28 单日完成全部迭代与评测，**没有第三方复现**。`text_evolution/` 设计上承担"自演化的唯一可信数据源"，但目前只有作者的一条 initial-eval 条目——按 skill 自己的标准，v0.4 之后的任何修改都还是猜测。`benchmarks.md` 里的"100% F1"测的是 description 写得好不好让评测 agent 投票通过，不是 skill 加载后 Claude 输出真的变好——这两件事不一样。在依赖这些数字之前，请读 [Limitations](#limitations)。
