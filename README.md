# skills

Agent skills by [wazeerc](https://www.skills.sh/wazeerc) — browse and install them with the [`skills` CLI](https://www.skills.sh/docs/cli).

## Installation

```bash
npx skills add wazeerc/skills
```

Install one skill with `--skill <name>`:

| Skill | Purpose | Install | Example |
|---|---|---|---|
| [`create-better-skill`](create-better-skill/SKILL.md) | Interview, draft, review, and validate reusable agent skills. | `npx skills add wazeerc/skills --skill create-better-skill` | `/create-better-skill capture my release-notes workflow as a reusable skill` |
| [`chrome-validation`](chrome-validation/SKILL.md) | Validate a running feature in Chrome with user flows, console and network checks, and evidence. | `npx skills add wazeerc/skills --skill chrome-validation` | `/chrome-validation validate sign-up at localhost:3000` |
| [`jest-coverage`](jest-coverage/SKILL.md) | Compare project and changed-file Jest coverage against configured or fallback thresholds. | `npx skills add wazeerc/skills --skill jest-coverage` | `/jest-coverage check coverage for Feature X` |
| [`cleaner-commits`](cleaner-commits/SKILL.md) | Group changes into focused commits with validated Conventional Commit messages. | `npx skills add wazeerc/skills --skill cleaner-commits` | `/cleaner-commits prepare these changes for commit` |
| [`bem-css`](bem-css/SKILL.md) | Implement and review CSS using BEM blocks, elements, modifiers, mixes, and low-specificity selectors. | `npx skills add wazeerc/skills --skill bem-css` | `Write CSS for this card component using BEM` |
| [`web-accessibility`](web-accessibility/SKILL.md) | Build, review, fix, and test web interfaces against WCAG 2.2 AA. | `npx skills add wazeerc/skills --skill web-accessibility` | `/web-accessibility review the checkout flow against WCAG 2.2 AA` |

> [!NOTE]
> `chrome-validation` requires the [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp/) server.
