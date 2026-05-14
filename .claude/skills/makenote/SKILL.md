---
name: makenote
description: Use when the user wants to create a new Obsidian wiki note from a title and outline file, or when the user says things like "帮我写一篇笔记", "创建笔记", "根据大纲写笔记", or provides a note title plus an outline file path. Triggers the full multi-agent pipeline: research → critique → integration → editing.
---

# makenote — Multi-Agent Obsidian Note Creator

## Overview

Orchestrates a 4-agent team to turn a title + outline into a comprehensive Obsidian wiki note. Agents collaborate via shared files in a temp workspace. Controversial issues go through a debate-resolution cycle before final editing.

## Workflow

```
User: Title + Outline path
  ↓
Phase 0: Create empty note + temp workspace
  ↓
Phase 1 (PARALLEL): Researcher 🔍 + Critic 🧠
  ↓
Phase 2: Integrator 🔗 (reads both outputs, resolves controversies, adds analogies)
  ↓
Phase 3: Editor ✍️ (writes final Obsidian note)
  ↓
Update index.md + log.md
```

## Phase 0: Setup

1. Read the outline file the user provides
2. Read `wiki/index.md` to understand existing structure
3. Read `CLAUDE.md` for schema rules
4. Create the empty note with Obsidian frontmatter at the correct wiki path (determine category from outline content)
5. Create temp workspace at `.temp/makenote-<slug>/` with shared files:
   - `00-大纲.md` (copy of outline)
   - `research.md` (template)
   - `critique.md` (template)
   - `争议区.md` (controversy board)
   - `integrated.md` (template)

## Phase 1: Researcher + Critic (Parallel, Background)

Launch both agents simultaneously with `run_in_background: true`. Name them for traceability.

### Researcher Agent Prompt Template

```
You are a professional researcher. Read the outline at <outline_path> and research every topic via web search. Write findings to <workspace>/research.md.

For each topic in the outline:
1. Search for authoritative sources (papers, official docs, tech blogs)
2. Record: core concepts, key data/cases, relationship to other topics
3. Use format: ## Topic Name → 核心概念 → 网上资料汇总(with URLs) → 关键数据/案例 → 与其他主题的关联

If you find contradictory information across sources, flag it in <workspace>/争议区.md.

WRITE TO FILE, do not just report in conversation.
```

### Critic Agent Prompt Template

```
You are a critical fact-checker. Read the outline at <outline_path> and independently verify every claim via web search. Write analysis to <workspace>/critique.md.

For each claim:
1. Search to verify accuracy → mark ✅ accurate / ⚠️ incomplete / ❌ wrong
2. Find counterexamples or edge cases where the claim fails
3. Check if information is outdated (look for 2024-2025 updates)
4. Flag oversimplifications that miss important nuance

Also: identify important topics MISSING from the outline (search for latest developments in the field).

Flag genuine controversies to <workspace>/争议区.md with format:
## 争议 N: [title]
**争议描述**: (why it's controversial)
**待讨论问题**: (specific questions to resolve)
**研究员回应**: (待填写)
**整合者裁决**: (待填写)
```

## Phase 2: Integrator (Sequential, Background)

Launch AFTER both Phase 1 agents complete. The integrator reads research.md, critique.md, and 争议区.md.

### Integrator Agent Prompt Template

```
You are a content integrator and arbiter. Read all three input files. Produce <workspace>/integrated.md.

Step 1: Resolve every controversy in 争议区.md. For each: summarize both sides, make a final ruling with evidence, specify how the note should handle it.

Step 2: Merge research + critique into a complete, accurate note body. Adopt what critique verified as accurate. Fix what critique found wrong. Fill what critique said was missing.

Step 3: Add simple, memorable analogies for difficult concepts. Requirements: everyday scenarios, accurately convey the core mechanism, not oversimplified to the point of distortion.

Step 4: Write the complete integrated draft to integrated.md.

Key principle: when researcher and critic agree → adopt directly. When they disagree → you decide with reasoning.
```

## Phase 3: Editor (Sequential)

After integrator completes, write the final Obsidian note. Use the **obsidian-markdown** skill for proper formatting.

### Key requirements:
- Proper YAML frontmatter (title, date, tags, aliases, status)
- Wikilinks `[[wiki/Category/page]]` for internal links
- Callouts (`> [!tip]`, `> [!warning]`, `> [!info]`) for emphasis
- Tables for comparisons
- Mermaid diagrams where helpful
- Frontmatter `status: done`

### Final note structure (adapt to outline, but generally):
1. 概述 (one sentence summary)
2. Core content (follow outline structure, enriched with research)
3. 比喻词典 (glossary of analogies for difficult concepts)
4. 核心知识点回顾 (key takeaways)
5. 相关概念 (wikilinks to related notes)
6. 来源 (references)

## Phase 4: Update Index and Log

After the note is written:
1. Add new page link to `wiki/index.md` under the appropriate category
2. Add entry to `wiki/log.md` under today's date, noting: what was created, key corrections made, controversies resolved

## Agent Communication Rules

- All agents write to shared files in the temp workspace — never rely on conversation memory
- The controversy board (争议区.md) is the designated channel for cross-agent debate
- Researcher should check 争议区.md for critic's questions and respond
- Integrator makes final rulings on all unresolved controversies
- When research and critique conflict on facts, integrator favors the better-sourced claim

## Common Pitfalls

| Pitfall | Prevention |
|---------|-----------|
| Agents report in conversation instead of writing files | Prompts explicitly say "WRITE TO FILE" |
| Researcher takes too long (>5 min) | Use `run_in_background: true`; check progress via file size |
| Outline has typos or inaccuracies | Critic catches these; integrator fixes in final note |
| Controversies left unresolved | Integrator must explicitly rule on every open controversy |
| Note too long or too short | Target 300-600 lines; adjust depth based on outline complexity |
| Wrong wiki category | Read index.md first to determine where the note belongs |
