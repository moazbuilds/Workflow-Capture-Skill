# Workflow Capture Skill for Claude Code

A Claude Code skill that extracts structured, reusable workflow documents from your conversation history. Instead of losing the knowledge buried in a long implementation session, this skill captures the phases, steps, decisions, dead ends, and reasoning into a markdown file you can reference or replay later.

## What It Does

When invoked, the skill reads your current Claude Code conversation and produces a structured workflow document containing:

- **Phases and steps** -the distinct stages of work and the specific actions taken in each
- **Decisions with alternatives** -what choices were made, what was considered, and why one option won
- **Dead ends** -what was tried and abandoned, why it failed, and what to avoid next time
- **Branching logic** -conditional paths captured as `if/else` structures, not flattened into linear lists
- **Artifacts and gotchas** -files produced and non-obvious pitfalls worth remembering

The output is saved to `.claude/workflows/` as a markdown file that serves as a reusable reference for similar future tasks.

## When to Use It

Use this skill when you have a **conversation you want to preserve** -typically an implementation session, debugging flow, or multi-step process where real decisions were made and problems were solved.

**Optimal context window: 40%-60% full.** The skill works best when the conversation has enough substance to extract meaningful phases and decisions, but isn't so long that critical details are summarized away. It will work with larger context windows too, but the sweet spot is that mid-range where the full detail is still available.

**Trigger phrases:**
- `capture workflow`
- `save workflow`
- `extract workflow`
- `record workflow`
- `document this workflow`
- `append workflow`

**Good candidates for capture:**
- You just finished implementing a feature and want to remember the approach
- You debugged a tricky issue and want to preserve the diagnostic steps
- You went through a multi-step setup process with decisions at each stage
- You want to build on a previous session's workflow (append mode)

**Not a good fit:**
- Conversations that are mostly Q&A with little implementation
- Sessions with fewer than 3 meaningful actions/decisions (the skill will tell you and stop)

## Installation

Copy the `.claude/skills/capturing-workflows/` directory into your project's `.claude/skills/` folder:

```
your-project/
  .claude/
    skills/
      capturing-workflows/
        SKILL.md
        workflow-template.md
```

## Usage

1. Work through an implementation session, debugging flow, or any multi-step process in Claude Code
2. When you're ready to capture, say **"capture workflow"** (or any trigger phrase)
3. Answer the three intake questions:
   - What's the goal of this workflow?
   - What made this approach work?
   - New workflow or appending to an existing one?
4. Review the generated workflow document
5. Request any changes, then confirm to save

The workflow is saved to `.claude/workflows/<name>.md`.

## Append Mode

If you're continuing work across multiple sessions, choose **append mode** during intake. The skill will:

1. Show you existing workflows in `.claude/workflows/`
2. Let you pick which one to extend
3. Detect overlapping content and ask how to handle it (merge, replace, or keep both)
4. Preserve existing metadata while adding new phases

## Output Format

Workflows follow a consistent template:

```markdown
# Workflow Title

> **Goal:** One-sentence description
> **Key Insight:** The critical technique that made it work
> **Captured:** Date
> **Status:** complete | in-progress

## Phases
### Phase 1: Name
#### Step 1.1: Name
- **How:** What to do
- **Why:** Reasoning
- **Decision:** Choice made and why
- **Output:** What this produces
- **Depends on:** Prerequisites

## Dead Ends
### What was tried
- **What happened:** Description
- **Why it failed:** Reason
- **Lesson:** What to avoid

## Gotchas
- Non-obvious pitfalls
```

## License

MIT
