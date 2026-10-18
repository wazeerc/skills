---
name: cleaner-commits
description: "Create Conventional Commits when committing changes."
---

# Cleaner Commits

Use these rules whenever the host AI agent creates a Git commit, ensuring the commit message follows Conventional Commits.

## Rules

1. Classify the change using one allowed type:
	- `feat`: add user-visible functionality
	- `fix`: correct faulty behavior
	- `docs`: change documentation only
	- `style`: change formatting without behavior changes
	- `refactor`: restructure code without changing behavior
	- `perf`: improve performance
	- `test`: add or change tests
	- `build`: change build or dependency configuration
	- `ci`: change automation or pipelines
	- `chore`: maintenance work
	- `revert`: undo an earlier commit
2. Infer a scope from the affected package, module, or feature when it is unambiguous. Omit the scope when guessing would be less accurate.
3. Write an imperative subject in this format: `type(scope): description`. Keep it concise and specific. Use `type: description` when there is no useful scope. Add a short body only for breaking or multi-part changes that need context beyond the subject.
4. For breaking changes, append `!` after the type or scope and explain the break in the body or footer with `BREAKING CHANGE:`.
5. Validate the final message against the Conventional Commits rules. If the type, subject, scope, or breaking-change explanation is unclear, resolve it from the change context rather than inventing details.

## Quality check

- The type accurately describes the primary change.
- The subject is imperative, specific, and short.
- The scope is included only when confidently inferred.
- A body is present only when it adds necessary context.
- Breaking changes are clearly marked and explained.
