# Changelog

All notable changes to this skill, with rationale.

This skill follows the principle stated in its own constitution: **"v1 完成即 v2 起点" — every released version is a starting point for the next.** No version is treated as "done".

---

## [v0.4] - 2026-04-28

### Added
- **`text_evolution/` self-evolution mechanism** — when this skill is brought into a new project (including inherited mid-development codebases), a `text_evolution/` folder is created at project root to log real usage data.
- New section in skill: "项目接入与演化记录" with the explicit clause: *"This is the only credible data source for evolving this skill; without this folder, every modification is speculation."*

### Changed
- **Description optimization**: added explicit mention of the "加 try/except 加配置 加抽象" temptations in the trigger description.
  - **Why**: a benchmark run revealed that prompt #8 ("我准备给这个 50 行脚本加 try/except 让它更健壮") — a textbook case for prohibition #4 — was being missed by the trigger evaluator. The original "改既有代码" was too generic.
  - **Result**: trigger recall went from 87.5% → 100% (majority vote across 4 runs × 16 queries).

### Verified
- Trigger precision = 100% (32/32 should-not-trigger queries correctly skipped, across 4 runs)
- Trigger F1 (majority vote) = 100%
- Token overhead = +5.6% per task (acceptable)
- Time overhead = +41% (mid-project) ~ +157% (simple 0→1) — inverse to task complexity

---

## [v0.3] - 2026-04-28

### Changed
- **Skip condition rewritten from descriptive to imperative**:
  - Before: "跳过条件（满足任一，直接动手，不用过两问）"
  - After: "跳过条件（满足任一 → **先写最小可运行版本，写完再问澄清**；不要先问后写）"
  - **Why**: in v0.2 testing, even when the skip condition theoretically triggered, the model still asked clarifying questions before writing code. By making the action ("先写") imperative and explicit, we forced action-first behavior. Verified working in v0.4 benchmarks: Test 1 went from "asks 2 questions then writes" → "directly writes 13-line minimal pypinyin script + verification + risk list".
- **Prohibition #5 extended with `*补*`**: added "收集证据时也按'主路径 + 兜底'——先推荐路径 A，备选 B/C 放末尾，不要 A/B/C 并列让用户挑."
  - **Why**: closes the loophole where the model would list 3 paths for "how to provide evidence" rather than 3 parallel solutions. Both are equally bad UX.

---

## [v0.2] - 2026-04-28

### Fixed (the original sin)
- **Resolved the internal contradiction between "止" stance and Prohibition #5.**
  - The conflict in v0.1: "止" stance instructed "list 2–3 explanations". Prohibition #5 forbade "listing parallel options for the user to pick". The model would inevitably violate one to comply with the other.
  - **Resolution via scope separation**:
    - "止" is now defined as **asking** ("讲不清用户要什么时 → 列 2–3 种对需求的解释 + 1 个澄清问题（这是'问'，不是'给方案'）")
    - Prohibition #5 is for **giving solutions** — clarified by adding: *"澄清需求时列多解不算'方案'，不受此限"*
  - **Why this is more elegant than v0.1**: it wasn't a rule conflict — it was vocabulary collision on the word "list". Once you separate "asking" from "solving", both rules coexist peacefully.

### Added
- **Skip condition** for entry questions (3 specific triggers):
  - Task estimate ≤ 20 lines / no new files / no structural changes
  - User already provided atomic facts (numbers / paths / stack traces / interface signatures)
  - Prompt explicitly contains "简单 / 快 / 跑通 / demo / hello"
- ***因* annotations** on every prohibition. The boundary of each rule is now defined by its rationale, not by the rule's literal text.
- **Self-conservation rewritten as imperative**: "增项前先尝试'再削一刀'；若不能删，再补不迟" (was: descriptive "本 skill 一屏可见").

### Verified
- Internal consistency score: 5/10 → 8/10
- Total skill score: 6.7 → 7.6

---

## [v0.1] - 2026-04-28 (initial)

### The starting point
The first version: 3 principles, 2 entry questions, 5 stances, 6 prohibitions, self-conservation rule. ~36 lines.

### Known issues at release (revealed by first benchmark)
1. **Internal contradiction**: "止" stance vs Prohibition #5 (described above, fixed in v0.2).
2. **Prohibitions had no rationale** — pure imperative rules with no edge-case guidance. Models would either over-comply or under-comply on borderline cases.
3. **No skip condition** — even simple "give me a quick demo" requests triggered the full entry-question ceremony.

### Why v0.1 still mattered
The benchmark on v0.1 revealed all of these issues in 3 test cases. Without releasing v0.1, we couldn't discover what was wrong. **The first version doesn't have to be right — it has to be falsifiable.**

---

## Versioning philosophy

Each version is shipped with:
- A specific behavioral change
- A reason for the change (drawn from a real failure case)
- A measurable improvement (or honest regression note)

Versions are not bumped for cosmetic changes. If the skill text changes but the behavioral test results don't move, that change doesn't earn a version bump — it's just a typo fix.
