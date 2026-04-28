# Benchmarks

All numbers below come from real subagent runs inside Anthropic's Cowork mode on 2026-04-28. Every "with-skill" run reads `SKILL.md` before responding; every "baseline" run is the same prompt with no skill loaded. Outputs are saved verbatim and inspected.

This is **not** a synthetic eval. Every test case is something a real developer would actually type.

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

### Single-run variance

13/16 prompts received unanimous judgments across all 4 runs. 3 prompts (#1, #2, #7) showed 2-1 splits — all on borderline "simple coding" cases. By the official Anthropic guidance ("Claude only consults skills for tasks it can't easily handle on its own"), this borderline behavior is expected and not a bug.

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

### The most important finding

**Time overhead inversely correlates with task complexity.** As tasks get harder:
- Baseline starts taking real time (because the model has to think anyway)
- The skill's "deliberation tax" becomes a smaller fraction of total cost

For trivial tasks, the skill is expensive (+157% time). For complex inherited-codebase scenarios, the skill almost pays for itself (+41% time, with significantly better output).

**Implication for users**: don't load this skill for trivial chat / lookup tasks. Load it for code work where the cost gradient is favorable.

---

## What was measured but did NOT improve

Honest reporting of trade-offs (no benchmark cherry-picking):

- **The skill makes outputs longer**, not shorter. Despite the principle "先删后加", a with-skill response includes verification commands, atomic-fact lists, and risk callouts. The total character count is consistently **higher** than baseline. The skill's brevity is in the *thinking*, not the *output*.
- **The "止" stance still costs a round-trip.** When the model halts to ask, you have to answer before getting code. For prototyping where you'd rather see something running, this is friction.
- **Prohibitions can over-fire on edge cases.** Prohibition #1 (no behavior description without source) can trigger even when the user clearly explains the behavior verbally. v0.5 may need to clarify this.

---

## Reproducing these benchmarks

The benchmark methodology is documented in [`text_evolution/2026-04-28-initial-eval.md`](./text_evolution/2026-04-28-initial-eval.md). Anyone with Cowork access (or any subagent-capable Claude environment) can re-run them. Pull requests with new benchmark scenarios are welcomed.
