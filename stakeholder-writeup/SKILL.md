---
name: stakeholder-writeup
description: Produces a stakeholder-facing md report from a multi-turn investigation conversation. Output is verified-fact-only, honest-impact, clear-scope, with a required anti-pattern list and an audience-tuned executive summary. On invocation, FIRST detects the output language (English default; Chinese when the conversation or user request is Chinese) and confirms the audience (1-Technical lead / 2-Non-technical exec / 3-PM·Business / 4-External customer), then tunes the skeleton accordingly. INVOKE ONLY WHEN EXPLICITLY CALLED — via the `/stakeholder-writeup` slash command, `$stakeholder-writeup`, or when the user explicitly names this skill. DO NOT auto-trigger on casual phrases like "write a report" / "写报告" / "写个 md" — wait for explicit invocation.
allowed-tools: Read Grep Bash Write Edit
---

# Stakeholder Report Writeup

## Core principle

**Every factual claim in a stakeholder report must trace to either (a) a concrete command output / API response in this conversation, or (b) an explicit statement from the user.** If a fact has no source, don't write it — don't estimate, don't infer, don't fill in reasonable-sounding defaults.

**Stakeholder version ≠ shortened technical doc.** It's a different genre: plain language, sourced numbers, honest uncertainty, clear scope, explicit anti-patterns.

## Language policy

This skill defaults to **English output**. Switch to **Chinese output** if any of these hold:

- The user's last 3 messages are primarily in Chinese.
- The conversation history in this session is primarily Chinese.
- The user explicitly requests Chinese ("用中文" / "中文版" / "Chinese please").

The workflow instructions below stay in English regardless. Only the produced report (section headings, closer, audience prompt) switches language. A side-by-side translation map for section names lives in step 6.

## When to use

- User wraps a multi-turn investigation and asks for a document artifact
- Audience is a decision-maker or mixed group (technical lead / executive / PM / business / customer)
- Output is persistent markdown, not chat prose

## When NOT to use

- User is asking a quick question, not for a document
- Investigation isn't done — root cause still unknown
- Audience is engineers only (write a technical RCA instead, jargon is fine)
- Context is <3 messages — there's no investigation to summarize

## Workflow (7 steps)

### 0. Confirm language and audience FIRST (before any writing)

Detect the output language (per "Language policy" above) and ask the user which audience this report is for. Present as a numbered choice in the detected language. Skip this step **only** if the conversation already makes it unambiguous (e.g. user said "for the CTO" / "给客户的" / "write this for the CEO").

**English prompt:**

```
Before I start, confirm the audience:
  1) Technical lead (keeps SQL / component tables / jargon is OK)
  2) Non-technical executive (CEO / CFO — drop SQL appendix, use analogies, 60-100 lines)
  3) PM / Business / Product (emphasize user impact, timeline, product-side asks)
  4) External customer (soft tone, liability statement, no internal architecture)
Reply with a number.
```

**Chinese prompt:**

```
开始写之前先确认受众类型：
  1) 技术 leader（懂技术，可保留 SQL / 组件表 / 术语）
  2) 不懂技术的高层（CEO / CFO / 大老板 —— 去 SQL 附录，加类比，篇幅 60-100 行）
  3) 产品 / 业务 / PM（重点写用户影响、时间线、是否需要改产品）
  4) 外部客户（语气软化、责任声明、不谈内部架构）
你回一个编号。
```

Store the choice and use it to tune step 6.

### 1. Re-read the conversation (don't write from memory)

Scroll back through the conversation. List:

- Triggering symptom
- Root cause (or "root cause not confirmed" — say so explicitly)
- User-stated business constraints (common rule-outs: "can't add caching" / "architecture frozen" / "must be real-time" / "read-only" — in Chinese: "不能加缓存" / "架构冻结" / "必须实时" / "只读")
- Actions already taken
- Open / unconfirmed items

### 2. Classify every fact before writing

Build a private mental list of every factual claim you plan to include. For each:

- **Source A** – command output / API response in this conversation
- **Source B** – explicit user statement
- **Source C** – your estimation or inference → **must be re-verified or dropped**
- **Source D** – ambient assumption / reasonable default → **same treatment**

**Dangerous claim types that often get fabricated** (watch these specifically):

- Severity level (P0/P1/P2) — only if user or a real alerting system assigned it
- Dates / timestamps — only if in conversation
- Counts and percentages — only from aggregate queries, never hand-summed samples
- Causation — "because X, therefore Y" only if evidence chain exists
- Scope claims — "affects all users" only if confirmed, otherwise "scope pending confirmation"

### 3. Verify C/D facts OR mark them explicitly uncertain

For each C/D class claim, either:

- **Re-verify**: run a fresh read-only query (SHOW VARIABLES / COUNT(*) / API / log grep) and upgrade to A
- **Drop**: remove from the report
- **Mark**: write with ⚠️ and explicit hedge ("~300 transactions, estimated from 20/min × 15min; actual failure count not measured" / "约 300 笔，基于 20笔/分 × 15分 的粗估，实际失败数未统计")

Never let a C/D claim appear in the body as a bare fact.

### 4. Enforce business constraints

Re-read user messages for rule-outs. Any option the user explicitly rejected:

- MUST NOT appear in top recommendations
- If discussed, mark clearly: 🚫 **NOT ALLOWED —** {reason stated by user}

### 5. Plan coverage scope

If the investigation touched multiple components (services, DB instances, regions, environments), write a coverage table.

**English:**

| Component | Handled this pass | Status / Reason |
|---|---|---|
| X | ✅ Fixed | ... |
| Y | ❌ Not addressed | business constraint / unconfirmed / out of scope |
| Z | ⚠️ Suspected | same-class signal but unconfirmed, needs follow-up |

**Chinese:**

| 系统/组件 | 本次处理 | 状态 / 原因 |
|---|---|---|
| X | ✅ 已修 | ... |
| Y | ❌ 未处理 | 业务约束 / 未确认问题 / 不在范围 |
| Z | ⚠️ 疑似 | 有同类信号但未确认，需跟进 |

Without this table, stakeholders assume "all fixed" and later find untouched components → trust lost.

### 6. Write md using the skeleton (all sections required unless marked optional), tuned by audience

#### Base skeleton — English

```markdown
# {System} {Incident/Alert Type} Analysis

> One-liner: **{risk level / core finding}; {what was done / main recommendation}.**  ≤15 words

---

## What happened
{Symptom. For audience 1: original SQL / log / error in full. For audiences 2/4: plain-language version only.}

## Why it happened
{Technical mechanism. Jargon density scales with audience — see variants below.}

## Actual impact (honest assessment)
{Use percentages of capacity, not scary absolutes. Say "no user-visible impact" if true. Separate confirmed from estimated with ⚠️ markers.}

## The real issue
{If alert noise vs real issue: distinguish. List what's actually hiding behind the noise.}

## What we did
| Where | Before | After |
| ... | ... | ... |

## Scope of this pass  【REQUIRED if multi-component; simplify for audience 2】
{Coverage table from step 5.}

## Follow-ups
{Non-urgent items.}

### What NOT to do  【REQUIRED, ≥4 items — ALL audiences】
- ❌ **{antipattern}** — {why not}
- ❌ **{rejected option}** — 🚫 {user's rejection reason}
- ...

{AUDIENCE-SPECIFIC CLOSER — see variants below}

---

## Appendix · Evidence (read-only, {date})
{Raw command outputs, SQL, API responses, timestamps. REQUIRED for audience 1; OMIT entirely for audiences 2 and 4; KEEP but trim for audience 3.}
```

#### Section-name map: English ↔ 中文

When writing in Chinese, swap the English headings for these:

| English (default) | 中文 |
|---|---|
| `One-liner:` | `一句话：` |
| `## What happened` | `## 发生了什么` |
| `## Why it happened` | `## 为什么会这样` |
| `## Actual impact (honest assessment)` | `## 实际影响（诚实评估）` |
| `## The real issue` | `## 真正的问题` |
| `## What we did` | `## 已经做了什么` |
| `## Scope of this pass` | `## 本次覆盖范围` |
| `## Follow-ups` | `## 后续建议` |
| `### What NOT to do` | `### 不要做的事` |
| `## Appendix · Evidence (read-only, {date})` | `## 附录 · 核查证据 (只读, {date})` |
| `≤15 words` (one-liner length limit) | `≤40字` |

#### Skeleton variants by audience

| Dimension | 1-Technical lead | 2-Non-technical exec | 3-PM / Business | 4-External customer |
|---|---|---|---|---|
| **Target length** | 100-200 lines | 60-100 lines | 80-150 lines | 60-120 lines |
| **SQL / log verbatim** | Body + appendix | Appendix only (or omit) | Appendix yes, body no | **Remove entirely** |
| **Jargon** | Terms + parenthetical notes OK | **Zero jargon**, analogies required | Moderate, common terms unflagged | Zero jargon + soft language |
| **"The real issue" section** | Keep | Merge into "Why it happened" | Keep | Remove |
| **Coverage table** | Full table | Simplified to "Fixed / Not fixed" two columns | Full table + user impact column | "Handled / Following up" two rows |
| **Appendix** | Required | **Remove** | Trimmed | **Remove** |
| **Closer section (EN / 中文)** | `## Executive Summary (for Tech Lead)` / `## 执行摘要（给技术 leader）` | `## One-Line for the Executive` / `## 一句话给老板` | `## Executive Summary (for PM / Business)` / `## 执行摘要（给 PM / 业务）` | `## A Note to Our Customer` / `## 致客户说明` |

#### Closer variants (REQUIRED, pick one by audience)

**1-Technical lead** (5-row summary)

English:

```markdown
## Executive Summary (for Tech Lead)  【REQUIRED】
- **Risk level**: Low / Medium / High ({reason})
- **Decision needed?**: No / Yes ({what decision})
- **Resources**: {person-days / budget / external dependencies}
- **Timeline**: {urgent / this week / any time / already done}
- **Open items**: {if any}
```

中文:

```markdown
## 执行摘要（给技术 leader）  【REQUIRED】
- **风险等级**：低 / 中 / 高 ({reason})
- **是否需要决策**：否 / 是 ({what decision})
- **资源需求**：{人天 / 预算 / 外部依赖}
- **时间线**：{紧急 / 本周 / 可随时 / 已完成}
- **待跟进**：{如果有开放项}
```

**2-Non-technical executive** (3-row, colloquial + analogy)

English:

```markdown
## One-Line for the Executive  【REQUIRED】
- **Is this dangerous right now?**: {analogy, e.g. "like a shelf in the warehouse came loose, but nothing fell off"}
- **Do you need to decide anything?**: {no — the engineering team can handle it / yes — we need {X} budget / people}
- **When will it be fully clean?**: {specific date + one-sentence guarantee}
```

中文:

```markdown
## 一句话给老板  【REQUIRED】
- **现在危险吗**：{类比，例如"像是仓库货架有一格松了，但没有货掉下来"}
- **要不要您拍板**：{否——技术团队自己能搞定 / 是——需要 {X} 预算 / 人力}
- **什么时候能彻底干净**：{具体时间点 + 一句保证}
```

**3-PM / Business** (4-row, users + timeline)

English:

```markdown
## Executive Summary (for PM / Business)  【REQUIRED】
- **Users affected**: {count or percentage + what they felt}
- **Impact on launch or features?**: No / Yes ({which feature, which window})
- **Timeline**: {done / this week / next sprint}
- **What product side needs to do**: None / {specific ask}
```

中文:

```markdown
## 执行摘要（给 PM / 业务）  【REQUIRED】
- **有多少用户受影响**：{人数或百分比 + 感知程度}
- **会不会影响发布/功能**：{否 / 是（哪个功能、什么时间窗）}
- **时间线**：{已修完 / 本周 / 下迭代}
- **需要产品侧配合的事**：{无 / 有 —— 具体是 ...}
```

**4-External customer** (softened, no internals)

English:

```markdown
## A Note to Our Customer  【REQUIRED】
- **What happened**: {in plain language, without internal architecture}
- **What we've done**: {fix / monitoring / process improvement}
- **What we're doing to prevent recurrence**: {concrete commitment, e.g. post-mortem, monitoring, periodic audit}
- **Questions?**: {contact / SLA reference}
```

中文:

```markdown
## 致客户说明  【REQUIRED】
- **事件摘要**：{发生了什么，不讲内部架构}
- **我们已经做的**：{修复 / 监控加强 / 流程改进}
- **后续保障**：{具体承诺，例如复盘、监控、定期巡检}
- **如有疑问**：{联系方式 / SLA 条款引用}
```

## Pre-send self-check (all must pass)

1. ⬜ **Language confirmed** — output language is EN (default) or 中文 (per rule), consistent throughout
2. ⬜ **Audience confirmed** (1/2/3/4) — either user picked or conversation made it unambiguous
3. ⬜ Every number / date / severity / percentage in the body has a Source A/B tag in my head (or ⚠️ hedge if unavoidable)
4. ⬜ Every number has a time-window label in context (24h / this incident / past week)
5. ⬜ No fabricated facts — didn't invent severity level, date, component name, or anything else not in conversation
6. ⬜ Before/after numbers have matching measurement basis / 口径 (or mismatch noted)
7. ⬜ Severity / color claims match the actual thresholds used in the code or monitor
8. ⬜ "What NOT to do" / "不要做的事" section exists with ≥4 items
9. ⬜ Coverage section exists (if multi-component) and marks every component (simplified form OK for audience 2)
10. ⬜ Closer section matches the chosen audience variant and uses that variant's fields
11. ⬜ Rejected options carry 🚫 NOT ALLOWED, not soft language like "not feasible at this time" / "暂不可行"
12. ⬜ Jargon density and SQL/log inclusion match the audience row in the variants table (audiences 2/4 body must be zero-jargon; audiences 2/4 appendix must be omitted)

Fail any → fix before sending. Show the user a checklist of pass/fail when they ask "are you sure this is OK?" / "确定没问题吗".

## Real pitfalls (typed, from actual runs)

Pitfall examples are kept in their original language — authenticity beats translation. Column headers and Fix text are in English.

| Type | Pitfall | Symptom | Fix |
|---|---|---|---|
| **Number** | Hand-sum as truth | "~676" was a 6-sample sum; real total 4,266 (6× off) | Aggregate query, never hand-sum |
| **Number** | Time window mix | 24h true value vs 40h aggregation bucket compared directly | Tag time window next to every number |
| **Number** | Scary absolute | "583s CPU/day" sounds bad; actually 0.7% | Give percentage of capacity, not bare seconds |
| **Fact fabrication** | Invented severity | subagent wrote "P1" when no one assigned it | Severity comes only from user / alerting system, never from Claude |
| **Fact fabrication** | Invented date | "日期: 2026-04-20" when conversation had none | Dates only if explicitly in conversation |
| **Fact fabrication** | Invented causation | "because X therefore Y" without evidence | Explicit evidence chain or "pending confirmation" / "待确认" |
| **Uncertainty leak** | Estimate looks like fact | "约 300 笔" in body, disclaimer buried in sentence | ⚠️ marker + explicit "estimation basis" |
| **Uncertainty leak** | Suspected state unmarked | "suspected leak" in coverage table read as confirmed | Use ⚠️ or 🟡 column; separate confirmed from suspected |
| **Language** | SQL shortening | `SELECT region_id FROM T` vs full `WHERE 1 LIMIT 0, 18446744073709551615` — reader asked which is which | SQL always complete parameterized template |
| **Language** | Jargon in body | "复合索引前导列" / "OOMKilled exit 137" / "cgroup" | Plain translation in body, term in parens or appendix |
| **Language** | Mixed language in one report | Half English half 中文 headings from unclear language choice | Step 0 Language policy check; one language per document |
| **Scope** | Missed coverage | Didn't say which DB instances were covered; reader found MongoDB still red | Coverage section required |
| **Scope** | Under-weighted finding | "service X also has leak risk" mentioned casually — it's actually a bigger problem | If incident discovery uncovers wider issue, elevate it |
| **Constraint** | Rejected option in top slot | User said "can't add caching" / "不能加缓存"; report still recommended Redis first | Re-rank after every constraint, 🚫 NOT ALLOWED for dropped |
| **Constraint** | Soft rejection language | "not feasible at this time" / "暂不可行" instead of 🚫 — reader might push to overturn | Use explicit emoji + reason |
| **Structure** | Too long | First draft 414 lines, reader gave up at 100 | Target 100-200 lines, split if more |
| **Structure** | No TL;DR | Report jumps into details without a 1-line summary | First section is always `> One-liner: ...` ≤15 words / `> 一句话：...` ≤40字 |
| **Structure** | No "don't do" section | Reader or next engineer repeats same mistake | REQUIRED section, ≥4 items |
| **Structure** | No closer / wrong closer for audience | Reader has to dig through the whole body, or a CEO got the 5-row technical summary meant for a tech lead | REQUIRED closer matching chosen audience variant |
| **Audience** | Used default closer without asking | Wrote a technical summary for a PM who needed user impact + timeline | Always run step 0 first; skip only if conversation is already unambiguous |
| **Evidence** | "all OK" no table | User asked "are you sure?"; answered with vibe | Show checklist pass/fail table |

## Output location

Use this order:

1. If the user gives an explicit output path, write there.
2. If a private stakeholder report archive is configured, write there:

```text
${STAKEHOLDER_WRITEUPS_DIR}/reports/YYYY/MM/YYYY-MM-DD_查询人_修复人_主题.md
```

Then update the archive indexes:

```text
${STAKEHOLDER_WRITEUPS_DIR}/index.md
${STAKEHOLDER_WRITEUPS_DIR}/index.json
```

If the archive is a git checkout, commit and push after writing unless the user explicitly says local-only:

```text
cd "${STAKEHOLDER_WRITEUPS_DIR}"
git add reports index.md index.json
git commit -m "Add stakeholder writeup: <topic>"
git push
```

3. If no archive is configured, default to `<project-root>/docs/<topic>-analysis.md`. If no `docs/` convention exists, ask first.

Private archive rules:

- The archive should be private if reports include internal system names, people, logs, SQL, customer context, or operational details.
- Do not store API keys, passwords, tokens, private keys, raw customer secrets, or full credential-bearing URLs.
- Use a searchable filename: `YYYY-MM-DD_查询人_修复人_主题.md`.
- Use today's local date only when the report context does not specify a concrete date.
- If query person or repair person is not confirmed, use `unknown`; do not invent names and do not ask an extra question only for the filename.
- If topic is not obvious, derive a short readable topic from the confirmed symptom/system.

## When this skill updates

When you hit a new pitfall — add a row to the table above with its type (Number / Fact fabrication / Uncertainty leak / Language / Scope / Constraint / Structure / Audience / Evidence). The skill gets stronger each iteration.
