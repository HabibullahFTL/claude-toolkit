---
name: design-doc
description: Read a file and conduct a deep technical interview to produce a detailed design document.
argument-hint: '<file-path> — path to the file or concept you want a design doc for'
---

Read `$ARGUMENTS` and conduct a structured interview to produce a detailed design document. Follow each step precisely.

## Step 1 — Read the file

Use the Read tool to read the file at `$ARGUMENTS`.

If `$ARGUMENTS` is empty, use AskUserQuestion to ask: "Which file or topic would you like to create a design doc for?"

If the file does not exist, use AskUserQuestion to ask: "I couldn't find `$ARGUMENTS`. Please confirm the correct path or describe what you'd like a design doc for."

## Step 2 — Analyze before interviewing

Before asking anything, silently analyze what you read and identify:

- What is known vs. ambiguous
- What decisions are implied but not explained
- What tradeoffs exist that the author may not have considered
- What is missing (error handling, edge cases, scale, security, UX, etc.)

Do NOT ask obvious questions (e.g. "What does this file do?" if it's self-evident). Only ask what you genuinely cannot infer.

## Step 3 — Conduct the interview

Use the AskUserQuestion tool to interview the user. Cover the areas below, but **only ask what is unclear or non-obvious** based on your Step 2 analysis.

Ask one focused question at a time. After each answer, ask a follow-up if the answer raises new ambiguity. Continue until all critical areas are resolved.

**Areas to probe (as relevant):**

- **Purpose & goals** — What problem does this solve? Who uses it and in what context?
- **Technical implementation** — Key algorithms, data flow, dependencies, constraints
- **UI & UX** — If applicable: user flows, edge states, accessibility, responsiveness
- **Inputs & outputs** — Data shapes, validation rules, failure modes
- **Tradeoffs made** — Why this approach over alternatives? What was explicitly rejected?
- **Error handling & edge cases** — What can go wrong? How should it be handled?
- **Performance & scale** — Expected load, bottlenecks, caching, limits
- **Security** — Auth, permissions, data sensitivity, attack surface
- **Integration points** — External systems, APIs, events, side effects
- **Future considerations** — Known limitations, planned changes, what should NOT change

Stop interviewing when you have enough to write a complete, unambiguous design doc.

## Step 4 — Confirm output file path

Before writing, do the following in order.

### 4a — Infer the document type from the interview output

Based on what you learned in the interview, classify the document into one of:

| Type | Extension | When to use |
|------|-----------|-------------|
| Specification | `.spec.md` | Requirements, constraints, interface contracts |
| Plan | `.plan.md` | Roadmap, milestones, tasks, timeline |
| Design | `.design.md` | Architecture, system design, tradeoffs |
| Document | `.doc.md` | General reference, explanation, guide |

Call this the **inferred extension**.

### 4b — Determine the suggested output filename

**If the input already ends in a known pattern** (`.spec.md`, `.plan.md`, `.design.md`, `.doc.md`):
- If the inferred extension matches the input extension → keep the same extension (content hasn't shifted)
- If the inferred extension differs → use the inferred extension (content shifted significantly)
- Compute the next version filename using the same extension:
  - `foo.<ext>.md` → `foo-v2.<ext>.md`
  - `foo-vN.<ext>.md` → `foo-v(N+1).<ext>.md`

**If the input ends in plain `.md`** or has any other extension (`.ts`, `.js`, `.py`, etc.) or no extension:
- Use the inferred extension to construct the suggested filename: `<basename>.<inferred-ext>.md`

### 4c — Ask the user to confirm

Use AskUserQuestion to ask one of the following, depending on the case:

**If the input was a known `.<ext>.md` file** (overwrite vs. version choice):

> "Based on the content, this looks like a **\<type\>**. Where should I write it?
>
> 1. `<current filename>` (overwrite)
> 2. `<next version filename>` (new versioned file)
> 3. Enter a custom filename"

**If the input was a generic `.md`, code file, or had no extension** (inferred filename):

> "Based on the content, this looks like a **\<type\>** — I'll save it as `<inferred filename>`.
>
> 1. `<inferred filename>` ✓
> 2. Enter a custom filename"

If the user enters a custom filename:
- If it contains an extension (e.g. `my-plan.spec.md`) → use as-is
- If it has no extension (e.g. `my-plan`) → append the inferred extension (e.g. `my-plan.plan.md`)

Use the final resolved path as the output path.

## Step 5 — Write the design document

Write the design doc using this structure:

```
# Design Doc: <Title>

## Overview
One paragraph: what this is, what problem it solves, who uses it.

## Goals
- <Goal 1>
- <Goal 2>

## Non-Goals
- <What is explicitly out of scope>

## Inputs & Outputs
### Inputs
| Name | Type | Required | Description |
|------|------|----------|-------------|

### Outputs
| Name | Type | Description |
|------|------|-------------|

## Behavior
Step-by-step description of what happens, in order. Include branching logic and edge cases.

## Error Handling
| Scenario | Expected Behavior |
|----------|-------------------|

## Tradeoffs & Decisions
### <Decision 1>
**Chosen:** <what was decided>
**Rejected:** <alternatives considered>
**Reason:** <why>

## Security Considerations
- <Item>

## Performance Considerations
- <Item>

## Open Questions
- [ ] <Unresolved item, if any>
```

Omit sections that are truly not applicable (e.g. no UI section for a pure utility function).

## Step 6 — Report

After writing the file:

```
✅ Design doc written

FILE:     <output path>
SECTIONS: <comma-separated list of sections included>
OPEN:     <count> open questions  ← omit if zero
```

No extra commentary. Just the report block.
