---
name: create-better-skill
description: "**GUIDED SKILL CREATION** — build a new SKILL.md with the user one small step at a time: interview, draft early, grill the weak parts, polish, hand off. USE WHEN: the user says create a skill, make a skill for X, turn this workflow/checklist into a skill, package this into a reusable skill, improve/rewrite/review my skill or when the conversation has just followed a repeatable workflow worth capturing. DO NOT USE FOR: answering one-off coding questions (no skill needed); creating .instructions.md, .prompt.md, hooks, or custom agents — this skill helps pick the right primitive but does not create those. Adapts to the host agent: uses the ask-questions or AskUserQuestion tool when one is available, otherwise asks in plain text."
---

# Create Better Skill

You are walking beside the user. The goal is the best possible SKILL.md for *their* situation.
Users come to skill creation with half an idea; your job is to pull the shape out of them a little at a time.

Three habits make this work:

1. **Small batches.** Two or three related questions per turn. Each question comes with
   concrete options and your recommendations.
2. **Draft early.** Once you understand the outcome, write a rough SKILL.md and iterate on it together.
3. **Explain the why.** When you recommend a choice, say why in one line. The user learns to make the
   next decision themselves, which is the point of walking beside them.

## Step 0 — Detect how you can ask

Check your available tools before asking anything.

- If a structured question tool exists (VS Code's `ask-questions`, Claude Code's `AskUserQuestion`,
  an equivalent native question affordance), use it for anything multiple-choice: give 2–5 options,
  mark your recommended one first and keep descriptions to one line each.
- Open-ended questions (describe the workflow, what does "done" look like) are better as plain text
  anyway — ask those regardless of tooling.
- Never invent a question tool. If none is available, ask in plain text with lettered options
  (A / B / C) so answering is still one keystroke.

Also note the host agent you are running in — it decides where the finished skill should be created and which frontmatter fields are supported. The user may also tell you the target agent; trust that.

## Step 1 — Start from where the user already is

Before asking anything, review the conversation. If the user has been following a multi-step workflow, checklist, debugging method, or review process — even loosely — you already have the skill. Extract:

- the step-by-step process as they actually did it
- the decision points where they branched or corrected you
- their quality bar
- the tools and files involved

Present it as a proposal: "You've been doing X → Y → Z with a check at each step. Want me to capture that as a skill?"
Confirm the outline in one short message, then move to drafting.

If the conversation contains no workflow, go to Step 2.

## Step 2 — Make sure a skill is the right primitive

Skills are one of several customization types. Spending the interview on the wrong one wastes the
user's time, so spend 30 seconds checking. Ask only if genuinely ambiguous.

| The user wants... | Better primitive |
| --- | --- |
| Something applied to *most* work in a project, always on | Instructions file (`.instructions.md` / `AGENTS.md`) |
| A single focused task with inputs, run via a slash command | Prompt (`.prompt.md`) |
| A subagent with context isolation or per-stage tool restrictions | Custom agent (`.agent.md`) |
| Deterministic enforcement at tool lifecycle events (block, format, gate) | Hook (`.json`) |
| Integration with an external API/system | MCP server |
| An on-demand workflow, especially with bundled scripts/templates | **Skill — you are here** |

If another primitive fits better, say so plainly, tell the user why and offer to help with that instead. Do not force their request into a skill.

## Step 3 — The guided interview

Work through these themes one turn at a time. Skip any theme the user already answered and skip
anything they tell you to decide yourself (then tell them what you decided and why).

| # | Theme | What you need |
| --- | --- | --- |
| 1 | Outcome | What the skill produces or makes possible. One sentence, concrete verb. |
| 2 | Trigger | The phrases a user would actually type that should activate it — at least 3, in the user's own vocabulary. |
| 3 | Shape | Checklist, multi-step workflow, reference-heavy knowledge, or script-driven? |
| 4 | Inputs & outputs | What goes in (files, URLs, arguments?) and what "done" looks like (file written? report? code changed?). |
| 5 | Scope | Where it lives (this project / personal across projects) and which agent(s) it must work in. |
| 6 | Assets | Any scripts, templates, or example files that should ship inside the skill folder. |

Interview rules:

- **One theme per turn.** Don't ask about packaging while the outcome is still fuzzy.
- **Offer options with a recommendation.** "Shape: (A) 5-step workflow with a checklist at the end —
  I'd start here, (B) compact checklist, (C) reference doc the agent consults. Which fits?"
- **Mirror back.** Before moving to the next theme, paraphrase the user's answer and ask whether you understood it correctly.
- **Watch for fatigue.** After two rounds of questions, offer: "I can draft now and assume X and Y —
  we'll fix anything wrong in the review."

## Step 4 — Draft immediately

Once themes 1–3 are answered, write the SKILL.md. Do not wait for completeness — fill gaps with clearly marked assumptions (`<!-- assumption: ... -->`) and list them for the user.

Use this layout when the generated skill needs bundled resources:

```
<skill-name>/
├── SKILL.md          # name + description frontmatter, then the workflow
├── references/       # optional: deep docs loaded only when needed
├── scripts/          # optional: deterministic code the agent runs
└── assets/           # optional: templates/files copied into outputs
```

- `name` must match the folder name exactly (lowercase, hyphens) — mismatch fails silently.
- `description` is the discovery surface: it carries everything about *when* to trigger. Write it
  from the trigger phrases collected in Step 3, keep it slightly pushy and quote it if it contains
  colons.
- Body under ~500 lines; push anything longer into `references/` with a "read this when..." pointer.

Save it, then show the user the path, the description and a 3-line summary of the workflow.
Create it in the scope selected during the interview: the current project's skill directory for project scope, or the host agent's global skill directory for global scope.
Do not install or publish it unless the user explicitly asks.
Run the self-review in Step 5, then ask the user the anchored questions it produces.

## Step 5 — Grill the weak parts

Read your own draft with fresh eyes and find the places a fresh agent would guess. Typically:

- vague verbs ("handle the data", "process the request") — what literally happens?
- missing error paths — what should the agent do when the input is wrong or a step fails?
- unstated quality bar — how does the agent know the output is good?
- a description that lists features but contains none of the user's trigger phrases
- steps that assume a tool, file, or credential that may not exist

Pick the 2–3 worst and ask about those *specifically*, quoting the offending line.
Anchored questions are cheap to answer; "anything else?" is not. Wait for the user's answers before finalizing the draft.
Apply the answers and any requested changes, then return to Step 5 if the changes introduce new uncertainty.
Proceed to Step 6 only when the user has no further corrections or has explicitly accepted the remaining assumptions.

## Step 6 — Validate

Run this checklist against the final draft:

- [ ] Folder name == frontmatter `name`
- [ ] Frontmatter is valid YAML: no tabs, colons inside values quoted, apostrophes not breaking scalars
- [ ] `description` contains the real trigger phrases and a do-not-use boundary
- [ ] Every reference file is mentioned with *when* to read it
- [ ] Instructions explain why, not just a wall of MUST/NEVER
- [ ] No secrets, tokens, or absolute paths to the user's machine
- [ ] Body < ~500 lines; heavy content lives in references/scripts

Fix anything that fails, then confirm the fixes in one line each.

## Step 7 — Hand off

Wrap up with four short things:

1. **What the skill does** — two sentences, plain language.
2. **Example prompts to try it** — three realistic sentences the user could type right now. These double as trigger tests: if the skill doesn't fire on them, the description needs work.
3. **A suggested test** — offer to run the first example prompt for real, or to run a fuller evaluate/iterate loop if the user wants more rigor.
4. **Next steps** — state the exact project or global path where the skill was created and how the user can invoke it there.
