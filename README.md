# Compound Engineering Template

> Drop into any project. Fill in `{placeholders}`. Works for any stack and any team size.

---

## The Loop

```
PLAN → WORK → REVIEW → COMPOUND → repeat
```

Every feature starts as a plan doc. Patterns compound over time. Drift is caught at review.

---

## Folder Structure

```
.cursor/rules/              ← enforcement (what AI must/must-not do in code)
CLAUDE.md                   ← reasoning (why decisions were made, system contracts)
AGENTS.md                   ← orchestration (who does what in the loop)
governance/
  decision-records/         ← ADRs (why each key decision was made)
  architecture-principles.md
  domain-boundaries.md
docs/
  plans/                    ← every feature starts here, before any code
  patterns/                 ← extracted reusable solutions
  solutions/                ← one-off solved problems for reference
  postmortems/              ← what went wrong and what changed
  runbooks/                 ← how to operate the system
services/                   ← backend services
apps/                       ← frontend applications
shared/
  domain/                   ← shared types and event schemas
  ui-components/
  utils/
infra/
  terraform/
  k8s/
  ci-cd/
```

---

## Separation of Concerns

| File                | Role           | Rule                                      |
|--------------------|----------------|-------------------------------------------|
| `.cursor/rules/`   | Enforcement    | Syntax, patterns, required boilerplate    |
| `CLAUDE.md`        | Reasoning      | Why, system contracts, anti-patterns      |
| `AGENTS.md`        | Orchestration  | Agent system prompts, review checklist    |
| `governance/`      | Governance     | ADRs, principles, ownership              |
| `docs/`            | Knowledge      | Plans, patterns, postmortems              |

> `.cursor/rules/` is the cop. `CLAUDE.md` is the architect. They never swap roles.

---

## Setup Steps

1. Copy this folder to your project root
2. Fill `{placeholders}` in `CLAUDE.md` (system identity, domain map, data stores, team)
3. Fill `{placeholders}` in `governance/domain-boundaries.md`
4. Customize `.cursor/rules/` with stack-specific patterns
5. First feature: create `docs/plans/FEAT-001-{slug}.md` using the template

---