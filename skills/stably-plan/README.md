# stably-plan

Discover test coverage gaps and generate `test.fixme()` plan files with `stably plan`.

## Installation

```bash
npx skills add stablyai/agent-skills --skill stably-plan
```

## When to Use

- Planning test coverage for a new or existing project
- Discovering untested user flows and coverage gaps
- Creating a test automation roadmap with prioritized skeletons
- Generating `test.fixme()` files for team review before implementation
- Scoping test coverage to specific features or areas

## What It Does

`stably plan` autonomously:

1. **Explores** your codebase to map routes, components, and existing tests
2. **Identifies** coverage gaps — critical flows with no E2E tests
3. **Generates** `test.fixme()` skeleton files with priorities (`@p0`–`@p3`)
4. **Validates** the plan for duplicates, variant coverage, and structural correctness

The output is standard Playwright spec files with `test.fixme()` blocks — branch-aware, reviewable in PRs, and upgradeable to real tests via `stably create`.

## Quick Reference

| Command | Description |
|---------|-------------|
| `stably plan` | Full autonomous discovery across entire app |
| `stably plan "focus on checkout"` | Scoped plan for specific area |
| `stably plan "plan tests for this PR"` | Plan based on PR changes |
| `stably create "implement all @p0 tests"` | Implement planned tests |

## Lifecycle

```
stably plan  -->  review  -->  stably create  -->  stably test  -->  stably fix
```

## Related

- [stably-cli](../stably-cli) - Full CLI assistant (plan, create, run, fix, verify)
- [stably-sdk-rules](../stably-sdk-rules) - AI rules for writing tests with Stably SDK
- [stably-sdk-setup](../stably-sdk-setup) - SDK setup assistant
