# E. 운영 템플릿

이 부록은 개인 repo에 바로 옮겨 쓸 수 있는 템플릿 모음이다. 그대로 복사하기보다, 실제 명령과 경로를 채운 뒤 100~200줄 안으로 유지하는 것을 권장한다.

## AGENTS.md 템플릿

```markdown
# AGENTS.md

## Project Snapshot

- Runtime:
- Package manager:
- Main app paths:
- Main test paths:
- Generated files:

## Common Commands

- Install:
- Typecheck:
- Unit test:
- Focused test:
- Build:
- Lint:

## Editing Rules

- Prefer existing local patterns over new abstractions.
- Do not edit generated files:
- Do not change public APIs unless the task explicitly asks for it.
- Keep unrelated refactors out of task changes.

## Command Output Policy

- For unknown or potentially large output, start with `head -c 4000`.
- For test failures and recent logs, start with `tail -c 12000`.
- Use `rg` before opening large files.
- Do not read lockfiles, build artifacts, or generated files in full unless necessary.

## Verification Policy

- Required checks before final response:
- If a check cannot run, report why and what was verified instead.
- If tests fail, summarize the first actionable failure and avoid pasting full logs.

## Final Response Policy

Include:
- Changed files
- Checks run
- Remaining risk

Do not include:
- Long command output
- Full diffs unless requested
- Repeated background already known from the task
```

## Bugfix 브리프

```markdown
## Goal

Fix:

## Expected / Actual

- Expected:
- Actual:

## Reproduction

1.
2.
3.

## Evidence

- Error message:
- Relevant log excerpt:
- Suspected files:

## Scope

- In scope:
- Out of scope:

## Constraints

- API compatibility:
- UX compatibility:
- Performance/security:

## Verification

- Focused test:
- Manual check:
- Regression check:
```

## Feature 브리프

```markdown
## Goal

Implement:

## User Flow

1.
2.
3.

## Scope

- Include:
- Exclude:

## Existing Patterns

- Similar component:
- Similar test:
- Similar API:

## Constraints

- API:
- Design:
- Performance:
- Accessibility:

## Verification

- Unit:
- Integration:
- Manual:
```

## Review 브리프

```markdown
## Review Target

- Branch/PR:
- Diff range:

## Review Priorities

1. Correctness
2. Security
3. Regression risk
4. Missing tests

## Known Context

- Intended behavior:
- Non-goals:
- Risky areas:

## Output Format

- Findings first
- File/line references
- Severity ordering
- Short summary only after findings
```

## 캐시 친화 프롬프트 골격

```markdown
## Stable Context

### Agent Role

You are a coding agent working in this repository. Use the repository rules and verify changes before reporting completion.

### Repository Rules

- Package manager:
- Test command:
- Generated files:
- Output policy:

### Reusable Rubric

- Find the smallest relevant context before editing.
- Prefer targeted search over full-file reads.
- Keep dynamic tool output out of stable context.
- Report changed files, checks, and residual risk.

## Dynamic Task Context

### Current Task

### Scope

### Evidence

### Latest Tool Output Summary

### User Additions
```

## 주간 Token Review 템플릿

```markdown
## Weekly Agent Token Review

Period:
Repositories:
Tasks sampled:

### Expensive Tasks

| Task | Cause | Avoidable | Policy Change |
| --- | --- | --- | --- |
| | | | |

### Cache Health

- Best cache read ratio:
- Worst cache read ratio:
- Repeated cache create cause:

### Tool Output

- Largest output:
- Was it needed:
- New cap rule:

### Action Items

- AGENTS.md:
- Brief template:
- Test/log command:
- Dashboard:
```

## 출처

- [Austin1serb/agents-md](https://github.com/Austin1serb/agents-md)
- [Anthropic: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic Claude Docs: Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)
