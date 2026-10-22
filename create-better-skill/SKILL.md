---
name: create-better-skill
description: "Generates an Agent Skill through a short interview, an early draft, and a weak-point review. Use when the user asks to create or write a skill or SKILL.md, or to turn this conversation's workflow into a reusable skill. Not for editing an existing skill, instructions files, prompt files, custom agents, hooks, or MCP servers."
---

# Create Better Skill

Goal: the best possible SKILL.md for the user's situation. They arrive with half an idea; pull the shape out a little at a time.

Three habits:

1. **One theme at a time.** Ask one theme per turn; batch only tightly related questions.
2. **Draft early.** Write a rough SKILL.md once the outcome and triggers are clear, then iterate on it together.
3. **Explain the why.** One line per recommendation so the user learns to make the next call themselves.

## Step 0 — Question tool

Use the host's native question tool for anything multiple-choice — OpenCode's `question`, VS Code's `askQuestions`, Claude Code's `AskUserQuestion`, or any equivalent. Give 2–5 options, recommended option first, one line per description. If the tool takes one question per call, ask one; if it batches, keep the batch within one theme. Open-ended questions ("describe the workflow") are fine as plain text. Plain text is the last resort, only when no question tool exists — then use lettered options (A / B / C).

Note the host agent you are running in: it decides the default save path and supported loading fields. If the user names a different target agent, use the save and loading rules in Step 4.

## Step 1 — Use the conversation if it already contains the workflow

Review the conversation before asking anything. If the user followed a multi-step workflow, checklist, or review process, extract the steps as they were actually done, the decision points, their quality bar, the tools/files involved, and when to use or not use the skill.

Propose it in one message — "You did X → Y → Z with a check at each step. Capture that as a skill?" — and once confirmed, go to Step 2 to confirm the primitive, then Step 3 to confirm the outcome, triggers, and name. Don't skip those checks.

Otherwise, go to Step 2.

## Step 2 — Confirm a skill is the right primitive

Check the table below; ask only if genuinely ambiguous.

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
| 1 | Outcome | What the skill produces. One sentence with a concrete verb, plus one real input and the output you'd accept. |
| 2 | Trigger | At least 3 phrases the user would actually type, in their vocabulary, and what the skill must not handle. |
| 3 | Shape | Checklist, multi-step workflow, reference-heavy, or script-driven? |
| 4 | Inputs & outputs | What goes in (files, URLs, arguments) and what "done" looks like. |
| 5 | Scope & loading | Project or personal; which agent(s) must load it; auto-load or `/name` only; which steps need confirmation before acting. |
| 6 | Assets | Scripts, templates, or examples to ship inside the skill. |

Rules:

- Ask for the new skill name before drafting. Normalize it to lowercase hyphens and confirm before creating the folder.
- Ask about loading and confirmation explicitly; don't assume.
- After themes 1–2, propose 2–3 names, recommend one, and confirm before creating the folder. Name rules are in Step 4.
- Mirror the answer back before moving to the next theme.
- Draft once themes 1–2 are answered; keep interviewing against the draft. If a theme stays unclear, mark it as an assumption and resolve it in review.

## Step 4 — Draft immediately

Once themes 1–2 are answered, write the SKILL.md. List assumptions in the chat, not in the file; turn accepted assumptions into plain instructions and leave no comments or TODOs before handoff.

Layout when bundled resources are needed:

```
<skill-name>/
├── SKILL.md          # name + description frontmatter, then the workflow
├── references/       # deep docs, loaded only when needed
├── scripts/          # deterministic code the agent runs
└── assets/           # templates/files copied into outputs
```

- `name` must match the folder name. Use 1–64 characters matching `^[a-z0-9]+(-[a-z0-9]+)*$`: lowercase letters, digits, and single hyphens; no leading or trailing hyphen and no `--`.
- `description` is the discovery surface: 1–1024 characters, always quoted. Use "what it does + Use when… + Not for…" with real trigger phrases; aim for 250–400 characters.
- Keep the body under ~500 lines; move longer content to `references/` with a "read this when..." pointer.
- Write it simple and direct: short imperative instructions, no filler or repeated context.
- Do not add `shell` or `bash` to `allowed-tools` unless the user explicitly accepts the risk of pre-approved terminal commands.
- Add an explicit confirmation before any step that changes files, deploys, sends messages, or spends money.
- Put attribution in frontmatter metadata:

  ```yaml
  metadata:
    generated-by: "create-better-skill (https://www.skills.sh/wazeerc/skills/create-better-skill)"
  ```

  Omit it if the user opts out.

### Save location

Use the scope and target host from Step 3. If the user gave a path, use it.

| Host | Project | Personal |
| --- | --- | --- |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Copilot / VS Code | `.github/skills/` or `.claude/skills/` | `~/.copilot/skills/` or `~/.claude/skills/` |
| Copilot CLI | `.github/skills/` or `.claude/skills/` | `~/.copilot/skills/` |
| OpenCode | `.opencode/skills/` or `.claude/skills/` | `~/.config/opencode/skills/` or `~/.claude/skills/` |

- In a git repo, default to project scope at the repo root. If there is no repo, ask whether to use a personal folder or a specific project; recommend personal for a reusable workflow.
- Use personal scope when the user asks for global/all projects.
- For a cross-agent project skill, use `.claude/skills/`: Claude Code, Copilot/VS Code, Copilot CLI, and OpenCode all read it. For a cross-agent personal skill there is no single path all hosts read; use the host table or symlink into more than one personal folder. `.agents/skills/` works for Copilot/VS Code and OpenCode, not Claude Code.
- Before writing, check the target and other supported locations for the same name. If it exists, confirm overwrite or choose another name.
- Show the user the path and quoted description. Don't install or publish unless asked. Then run Step 5.

### Loading

Use the choice from Step 3.

- Claude Code, Copilot, and VS Code: use `disable-model-invocation: true` for manual-only, or `user-invocable: false` for auto-load only.
- OpenCode: put `opencode/autoinvoke: "false"` under `metadata` to hide it from automatic selection, and `opencode/slash: "true"` to expose `/name`. Quote both values:

  ```yaml
  metadata:
    opencode/autoinvoke: "false"
    opencode/slash: "true"
  ```

Recommend manual-only for workflows that change files, deploy, or spend money; auto-load for reference or guidance. These are host-specific fields: for several hosts, include the fields each host understands and test in each. `skills-ref` will flag them as non-spec. Validate the choice in Step 7.

## Step 5 — Grill the weak parts

Re-read the draft and find where a fresh agent would have to guess:

- vague verbs ("handle the data", "process the request") — what literally happens?
- missing error paths — what should happen when input is wrong or a step fails?
- unstated quality bar — how does the agent know the output is good?
- a description without real trigger phrases or a "Not for" boundary
- steps that assume a tool, file, or credential that may not exist
- destructive or irreversible steps without a confirmation point

Ask about the 2–3 worst specifically, quoting the offending line. Apply the answers, then run at most two review rounds. After that, list what is still assumed and ask the user to accept or correct it.

## Step 6 — Validate

If `skills-ref` is installed and the skill uses only spec fields, run `skills-ref validate <skill-dir>`. It checks frontmatter only and rejects host-specific fields such as `disable-model-invocation`, so skip it for host-specific skills or expect those flags.

- [ ] `name` is 1–64 characters, matches `^[a-z0-9]+(-[a-z0-9]+)*$`, and matches the folder name, with no `--` or leading/trailing hyphen
- [ ] Frontmatter is valid YAML: no tabs, colons inside values quoted
- [ ] `description` is 1–1024 characters, quoted, with real trigger phrases and a "Not for" boundary
- [ ] Loading mode matches the user's choice and uses the target host's fields
- [ ] Every reference file says *when* to read it
- [ ] Markdown links resolve; bundled paths are relative and use `/`; no absolute paths
- [ ] No unresolved assumption comments, TODOs, or placeholder text
- [ ] No secrets or tokens
- [ ] Instructions explain why, not just MUST/NEVER
- [ ] Simple and direct: imperative sentences, no filler
- [ ] Body < ~500 lines; heavy content lives in references/scripts

Fix anything that fails and confirm each fix in one line.

## Step 7 — Hand off

1. **What it does** — two sentences.
2. **Path and loading** — where it was created and whether it auto-loads or only runs as `/name`.
3. **Example prompts** — three realistic positive prompts and two negative prompts that should not trigger the skill.
4. **Test** — reload skills (`/reload-skills` in Claude Code, `/skills reload` in Copilot CLI, restart or reload in OpenCode and VS Code), start a fresh session, and run one positive prompt, one `/name` prompt, and one negative prompt. A same-session run only tests memory, not discovery. For a manual-only skill, confirm the positive prompt does not auto-load it. Ask whether to run the first positive prompt for real.
5. **Next steps** — the exact path and how to invoke it.