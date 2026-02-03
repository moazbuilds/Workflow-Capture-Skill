---
name: capturing-workflows
description: >
  Captures a Claude Code conversation into a structured, reusable workflow document by extracting phases, steps, decisions, dead ends, and reasoning. Use after completing an implementation session, debugging flow, or multi-step process to preserve the approach for reuse. Supports multi-session append. Trigger: capture workflow, save workflow, extract workflow, record workflow, document this workflow, append workflow.
allowed-tools: Read, Glob, Write, Bash
user-invocable: true
---

# Capturing Workflows

## Step 1: Intake

Ask these 3 questions and wait for all answers before proceeding:

1. "What's the goal of this workflow? (one sentence describing what the process achieves)"
2. "What made this approach work? (the key insight or technique that was critical)"
3. "Is this a new workflow, or appending to an existing one?"

Store the answers as:
- `workflow_goal` — the goal sentence
- `key_insight` — the critical technique/insight
- `mode` — "new" or "append"

## Step 2: Discovery

**If mode is "append":**

1. Run `Glob` on `.claude/workflows/*.md`
2. Display each found workflow with its filename and first-line title
3. Ask: "Which workflow should I append to?"
4. If no workflows found, tell the user: "No existing workflows found in `.claude/workflows/`. Starting a new workflow instead." Switch mode to "new"
5. If a workflow is selected, read its full content with `Read`

**If mode is "new":**

1. Check if `.claude/workflows/` exists
2. If not, run: `mkdir -p .claude/workflows`

## Step 3: Extraction

Read the conversation context. Internally work through these QA-CoT questions (do not display to user):

1. What was the overall intent of this conversation?
2. What distinct phases did the work go through?
3. Within each phase, what specific actions were taken?
4. Where were decisions made? What were the alternatives? Why was one chosen?
5. What was tried and abandoned? Why did it fail?
6. What preconditions or dependencies exist between steps?

Structure answers into phases with steps. Each step must have: **how**, **why**, **decisions** (with alternatives and reasoning), **output**, and **dependencies**.

Capture branching logic as "if X then Y, else Z" structures. Record dead ends in a separate list with what happened, why it failed, and the lesson.

**Minimum content threshold:** If the conversation has fewer than 3 procedurally meaningful messages (actions, decisions, outcomes), stop and tell the user: "This conversation doesn't have enough actionable steps to extract a meaningful workflow. Try again after a more implementation-focused session." Do not proceed to Step 4.

**Multiple workflows detected:** If distinct unrelated goals are found, ask: "I see work on [topic A] and [topic B]. Which workflow should I capture? Or should I save them as separate workflows?"

**Append overlap handling:** If mode is "append", compare extracted content against the existing workflow. If overlapping steps are detected (matching phase names or step descriptions), present the specific overlapping sections and ask: "I found overlap in [area]. Should I: (a) merge the new detail into existing steps, (b) replace the overlapping section, or (c) keep both versions labeled with session dates?"

Default to clustering by procedural elements (intent, action, outcome) rather than raw chronological order. Prefer fewer phases with rich detail over many shallow phases.

## Step 4: Draft and Review

Render the extracted workflow using the template from [workflow-template.md](workflow-template.md) so the user sees exactly what will be saved. Display the full workflow in the conversation.

Then say: "Here's the captured workflow. Review it and tell me what to change — add, remove, or rephrase anything. Once you confirm, I'll save it."

Apply the user's feedback. This is exactly one round of review — after adjustments, proceed to Step 5.

## Step 5: Save

1. Auto-generate filename: take `workflow_goal`, convert to kebab-case, truncate to 50 chars max, append `.md`
2. Display: "I'll save this as `.claude/workflows/[name].md` — want a different name?"
3. If user provides an alternative, use that instead

**For new mode:**
- Write the complete workflow to `.claude/workflows/[name].md`
- Include header with goal, key_insight, captured date (today), and status "complete" (or "in-progress" if user indicated more sessions to come)

**For append mode:**
- Write the merged/appended content to the existing file path
- Preserve existing header metadata
- Update the captured date

Confirm: "Workflow saved to `.claude/workflows/[name].md`"

<rules>

## MUST

- Capture branching logic and conditional decisions — never flatten decision trees into linear step lists
- Separate signal from noise — only extract actions taken, decisions made, and reasoning expressed
- Capture the "why" behind each step — every step needs its reasoning
- Record dead ends and failed approaches in a dedicated section
- Save to `.claude/workflows/[name].md` — create the directory if it does not exist

## MUST NOT

- Do not treat every conversation message as relevant — only procedurally meaningful content gets extracted
- Do not execute, replay, or convert workflows — the skill captures only
- Do not silently merge when appending — present overlap and ask the user how to resolve
- Do not skip the review round — always present the workflow before saving
- Do not extract from external sources — only the current Claude Code conversation context

## PREFER

- Use QA-CoT internally for extraction — structured questions catch branching logic that flat extraction misses
- Cluster by procedural elements (intent, action, outcome) rather than raw chronological order
- Produce fewer phases with more detail over many shallow phases

</rules>

## Examples

**Example 1: New workflow from implementation session**
- **Input:** User invokes skill after a conversation where they researched an API, chose a library, implemented the integration, hit a CORS error, fixed it, and wrote tests
- **Output:** A workflow file at `.claude/workflows/integrating-payment-api.md` with 3 phases (Research, Implementation, Testing), the library decision captured as a branching point with alternatives considered, the CORS dead end with why it happened and how it was fixed, and a gotcha about CORS configuration

**Example 2: Appending to existing workflow**
- **Input:** User has an existing workflow `building-auth-system.md` covering research and implementation. New conversation covers writing tests and fixing edge cases. User says "append to existing"
- **Output:** Updated `building-auth-system.md` with a new "Phase 3: Testing" appended, existing header metadata preserved, status updated as appropriate

**Edge Case 1: Insufficient content**
- **Input:** User invokes skill after a conversation that was 90% asking questions about how React hooks work, with only one small code change
- **Output:** No file written. User told: "This conversation doesn't have enough actionable steps to extract a meaningful workflow. I found only 1 action (a small hook refactor) across the conversation. Try capturing after a session with more implementation work."

**Edge Case 2: Overlap on append**
- **Input:** Existing workflow has "Phase 2: Implementation" with steps about setting up routes. New conversation also covers route setup with a different approach
- **Output:** Skill detects overlap, asks user to choose: merge new details into existing steps, replace the overlapping section, or keep both versions labeled with session dates. File updated according to user's choice

## Output Format

ALWAYS structure workflow output as specified in [workflow-template.md](workflow-template.md).

## Edge Cases

**When conversation has fewer than 3 procedurally meaningful messages:** Inform the user the conversation lacks enough substance and do not write a file.

**When appending and new content overlaps existing steps:** Present specific overlapping sections and ask: merge, replace, or keep both. Never auto-merge.

**When conversation contains multiple unrelated workflows:** Ask the user which workflow to capture or whether to save as separate workflows.

**When `.claude/workflows/` directory does not exist:** Create it via `mkdir -p .claude/workflows`.

**When user says append but no existing workflows are found:** Inform user and switch to new mode.
