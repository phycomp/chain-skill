---
name: chain-skill
description: >
  Use when reviewing, refactoring, or modularizing a set of skills into explicit
  parent-child chains: analyze usage scenarios, remove redundancy, build parent-child
  skill chains, hook lower-level skills only when needed, and close the loop with
  doc-layer + code-layer compliance. Triggers on "inspect skills", "chain skills",
  "refactor skills", "modularize skills", "parent-child chain", "skill audit",
  "skill loop", "閉環", "skill 閉環", or "skill 開發規格".
---

# Chain Skill — Refactor as a Closed Loop

Review a collection of skills and refactor it into a non-redundant, modular
hierarchy with clear parent-child relationships. The refactor itself must pass
the same closed loop it teaches.

## Core invariant (the loop)

Every skill change follows one loop. Do not skip a stage; do not add a stage
that has no trigger.

```
Usage scenario → Strict logic → Doc compliance → Code compliance
      ↑                                              │
      └──────── need-based hook + distill ←──────────┘
```

| Stage | Question it answers | Evidence |
|---|---|---|
| 1. Usage scenario | When is this invoked, and what problem is unique to it? | Inventory + scenarios |
| 2. Strict logic | Does every rule follow the invariant, with no stale/contradictory rule? | `rev-skill` Logic Consistency Check |
| 3. Doc compliance | Does the skill's own SKILL.md match current reality? | Read the doc, compare to code |
| 4. Code compliance | Does the actual demo/implementation obey the documented backend/widget/API? | Open the real `app.py` / code |
| 5. Need-based hook | Is a child skill actually required for this goal? | Goal predicate, not skill name |
| 6. Distill / close loop | Is the corrected rule recorded back into the source of truth? | Write into SKILL.md, re-audit |

The loop is closed only when stage 6 writes the correction back and re-runs the
audit. Reporting a problem without writing it back is an open loop.

## 1. Inventory

List the skills under review and capture for each:

- `name` and `description`
- Trigger conditions
- Core actions (what it does)
- Inputs/outputs
- Dependencies on other skills or tools

### Self-developed skills

> **唯一來源宣告：** `~/.config/skill-timer/self-developed.list`（由 `skill-timer` 擁有）是「哪些 skill 算自行開發」的 single source of truth。下表**僅供重構盤點索引**，不是判定依據。增刪 skill 時先改 allow-list，再同步下表。

Example inventory (self-developed skills):

| Skill | Triggers | Core actions | Lower-level hooks |
|---|---|---|---|
| `paperclip-launch` | install/launch Paperclip | managed CLI, service, health, repairs | `repo-pull` (for env-profile) |
| `assign-maus` | deploy/assign OpenMausBot | create bots/channels, kickoff, assign tasks | `paperclip-launch` (if Paperclip needed) |
| `audit-skill` | any skill work | security scan, manifest, verdict | none |
| `repo-pull` | clone/update repo | git clone/pull, install script | none |
| `rev-skill` | review/audit a skill, check rule logic | security alias → `audit-skill`; rename; top-to-bottom Logic Consistency Check | `audit-skill` (security gate) |
| `publish-to-github` | publish to GitHub | git init, create repo, push, release | `audit-skill` (before publish) |
| `quant-algo-auto` | quant trading/channel ops | strategy, backtest, risk, content | none |
| `viz-dboard` | visualize skill hierarchies | inventory, build hierarchy, render SVG | `chain-skill` (optional, post-refactor) |
| `social-router` | social content loop | produce → publish → feedback orchestration | platform skills (by goal) |
| `sci-router` | scientific topic distill → references/paper (parent) | 7-stage local+arXiv pipeline | `sci-research-distill` (if goal = product concept → market/business plan); `sci-concept-valid` (if goal = concept verification) |
| `sci-research-distill` | product concept → market/business plan (child) | 11-step artifact→market research | none (child; hooked by sci-router) |
| `sci-concept-valid` | verify physical/science concept (child) | evidence-bounded literature+calculation check | none (child; hooked by sci-router) |
| `file-organizer` | general file/folder organization | analyze, dedup, propose structure, cleanup | `file-distill` (when goal = topic sedimentation) |
| `file-distill` | evidence-based topic sedimentation | 6-phase classify with move log | none (child) |

### External (installed) skills

| Skill | Source | Role | Hooked by |
|---|---|---|---|
| `grill-me` | `vinvcn/mattpocock-skills-zh-cn` | 持續追問式計畫壓力測試（wrapper → `grilling`） | `assign-maus`（指派任務前可選） |
| `grilling` | `vinvcn/mattpocock-skills-zh-cn` | 實際執行 grill 流程 | `grill-me` |

## 2. Analyze usage scenarios

For each skill, answer:

- When does a user or agent reach for it?
- What problem does it solve that no other skill covers?
- Which skills are always used together?

Group skills that share triggers or are always co-invoked. A trigger that is
not resolvable by a later rule is a stale trigger — remove it or write the rule.

## 3. Remove redundancy

- Merge skills whose descriptions overlap and whose bodies duplicate guidance.
- Split skills that mix unrelated concerns (e.g., install + publish).
- Keep one source of truth for shared reference (e.g., GitHub publishing steps).
- If two skills teach the same rule, the child owns it and the parent hooks it —
  neither duplicates it.

## 4. Build parent-child chains

Design a hierarchy where:

- **Parent skills** orchestrate a workflow and delegate to children.
- **Child skills** are atomic, single-purpose capabilities.

Example chains:

```
paperclip-launch
  └── repo-pull          (pull env-profile before install)

publish-to-github
  └── audit-skill        (security gate before publish)

assign-maus
  └── paperclip-launch   (only if Paperclip must be running)

chain-skill
  └── rev-skill          (logic coherence; review goal only)
        └── viz-dboard    (doc layer + code layer compliance)

sci-router (parent)
  ├── sci-research-distill   (child; only when goal = product concept → market/business plan)
  └── sci-concept-valid      (child; only when goal = evidence-bounded concept verification)

file-organizer
  └── file-distill       (only when goal = evidence-based topic sedimentation)
```

## 5. Hook lower-level skills — only when needed

When a parent needs a child's capability, reference it explicitly:

- In prose: "Run the `audit-skill` skill before publishing."
- In a checklist: "Prerequisite: `repo-pull` completed."
- In code: invoke the child skill by name.

Do not duplicate the child's steps in the parent; hook instead.

### Need-based predicate

Hook by **workflow goal**, not by skill name:

- Goal is a **logic/compliance review** → hook `rev-skill` → `viz-dboard`.
- Goal is **publishing/pushing** → hook `audit-skill` only; do **not** hook
  `viz-dboard` (no drawing or demo verification needed).
- User explicitly asks to "visualize / mindmap / show skill map" → hook
  `viz-dboard` for the mindmap.
- Within **sci-router (parent)** workflow: hook a child only by goal —
  - Goal is **product concept → market/business plan** → hook `sci-research-distill`.
  - Goal is **evidence-bounded concept/model verification** → hook `sci-concept-valid`.
  Do not duplicate children's steps in the parent; each child owns its rules.
- Goal is **evidence-based topic sedimentation** (classify a messy dir into subtopic folders with a move log) → hook `file-distill`; general tidy/dedup/structure-suggest stays in `file-organizer`.
- Two workflows sharing a schedule slot but holding different goals (e.g. skill-compliance check `[report-only]` vs personal-profile push `[publishes]`) → order them at the **systemd layer** (`Wants/After=` between services), not by hooking one skill body into another. Each service keeps its own invariant; the timer never publishes and the push service never audits.

A hook without a matching goal is a violation — it adds cost and blurs ownership.

### Optional visualization hook

After completing a refactor (Step 4), optionally invoke `viz-dboard` to render a mindmap of the resulting hierarchy:

- Trigger: user asks to "visualize", "mindmap", or "show skill map"
- Input: the parent-child chains defined in Step 4
- Output: SVG file (e.g., `skill-map.svg`)
- This is a **conditional hook** (dashed line in mindmap): only invoked on explicit request, not part of the core workflow.

## 6. Validate — close the loop

Run the stages in order; a later stage cannot replace an earlier one.

- **Scenario coverage** — every trigger in the inventory resolves to a rule.
- **Strict logic** — `rev-skill` Logic Consistency Check passes; no rule
  contradicts the invariant, no stale rule survives a behavior change.
- **Doc compliance** — the skill's own SKILL.md matches current reality. A
  stale "two demos" entry after the features were folded into one `app.py` is
  a doc-layer failure, even if the code is correct.
- **Code compliance** — open the actual code. A README line, a Demo-Tools
  table, or a `legacy` label does **not** make a violating implementation
  compliant. If code contradicts a rule, fix the code or remove the demo from
  the documented set — never paper over it with a nuance clause.
- **Need-based hook** — `chain-skill → rev-skill → viz-dboard` is used for a
  review goal; a publish/push workflow skips `viz-dboard`. Do not duplicate
  either check here — each is the single source of truth for its own layer.
- **Path invariant** — the canonical root is the traditional-character
  `~/薦家/布球/` (`薦`, not 荐). Any `~/荐家/` is a logic error: it creates a
  second empty tree and makes `cd A 2>/dev/null || cd B` pick the wrong one.
  Use the literal `~/薦家/...` and `test -d` first; never the fallback pattern.
- **Loop closure** — corrections are written into SKILL.md (not merely
  reported), then audit + drift verify are re-run. Record the date and change
  in the `Logic check:` line.

Before finalizing a refactor, hook `chain-skill → rev-skill → viz-dboard` for
the review goal; `rev-skill` judges rule coherence and hooks `viz-dboard` to
verify doc-layer and code-layer demo compliance.

## Chinese conversion (merged from rev-skill)

When converting a skill into Traditional Chinese (繁體中文):

- Preserve code, commands, file paths, URLs, identifiers, and API names.
- Translate prose, headings, explanations, and comments.
- Keep trigger keywords (English and/or Chinese) for discoverability.
- Ensure `description` still starts with "Use when...".
- Validate frontmatter and relative links after editing.

## Common mistakes

- Merging skills that have different trigger conditions.
- Creating parent skills that just repeat child steps.
- Hooking a child that is not actually needed.
- Leaving orphaned skills with no clear owner.
- Changing the doc but not verifying the code (open loop).
- Using a `legacy` / `exception` label to excuse a rule violation.
- Using `~/荐家/` or a `cd A || cd B` fallback for canonical paths.

## Notes

- This skill is a reference process, not an automated refactor tool.
- Apply changes incrementally: refactor one chain at a time.
- Keep `audit-skill` as a gate before publishing or installing any skill.
- `rev-skill` owns the Logic Consistency Check; `viz-dboard` owns demo-code
  compliance. This skill owns the loop and the goal-based hook decision.
- Logic check: 2026-11-22 — renamed from `inspect-skill`; frontmatter/description/title updated, all parent-child references rewritten, stray `EOF`/`echo` removed. Second pass: `rev-skill` inventory row corrected (alias + rename + logic check, not Chinese conversion); Step 6 now hooks the chain `chain-skill → rev-skill → viz-dboard` for doc/code compliance instead of duplicating them.
- Logic check: 2026-11-23 — rewritten as the **loop methodology** (user request: 「依照使用情境需求、嚴格邏輯、規則清楚、文件層合規、程式層合規、上下級鏈、分工明確、需要時才 hook、形成閉環、沉澱成 loop」). Added Core invariant loop table, need-based hook predicate, doc/code compliance stages, path invariant, and loop-closure requirement. Name kept as `chain-skill` per user decision; no new `loop-skill` created. RED evidence reused from prior observations (stale two-demo doc, raw matplotlib source misused, 荐家 path, unnecessary viz-dboard hook, doc-only fix).
- Logic check: 2026-11-23 (second pass) — declared `skill-timer`'s allow-list as the **single source of truth** for "self-developed"; the Inventory table is now explicitly **index-only**. Same declaration added to `skill-router`'s Available skills table so the three lists cannot drift silently. No refactor rules duplicated.
- Logic check: 2026-11-23 (social-router) — added the social content skill to allow-list first, then synced this index table and `skill-router`'s table. Index-only; allow-list remains authoritative. Renamed `social-loop` → `social-router` before first push.
- Logic check: 2026-11-23 (parent-child chains) — added two chains per user request: (1) `sci-concept-valid` as shared child hooked by both `sci-router` and `sci-research-distill`; (2) `file-distill` as child of `file-organizer`. Updated Inventory table, Example chains, and need-based predicate. Only chain-skill edited (sub-skills' existing references left as-is).
- Logic check: 2026-11-23 (research chain redesign) — user decided `sci-router` is the single parent with `sci-research-distill` and `sci-concept-valid` as children (replacing the earlier shared-child design). Inventory hooks, Example chains, and need-based predicate updated; file chain unchanged.
