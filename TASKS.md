# Task Tracker

**Version**: v1.5.0
**Updated**: March 20, 2026

---

## Quick Reference

```
/task → /implement → /simplify → /test → /document → /ship → /release
```

| Status | Meaning |
|--------|---------|
| PLANNED | Ready for `/implement {ID}` |
| IN_PROGRESS | Being implemented |
| TESTING | Being tested via `/test` |
| APPROVED | Ready for `/ship` |
| READY_TO_SHIP | PR created, awaiting merge |
| SHIPPED | Merged and released |

**Task IDs:** Use simple numbers (1, 2, 3) to reference tasks. Example: `/implement 1`

---

## In Progress

| ID | Task | Priority | Started | Task Doc |
|----|------|----------|---------|----------|
| - | - | - | - | - |

---

## Planned

Tasks ready for `/implement {ID}`. Created via `/task`.

| ID | Task | Priority | Type | Task Doc | Created |
|----|------|----------|------|----------|---------|
| - | - | - | - | - | - |

---

## Testing

Tasks being tested via `/test`. Returns to `/implement` if FAIL.

| ID | Task | Type | Task Doc | Test Report | Status |
|----|------|------|----------|-------------|--------|
| - | - | - | - | - | - |

---

## Approved

Tested and approved. Ready for `/ship`.

| ID | Task | Type | Task Doc | Feature Doc | Test Report | Approved |
|----|------|------|----------|-------------|-------------|----------|
| - | - | - | - | - | - | - |

---

## Ready to Ship

PRs created via `/ship`. **Items stay here until `/release`** (even after merge).

| ID | Task | Type | Branch | PR | Merged | Task Doc |
|----|------|------|--------|-----|--------|----------|
| - | - | - | - | - | - | - |

---

## Shipped

| ID | Task | Type | PR | Release | Shipped |
|----|------|------|----|---------|---------|
| - | - | - | - | - | - |

---

## Backlog

### Features

| Feature | Priority | Notes |
|---------|----------|-------|
| - | - | - |

### Technical Debt

| Task | Priority | Notes |
|------|----------|-------|
| - | - | - |

---

## Known Issues

| Issue | Severity | Doc |
|-------|----------|-----|
| - | - | - |

---

## Skills Reference

### Workflow Skills

| Skill | Model | Purpose |
|-------|-------|---------|
| `/task` | sonnet | Create task documents with BDD acceptance criteria |
| `/implement` | sonnet | Implement tasks |
| `/simplify` | sonnet | Quality gate — coding standards + deviation check |
| `/test` | haiku | E2E testing (Playwright) |
| `/document` | haiku | Update docs |
| `/ship` | haiku | Create PRs |
| `/release` | haiku | Versioned releases |

### Specialized Skills

| Skill | Purpose |
|-------|---------|
| `/react-best-practices` | React/Next.js optimization |
| `/postgres-best-practices` | Database queries, RLS, schema |
