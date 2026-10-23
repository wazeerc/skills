# skills

Agent skills by [wazeerc](https://www.skills.sh/wazeerc) — browse and install them with the [`skills` CLI](https://www.skills.sh/docs/cli).

## Installation

```bash
npx skills add wazeerc/skills
```

Add `--skill create-better-skill`, `--skill chrome-validation`, `--skill jest-coverage`, or `--skill cleaner-commits` to install just one.

## create-better-skill

Simply a skill that helps you *create better skills*.

```
/create-better-skill I keep doing the same release-notes process by hand, let's capture it as a skill
```

Or just describe the workflow you've been repeating and the skill picks it up:

```
"Turn what we just did into a reusable skill"
```

It will interview you, generate the SKILL, review its weak points with you, and validate the final draft.

## chrome-validation

Validate a feature in a real Chrome browser with Chrome DevTools MCP: exercise the acceptance criteria as a user, check console and network, then report PASS, FAIL, or BLOCKED with evidence.

```
/chrome-validation validate sign-up at localhost:3000 — I can register with email + password and land on the dashboard
```

Requires the [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp/) server. If it isn't connected, the skill stops with setup instructions.

## jest-coverage

Check whether a React branch has enough Jest coverage to support a PR. It runs your existing coverage command, reads `coverage/lcov.info`, and compares project-wide and changed-file coverage against your Jest config or an 80% fallback.

```
/jest-coverage check coverage against main
```

Reports PASS, FAIL, or BLOCKED with the command, targets, per-file results, and next steps. It won't write tests or change thresholds.

## cleaner-commits

Use validated [Conventional Commits](https://www.conventionalcommits.org/) when writing commit messages.
