---
name: jest-coverage
description: "Verify Jest coverage for React changes, analyze lcov.info, compare changed-file coverage with configured or 80% targets, and report PASS/FAIL/BLOCKED. Do not write tests or change thresholds."
---

# Jest Coverage

Determine whether the current React branch has enough Jest coverage to support a PR.
Treat coverage as an evidence-gathering task: do not modify source, tests, configuration, lockfiles, generated reports, or dependencies.

## 1. Confirm the project and inputs

1. Find the nearest `package.json` that owns the current workspace. Confirm that it is a React project by checking its dependencies or devDependencies for `react` or `react-dom`.
	- In a monorepo, identify every affected package from the changed paths. Assess each affected React package with its own config, scripts, lockfile, and coverage report. A package with no usable coverage command is `BLOCKED`; do not silently omit it.
2. Read the repository instructions that apply to testing, including `AGENTS.md`, `CONTRIBUTING.md`, and local agent instructions when present. Follow their commands and restrictions.
3. Inspect `git status --short` and the branch history. Include both committed and uncommitted work:
	- For committed branch changes, compare `HEAD` with the merge base of the current branch and its default branch.
	- Include staged and unstaged changes in the working tree.
	- If the default branch or merge base cannot be determined, ask for a base ref or report `BLOCKED`; do not guess.
4. Accept an explicit base ref or target percentage from the user when supplied. Otherwise infer both from the repository.
5. If there are no committed, staged, or unstaged source changes and the user did not provide an explicit base ref or file scope, stop with `BLOCKED`. State that this skill requires a change scope and show how to provide one.

If the repository is not a React project, or Jest and a usable coverage command are absent, stop with `BLOCKED` and explain the exact prerequisite that is missing.

## 2. Resolve the coverage target

Use the most specific applicable target, in this order:

1. An explicit target supplied by the user for this run.
2. Jest coverage thresholds in the repository's Jest configuration, including `jest.config.*`, `package.json`, or an equivalent config referenced by the test script. Respect separate global, file, and path thresholds when they exist.
3. A documented repository or CI coverage threshold, if it is clearly authoritative.
4. `80%` for statements, branches, functions, and lines when no target is configured.

Record the source of every target. Do not lower, rewrite, or temporarily bypass an existing threshold. If thresholds conflict, prefer the command's effective Jest configuration and call out the conflict in the report.

## 3. Choose and run the existing command

1. Inspect `package.json` scripts and lockfiles to determine the package manager. Prefer the repository's existing coverage script, such as `test:coverage`, `coverage`, or a test script that passes `--coverage`.
2. Use the matching package-manager invocation (`npm`, `pnpm`, `yarn`, or `bun`). Do not install packages or invent a new test command when a repository command exists.
3. If no coverage script exists but the local Jest binary is installed, run the package-manager equivalent of `jest --coverage` from the owning package and state it before running it. Preserve the project's normal test environment and configuration. If the local binary is unavailable, stop with `BLOCKED` and give the exact dependency or script setup needed; do not download a binary implicitly.
4. Run coverage once from the package root. Capture the exact command, exit code, summary output, and report paths.

Do not treat a failing test run as a coverage pass. If the command fails, abort the assessment with `BLOCKED` when coverage evidence is unusable, or `FAIL` when the failure itself demonstrates that the branch is not ready. Include the exact error and recovery steps. Do not retry with altered thresholds or flags unless the user explicitly requests that experiment.

## 4. Analyze the generated evidence

Use the command output and generated reports together. Prefer structured report data over manually estimating percentages.

1. Locate `coverage/lcov.info` or the path reported by Jest. If it is missing, stale, empty, or unreadable, report `BLOCKED` and explain how to produce it.
2. Read the LCOV records and collect, for each source file, line totals and covered lines (`LF` and `LH`). Also note function and branch totals when the record contains them (`FNF`/`FNH`, `BRF`/`BRH`).
3. Map changed source files to LCOV paths. Normalize repository-relative paths and common `file://` or workspace prefixes without changing the source tree.
4. For each changed source file with executable lines, calculate line coverage as $100 \times LH / LF$. Report files with no LCOV record, no executable lines, or source-map/path mismatches as `unmeasured`, not as 100%.
5. Use Jest's global summary for the project-wide statement, branch, function, and line percentages. If the text summary is unavailable, use `coverage/coverage-summary.json` or the configured JSON summary report. If the summary and LCOV disagree, report both and treat the effective Jest threshold result as authoritative.
6. Identify uncovered changed lines when the LCOV record provides line hits (`DA:<line>,0`). Focus the report on changed files and uncovered changed lines; do not dump the entire LCOV file.

When a changed file is deleted, renamed, generated, a fixture, or excluded by the coverage configuration, record that treatment explicitly. Do not demand coverage for a file that the effective configuration intentionally excludes, but do flag changed source files that are unexpectedly absent from the report.

## 5. Decide the result

Return `PASS` only when all available required checks meet their effective targets:

- the coverage command completed successfully;
- the project-wide Jest metrics meet their configured or fallback targets;
- every measured changed source file meets the applicable line target; and
- no changed source file required for the assessment is unmeasured.

Return `FAIL` when the command completed and the evidence shows a metric or changed file below its target. Return `BLOCKED` when the result cannot be trusted because setup, command selection, configuration, report generation, parsing, or change comparison failed.

Do not turn an unmeasured file into a pass, and do not infer coverage from test names or source inspection. Separate test failures from missing tooling and from insufficient percentages.

## 6. Report

Use this concise, evidence-first format:

```markdown
## Result

PASS | FAIL | BLOCKED

## Scope

- Project:
- Base ref:
- Changed source files:
- Includes committed changes: yes/no
- Includes working-tree changes: yes/no

## Target

- Source: explicit input / Jest config / repository CI docs / 80% fallback
- Statements:
- Branches:
- Functions:
- Lines:

## Coverage

| Scope | Statements | Branches | Functions | Lines | Result |
| --- | ---: | ---: | ---: | ---: | --- |
| Project | ... | ... | ... | ... | PASS/FAIL/BLOCKED |
| Changed files | ... | ... | ... | ... | PASS/FAIL/BLOCKED |

## Changed-file details

| File | Lines | Target | Uncovered changed lines | Result |
| --- | ---: | ---: | --- | --- |
| ... | ... | ... | ... | PASS/FAIL/UNMEASURED/EXCLUDED |

## Evidence

- Command:
- Test exit code:
- LCOV path:
- Relevant errors or warnings:

## Next steps

- For `FAIL`: name the files or metrics below target and the tests that need attention.
- For `BLOCKED`: give exact setup or recovery commands and say what must be rerun.
```

Keep the report reproducible. Include the command and paths, but omit secrets and unrelated test output. Never claim that coverage is sufficient when the report is stale, incomplete, or based only on a subset of the changed source files.
