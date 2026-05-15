# Benchmarks

All numbers come from subagent runs inside Anthropic's Cowork mode on 2026-04-28. With-skill runs read `SKILL.md` before responding; baseline runs use the same prompt with no skill loaded.

### Methodology caveats (read these before the numbers)

- **N is small.** Trigger accuracy is 16 prompts × 4 runs. Execution quality is 6 paired cases. This is a sanity check, not a benchmark in the statistical sense.
- **Single author, single day.** All four versions (v0.1 → v0.4) were written and evaluated on 2026-04-28 by the skill's author. Replication by a third party would carry more weight than the numbers below.
- **Trigger accuracy measures the description, not the skill body.** The evaluator agent only reads the `description` field. A well-tuned description that triggers correctly says nothing about whether the loaded skill changes Claude's output.
- **Baseline = "no skill loaded", not "modern system prompt".** Recent Claude versions already discourage some of these anti-patterns. The incremental value over a current system prompt is **not** measured here.
- **Same author writes both the skill and the test prompts.** Goodhart's risk applies: the description was tuned against these prompts.

---

## Trigger accuracy (does the skill fire when it should?)

**Method**: 16 user prompts (8 should-trigger, 8 should-not-trigger). For each prompt, an independent evaluator subagent reads only the skill's `description` field and judges whether the skill should fire. Run 4× across two description versions for variance.

### Final result (description v0.4, majority vote across 4 runs)

| Metric | Value |
|--------|-------|
| Accuracy | 16/16 = **100%** |
| Precision (no false fires) | 8/8 = **100%** |
| Recall | 8/8 = **100%** |
| F1 | **100%** |

### Single-run variance (what the majority-vote headline hides)

13/16 prompts received unanimous judgments across all 4 runs. 3 prompts (#1, #2, #7) showed 2-1 splits — all on borderline "simple coding" cases. Majority vote rounds these to 100%, but a different sample of 4 runs could plausibly flip one of them.

By the official Anthropic guidance ("Claude only consults skills for tasks it can't easily handle on its own"), borderline behavior on borderline cases is expected. But the headline F1 = 100% should be read as **"100% on majority vote across this particular 4-run sample"**, not as "the description partitions cases cleanly."

### Should-not-trigger correctness

Across 4 runs × 8 should-not-trigger queries = 32 evaluations.
**32/32 correctly judged as N (no false trigger).**

The 8 negative cases include near-misses designed to test if the description is too greedy:

- "解释下 SOLID 原则是什么"
- "Python 装饰器原理是啥？"
- "面试题：用 Python 实现快排"
- "Python 和 Go 哪个更适合做后端"
- "在 Excel 里把 A 列和 B 列合并"
- "翻译下这段英文文档"
- "我想买台 MacBook，开发够用吗"
- "总结一下这篇论文"

---

## Execution quality (does the skill produce better output when it does fire?)

**Method**: 6 prompts × 2 configurations (with-skill, baseline) = 12 independent subagent runs. Outputs read verbatim and compared.

### Test set 1: 0→1 simple tasks

| # | Prompt | Baseline behavior | With-skill behavior | Verdict |
|---|--------|-------------------|---------------------|---------|
| 1 | "做个简单的命令行工具，输入中文输出拼音" | Writes 3 alternative scripts (recommended/with-tone/detailed) | **Directly writes 13-line minimal pypinyin script + verification command + risk list. No alternatives.** | Skill enforces single main path |
| 2 | "帮我优化下这个脚本的性能" (no script attached) | "Please share the script — I can help with bottlenecks, complexity, memory, ..." | "我看不到代码，没法优化. 给数字和源码再说." | Skill is sharper |
| 3 | "我的 React 组件不渲染，帮我修" (no source) | Lists 5 generic causes (syntax/conditional/state/return/CSS) and asks for code | Refuses to speculate. Asks for either error stack OR source code, with explicit clarification of what "不渲染" means | Skill blocks pattern-matching speculation |

### Test set 2: Mid-project / inheritance scenarios

| # | Prompt | Baseline | With-skill | Verdict |
|---|--------|----------|------------|---------|
| 1 | "刚接手这个 React 项目，老板让我加个搜索功能" | 100+ lines: project structure / requirements / 5 future optimizations (debounce/highlight/history) / ready-to-paste code | Asks 1 clarifying question (client filter vs server search) before writing anything. Refuses to pre-list "future optimizations" | Skill blocks Prohibition #4 violations |
| 2 | "接手别人写的 Python 服务，try/except 套了 5 层我想清理下" | Lists 7 refactoring patterns (merge / extract / context manager / exception chain / EAFP / linter / checklist) — encyclopedic | Demands first the **atomic facts** about each layer (what exception, what failure case, what frequency in production logs). Refuses to apply patterns without seeing actual code. | Skill blocks "套通用模式" |
| 3 | "这个 issue 我 debug 2 小时了还没头绪，原报告说'登录偶尔失败'，我现在在看 token 刷新逻辑" | Lists 4 categories of common causes (token logic / clock skew / session edge cases / network) and follows the user's current focus | **Triggers "复" stance**: stops, points out 2 hours = divergence signal, returns to original issue's first sentence ("'偶尔'是什么频率？"), refuses to deepen the user's existing assumption (token refresh) until base facts are established | Skill is dramatically better |

---

## Efficiency (what does this cost?)

| Scenario | Baseline avg | With-skill avg | Token Δ | Time Δ |
|----------|-------------|----------------|---------|--------|
| 0→1 simple tasks | 57.6k tokens, 8.1s | 60.8k tokens, 21.0s | **+5.6%** | **+157%** |
| Mid-project scenarios | 59.2k tokens, 19.2s | 60.8k tokens, 27.1s | **+2.6%** | **+41%** |

### Reading the time cost

Time overhead is inverse to task complexity:
- On trivial 0→1 tasks the baseline finishes in ~8s, so the skill's "deliberation tax" looks large (+157%).
- On mid-project tasks the baseline already takes ~19s, so the same tax is a smaller fraction (+41%).

This is not the skill "paying for itself" on hard tasks — it is the **denominator growing**. The skill still adds 5–8 seconds of deliberation either way. Whether that deliberation produces better output on hard tasks is the question, and the 3 mid-project cases above are too few to answer it confidently.

**Practical takeaway**: don't load this skill for trivial chat / lookup / one-shot script tasks. The cost gradient is unfavorable and the skill's value is hardest to see there.

---

## What did not improve

- **Output length goes up, not down.** Despite the principle "先删后加", with-skill responses add verification commands, atomic-fact lists, and risk callouts. Character count is consistently higher than baseline. The skill's brevity lives in the *thinking*, not the *printed output*.
- **"止" costs a round-trip.** When the model halts to ask, you have to answer before getting code. For prototyping where you want something running fast, this is friction.
- **Prohibition #1 over-fires.** It can trigger even when the user clearly explains the behavior verbally. Known issue, not fixed in v0.4.

---

## What is not measured

- **Whether the skill beats Claude's current system prompt.** Baseline is no-skill, not modern-system-prompt. Some anti-patterns the skill targets are already discouraged by default.
- **Whether it helps users other than the author.** Same person wrote the skill, the prompts, and the post-run inspection. Independent replication would carry more weight than these numbers.
- **Whether the qualitative wins on the 6 paired cases generalize.** N=6 is a demonstration, not a benchmark.

---

## Reproducing these benchmarks

Methodology log: [`text_evolution/2026-04-28-initial-eval.md`](./text_evolution/2026-04-28-initial-eval.md). Anyone with subagent-capable Claude access can re-run. New scenarios — especially ones where the skill **fails** — are more valuable than ones where it succeeds; PRs welcomed.
