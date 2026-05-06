---
name: project-handoff
description: Generate a comprehensive handoff document when the user wants to continue a long-running project conversation in a new chat. Use this whenever the user says things like "สรุปทุกอย่างให้หน่อยจะไปขึ้น chat ใหม่", "create a handoff doc", "this chat is too long, summarize for next chat", "ช่วยสรุปสำหรับ chat ใหม่", or any request to consolidate project context, decisions, working agreements, and recent work into a portable .md file. Also use when the conversation has spanned many turns on the same project and continuity into a fresh chat is needed. The output must capture role structure, communication rules, tech stack, architectural decisions (closed vs open), recent work, open bookmarks, and onboarding instructions for the next chat — so a fresh Claude instance can resume work without losing context.
---

# Project Handoff Document Generator

A skill for producing a comprehensive, portable handoff document that lets a user continue a long project conversation in a new chat without losing context. The document is written for the **next Claude instance** to read, but in language the user can also verify.

## When this skill triggers

The user is in a long-running project conversation and wants to start a fresh chat to escape context-window slowdown. They need a single `.md` file that captures everything the next Claude needs: who the user is, what the project is, decisions already made, working agreements, recent work, open items, and onboarding steps.

Common phrasings that should trigger this skill:

- "สรุปทุกอย่างให้หน่อย จะไปขึ้น chat ใหม่"
- "this chat is too long, summarize for the next one"
- "create a handoff doc / handoff brief / context doc"
- "I want to continue this in a new chat — give me a summary"
- "เขียน handoff ให้ Claude ตัวต่อไป"

## Core principles

**Write for the next Claude, not for the user.** The doc is a context dump consumed by an AI. Use second-person ("you") sparingly; describe roles, rules, and decisions as facts.

**Imperative + scannable.** Section headers, tables, bullet lists. The next Claude should be able to grep for an answer without reading linearly.

**Closed vs open distinction is critical.** The next Claude must not relitigate decisions already made. Mark them clearly as `[DECISION]` blocks with rationale. Open items go in a separate "Open Bookmarks" section.

**Capture working agreements verbatim.** Communication style, language rules, role boundaries — these are easy to lose in translation. Quote the user's own words when the rule originated as a user instruction.

**Include enough to resume work, no more.** Don't dump the entire conversation. Distill.

## Information to extract from the current conversation

Before writing, scan the conversation for these categories. Some may not apply to every project.

### 1. User profile
- Name, role, company, business context
- Technical level (developer / non-technical PM / domain expert)
- Languages used (response language preferences)
- Primary tools/IDEs

### 2. Project identity
- Project name, goal, scope
- Phase/milestone currently in progress
- Why this project exists (business problem being solved)

### 3. Role structure
- Who does what — user / Claude Chat / Claude Code / other AIs / human collaborators
- Working loop (e.g., "user → Claude Chat plans → Claude Code executes → user verifies → commit")
- Approval gates (what requires explicit user approval)

### 4. Communication rules
- Language per channel (e.g., Thai with user, English for Claude Code prompts, Thai for commits)
- Style (short paragraphs, no overwhelm, ask one question at a time)
- Tools to use for elicitation (e.g., `ask_user_input_v0` for choices)
- Things the user has explicitly forbidden ("don't ask me about implementation detail")
- How the user signals frustration ("ลึกไป" = back off)

### 5. Tech stack & infrastructure
- Confirmed choices with rationale
- **Rejected alternatives** — record what was NOT chosen and why (prevents the next Claude from re-suggesting them)
- Repos with URLs and visibility
- Hosting, deployment paths, server URLs
- Memory/sync infrastructure if any

### 6. Architectural decisions (closed)
- Decision name + chosen option + rejected options + rationale
- These are off-limits for the next Claude to revisit

### 7. Domain knowledge
- Taxonomies, naming conventions, business rules
- Glossary of terms specific to this project

### 8. Recent work
- Last few sessions' outcomes
- Commit hashes / tags / PR links
- Bugs fixed, features shipped

### 9. Open bookmarks
- Pending tasks (priority-ordered)
- Known issues that don't block current work
- Phase 2+ candidates

### 10. Critical rules / mistakes to avoid
- Rules learned the hard way (e.g., "always run X after Y")
- Mistakes Claude made in this conversation that the next Claude should avoid repeating
- Be honest about errors — the next Claude benefits more from candor than from face-saving

### 11. Onboarding for new chat
- Day-1 steps: what to verify, what to read, what to ask
- How to load context (file uploads, repo pulls, memory sync)
- A "cheat sheet" table for common situations

## Document structure

Use this exact section order. Adjust section titles only if the project domain demands it (e.g., a creative writing project may not need "Tech Stack" but might need "Style Guide"). When skipping a section, omit it cleanly — do not leave placeholder headers.

```
# [PROJECT NAME] — Handoff Document for [next-chat-name]

> Purpose: ...
> Audience: Next Claude instance
> Status: [phase] · [date]

## 1. User Profile
## 2. Role Structure
## 3. Communication Rules
## 4. Tech Stack (or domain-equivalent)
## 5. Repositories / Resources
## 6. Project Structure
## 7. Architectural Decisions (closed — do not revisit)
## 8. Domain Standards / Taxonomies
## 9. Recent Work — current/last session
## 10. Open Bookmarks (pending — not blocking)
## 11. Next Major Work (priority-ordered)
## 12. Critical Rules (do not break)
## 13. Mistakes to avoid (lessons learned)
## 14. Git Commit History (key references)
## 15. Key Docs & References
## 16. Instructions for New Chat
## 17. Day-1 Steps for New Chat
## 18. Cheat Sheet — quick reference

## END OF HANDOFF
> Created: [date]
> Last session ended at: [time/state]
> Memory state: [synced/pending]
> Code state: [latest commits]
> Open work: [next priority]
```

Sections may be merged when a project is small (e.g., #4 and #5 combine for a non-code project). The minimum viable handoff has #1, #2, #3, #7, #9, #10, #16.

## Working with the user

### Step 1 — Confirm scope before writing

Ask one short question: "ครอบคลุมตั้งแต่ session ไหนถึงไหนครับ?" (or in EN). Three possibilities:
- **All sessions** — pull from full conversation history (use transcript tool if available)
- **This chat only** — current visible conversation
- **Recent N sessions** — user specifies

If transcripts/past chat tools are available and the user implies "everything", use them. Don't write from memory if you can read.

### Step 2 — Mine information

Read transcripts/conversation systematically. Take notes on each of the 11 information categories above. If a category has no signal, skip it in the output.

### Step 3 — Draft

Write the document to a file in the appropriate output directory (e.g., `/mnt/user-data/outputs/<project-name>-handoff-<target-chat-name>.md`).

Length guidance:
- **Small project** (a few sessions): 200–400 lines
- **Medium project** (multi-week, several decisions): 400–700 lines
- **Large project** (multi-month, deep architecture): 700–1200 lines

Going over 1200 lines usually means the user will benefit more from splitting into multiple files (handoff + appendix) than from one monolithic doc.

### Step 4 — Present and explain

Use `present_files` to share the doc. Briefly summarize what's inside (sections, line count). Tell the user how to use it in the new chat:

> "Upload this file in the first message of the new chat, then say: 'นี่คือ handoff doc · อ่านครบก่อนเริ่มงาน · ยืนยันเข้าใจ role structure ก่อน'"

Don't explain the doc's contents in detail — the doc is for the next Claude, not for re-reading in chat.

## Writing patterns

### Decision blocks

```markdown
### [DECISION 1] [Short name]

- **Chosen:** [option name]
- **Rejected:** [other options]
- **Rationale:**
  - reason 1
  - reason 2
```

### Role table

```markdown
| Role | Who | Responsibility | Language |
|---|---|---|---|
| ... | ... | ... | ... |
```

### Cheat sheet

```markdown
| If | Then |
|---|---|
| User asks business/scope question | Answer directly · you are consultant |
| User asks implementation detail | Send to Claude Code first |
| ... | ... |
```

### Mistakes section — be honest

```markdown
## Mistakes to avoid (lessons learned)

1. ❌ I assumed [X] without verifying → caused [Y]. Next time: check [Z] first.
2. ❌ I edited [wrong file] because I confused [A] vs [B]. Next time: confirm scope before edit.
```

Include 3–5 candid items. The next Claude learns more from these than from triumphant summaries.

## What NOT to include

- Long verbatim transcripts of conversations — distill instead
- Tutorial-style explanations of basic concepts — assume the next Claude knows the field
- Speculation about future direction beyond what the user has confirmed
- Praise for the user or for the project — the doc is operational, not promotional
- Your own emotional state or apologies

## Output file naming

Default pattern: `<project-slug>-handoff-<target-chat-label>.md`

Examples:
- `acme-erp-handoff-consult-v2.md`
- `personal-website-handoff-redesign-chat.md`
- `book-draft-handoff-chapter-7-chat.md`

If the user has named the new chat (e.g., "consult ERP2"), use that label. If not, use a generic suffix like `next-chat` or omit it.

## Variable handling

Skills should be portable across projects. Treat these as variables to extract from the conversation:
- `<PROJECT_NAME>` · `<USER_NAME>` · `<COMPANY>` · `<DOMAIN>` · `<TECH_STACK>` · `<REPOS>` · `<NEW_CHAT_LABEL>`

Do not hardcode any specific user, project, or company name in the output structure. The skeleton is universal; the contents are filled from each conversation's actual data.

## A few examples of when this skill is the right call

**Trigger example 1:**
> User: "แชทนี้ยาวมากแล้ว โหลดช้า · ไปขึ้น chat ใหม่ดีกว่า · ช่วยสรุปทุกอย่างให้หน่อย"
→ Trigger this skill. Confirm scope, then produce handoff doc.

**Trigger example 2:**
> User: "Can you write a brief I can paste into a new conversation? Include all the decisions we've made about architecture, the tech stack, and what's left to do."
→ Trigger this skill.

**Non-trigger example:**
> User: "What did we decide about authentication last week?"
→ Don't trigger. The user wants a single answer, not a full handoff.

**Non-trigger example:**
> User: "Summarize this article."
→ Don't trigger. Not a project handoff.
