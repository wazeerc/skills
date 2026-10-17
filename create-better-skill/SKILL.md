---
name: create-better-skill
description: Create or improve a SKILL.md with the user through a short guided interview, early drafting, and a weak-point review.
---

# Create Better Skill

Goal: the best possible SKILL.md for the user's situation. They arrive with half an idea; pull the shape out a little at a time.

Three habits:

1. **Small batches.** 2–3 related questions per turn, each with concrete options and your recommendation.
2. **Draft early.** Write a rough SKILL.md as soon as the outcome is clear and iterate on it together.
3. **Explain the why.** One line per recommendation so the user learns to make the next call themselves.

## Step 0 — Question tool

Use the host's native question tool for anything multiple-choice — OpenCode's `question`, VS Code's `askQuestions`, Claude Code's `AskUserQuestion`, or any equivalent. Give 2–5 options, recommended option first, one line per description. Open-ended questions ("describe the workflow") are fine as plain text. Plain text is the last resort, only when no question tool exists — then use lettered options (A / B / C).

Note the host agent you are running in: it decides where the finished skill goes and which frontmatter fields are supported. Trust the user if they name a different target agent.

## Step 1 — Use the conversation if it already contains the workflow

Review the conversation before asking anything. If the user followed a multi-step workflow, checklist, or review process, extract the steps as they were actually done, the decision points, their quality bar, and the tools/files involved.

Propose it in one message — "You did X → Y → Z with a check at each step. Capture that as a skill?" — and once confirmed, go straight to drafting.

Otherwise, go to Step 2.

## Step 2 — Confirm a skill is the right primitive

Spend 30 seconds checking; ask only if genuinely ambiguous.

| The user wants... | Better primitive |
| --- | --- |
| Always-on guidance for most project work | Instructions (`.instructions.md` / `AGENTS.md`) |
| One focused task with inputs, run by slash command | Prompt (`.prompt.md`) |
| Context isolation or per-stage tool limits | Custom agent (`.agent.md`) |
| Deterministic enforcement at tool events | Hook (`.json`) |
| Integration with an external API/system | MCP server |
| An on-demand workflow, possibly with scripts/templates | **Skill — you are here** |

If another primitive fits better, say so and offer that instead. Don't force the request into a skill.

## Step 3 — Interview

One theme per turn. Skip themes the user already answered or delegated to you — tell them what you decided and why.

| # | Theme | What you need |
| --- | --- | --- |
| 1 | Outcome | What the skill produces. One sentence with a concrete verb. |
| 2 | Trigger | At least 3 phrases the user would actually type, in their vocabulary. |
| 3 | Shape | Checklist, multi-step workflow, reference-heavy, or script-driven? |
| 4 | Inputs & outputs | What goes in (files, URLs, arguments) and what "done" looks like. |
| 5 | Scope | Project-level by default; global only if the user asks. Which agent(s) it must work in. |
| 6 | Assets | Scripts, templates, or examples to ship inside the skill. |

Rules:

- Offer options with a recommendation through the question tool.
- Mirror the answer back before moving to the next theme.
- After two rounds, offer to draft with assumptions and fix anything wrong in review.

## Step 4 — Draft immediately

Once themes 1–3 are answered, write the SKILL.md. Fill gaps with `<!-- assumption: ... -->` and list them for the user.

Layout when bundled resources are needed:

```
<skill-name>/
├── SKILL.md          # name + description frontmatter, then the workflow
├── references/       # deep docs, loaded only when needed
├── scripts/          # deterministic code the agent runs
└── assets/           # templates/files copied into outputs
```

- `name` must match the folder name (lowercase, hyphens).
- `description` is the discovery surface: real trigger phrases plus a do-not-use boundary. Quote it if it contains colons.
- Keep the body under ~500 lines; move longer content to `references/` with a "read this when..." pointer.
- Write it simple and direct: short imperative instructions, no filler or repeated context.

Save it to the project-level skills directory your own host uses — decide this yourself, don't ask the user (e.g. `.github/skills` for Copilot, `.opencode/skills` for OpenCode, `.agents/skills` for cross-agent). Go global only if the user asks. Then show the user the path, the description, and a 3-line workflow summary. Don't install or publish unless asked. Run Step 5.

## Step 5 — Grill the weak parts

Re-read the draft and find where a fresh agent would have to guess:

- vague verbs ("handle the data", "process the request") — what literally happens?
- missing error paths — what should happen when input is wrong or a step fails?
- unstated quality bar — how does the agent know the output is good?
- a description without real trigger phrases
- steps that assume a tool, file, or credential that may not exist

Ask about the 2–3 worst specifically, quoting the offending line. Wait for answers, apply them, and repeat until the user has no corrections or explicitly accepts the remaining assumptions.

## Step 6 — Validate

- [ ] Folder name == frontmatter `name`
- [ ] Frontmatter is valid YAML: no tabs, colons inside values quoted
- [ ] `description` contains trigger phrases and a do-not-use boundary
- [ ] Every reference file says *when* to read it
- [ ] Instructions explain why, not just MUST/NEVER
- [ ] Simple and direct: imperative sentences, no filler
- [ ] No secrets, tokens, or absolute paths
- [ ] Body < ~500 lines; heavy content lives in references/scripts

Fix anything that fails and confirm each fix in one line.

## Step 7 — Hand off

1. **What it does** — two sentences.
2. **Example prompts** — three realistic prompts the user could type now; they double as trigger tests.
3. **Suggested test** — ask whether to run the first prompt for real.
4. **Next steps** — the exact path where the skill was created and how to invoke it.
