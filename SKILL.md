---
name: chain-skill
description: >
  Use when reviewing, refactoring, or modularizing a set of skills into explicit
  parent-child chains: analyze usage scenarios, remove redundancy, build parent-child
  skill chains, and hook lower-level skills. Triggers on "inspect skills", "chain skills",
  "refactor skills", "modularize skills", "parent-child chain", or "skill audit".
---

# Chain Skill

Review a collection of skills and refactor it into a non-redundant, modular
hierarchy with clear parent-child relationships.

## 1. Inventory

List the skills under review and capture for each:

- `name` and `description`
- Trigger conditions
- Core actions (what it does)
- Inputs/outputs
- Dependencies on other skills or tools

### Self-developed skills

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

### External (installed) skills

| Skill | Source | Role | Hooked by |
|---|---|---|---|
| `grill-me` | `vinvcn/mattpocock-skills-zh-cn` | 持續追问式計畫壓力測試（wrapper → `grilling`） | `assign-maus`（指派任務前可選） |
| `grilling` | `vinvcn/mattpocock-skills-zh-cn` | 實際執行 grill 流程 | `grill-me` |

## 2. Analyze usage scenarios

For each skill, answer:

- When does a user or agent reach for it?
- What problem does it solve that no other skill covers?
- Which skills are always used together?

Group skills that share triggers or are always co-invoked.

## 3. Remove redundancy

- Merge skills whose descriptions overlap and whose bodies duplicate guidance.
- Split skills that mix unrelated concerns (e.g., install + publish).
- Keep one source of truth for shared reference (e.g., GitHub publishing steps).

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
```

## 5. Hook lower-level skills

When a parent needs a child's capability, reference it explicitly:

- In prose: "Run the `audit-skill` skill before publishing."
- In a checklist: "Prerequisite: `repo-pull` completed."
- In code: invoke the child skill by name.

Do not duplicate the child's steps in the parent; hook instead.

### Optional visualization hook

After completing a refactor (Step 4), optionally invoke `viz-dboard` to render a mindmap of the resulting hierarchy:

- Trigger: user asks to "visualize", "mindmap", or "show skill map"
- Input: the parent-child chains defined in Step 4
- Output: SVG file (e.g., `skill-map.svg`)
- This is a **conditional hook** (dashed line in mindmap): only invoked on explicit request, not part of the core workflow.

## 6. Validate

- Every skill has a unique, non-overlapping trigger.
- No skill contains duplicated guidance from another.
- Parent-child references are accurate and minimal.
- Each skill still passes its own tests (if any).
- **Before finalizing a refactor, hook the chain `chain-skill → rev-skill → viz-dboard`**: refactor is a review goal, so `rev-skill` steps 8–9 judges rule coherence and then hooks `viz-dboard` to verify doc-layer and code-layer demo compliance. A publish/push workflow does **not** hook `viz-dboard` — no drawing needed. Do not duplicate either check here — each is the single source of truth for its own layer.

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

## Notes

- This skill is a reference process, not an automated refactor tool.
- Apply changes incrementally: refactor one chain at a time.
- Keep `audit-skill` as a gate before publishing or installing any skill.
- Logic check: 2026-11-22 — renamed from `inspect-skill`; frontmatter/description/title updated, all parent-child references rewritten, stray `EOF`/`echo` removed. Second pass: `rev-skill` inventory row corrected (alias + rename + logic check, not Chinese conversion); Step 6 now hooks the chain `chain-skill → rev-skill → viz-dboard` for doc/code compliance instead of duplicating them.