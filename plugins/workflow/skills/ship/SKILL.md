---
name: ship
description: Deploy approved features to production. Creates PRs, runs pre-deployment checks, and updates TASKS.md. Use after /document completes. Supports task IDs for easier invocation.
---

# /ship - Deployment Agent

## Workflow

```
/ship {ID}
       ↓
1. Resolve task ID → find task document
2. Verify task is in "Approved" in TASKS.md
3. Check Automation field (manual | auto)
4. Move to "Ready to Ship" in TASKS.md
5. Run pre-deployment checks (build, typecheck, lint)
6. Check if branch already exists on origin (worktree flow)
7. Create Pull Request with [task-{ID}] reference
8. Update TASKS.md with PR link
       ↓
After merge → Update "Merged" column (stay in "Ready to Ship")
       ↓
/release → Moves to "Shipped" with version
```

**Items stay in "Ready to Ship" even after merge. Only `/release` moves items to "Shipped".**

---

## Auto Mode Behavior

When task document has `Automation: auto`:

```
≡ SIGNAL
STAGE: ship
STATUS: DONE
TASK: {ID}
SUMMARY: PR #{number} created and ready for review
PR: {full PR URL}
≡ END
```

The orchestrator receives this, displays pipeline complete summary, and exits. Automation does NOT auto-merge.

---

## Pre-Deployment Checklist

### 1. Verify Approval
- Task in "Approved" section of TASKS.md
- Feature doc exists
- Test report shows PASS

### 2. Review Changes
```bash
git status
git diff main...HEAD
```
- All intended files changed
- No unintended changes
- No sensitive files (`.env`, credentials)

### 3. Run Pre-Deployment Checks
```bash
pnpm build
pnpm typecheck
pnpm lint
```
All must pass before creating PR.

---

## Git Workflow

### Branch Strategy
```
feature/{ID}-{task-name}     - New features
fix/{ID}-{task-name}         - Bug fixes
enhance/{ID}-{task-name}     - Enhancements
```

### If Branch Already Exists on Origin (Worktree Flow)

Check first:
```bash
git ls-remote --heads origin feature/{ID}-{task-name}
```

If exists → do not checkout; go straight to PR creation:
```bash
gh pr create --base main --head feature/{ID}-{task-name} --title ...
```

### If Branch Doesn't Exist (Manual / Fallback Flow)

```bash
git checkout -b feature/{ID}-{task-name}
git add {files-changed-by-this-task}
git commit -m "[task-{ID}] feat: {description}

{Detailed description of changes}

Task: docs/task/{ID}-{task-name}.md
Test: docs/testing/{ID}-{task-name}.md

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
git push -u origin feature/{ID}-{task-name}
```

---

## Create Pull Request

```bash
gh pr create --title "[Task #{ID}] {PR Title}" --body "$(cat <<'EOF'
## Summary

{2-3 bullet points describing the changes}

## Task Documentation

- **Task ID:** #{ID}
- **Task Doc:** [docs/task/{ID}-{task-name}.md](link)
- **Test Report:** [docs/testing/{ID}-{task-name}.md](link)
- **Feature Doc:** [docs/features/{feature}.md](link)

## Changes

| File | Change |
|------|--------|
| `path/to/file` | Description |

## Commits

{List commits with [task-{ID}] prefix}

## Testing

- [x] E2E tests passed (see test report)
- [x] Manual testing completed
- [ ] Ready for code review

---

Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

---

## Update TASKS.md

**After moving to Ready to Ship:**
```
| ID | Task | Branch | PR | Merged | Task Doc |
|----|------|--------|----|--------|----------|
```

**After PR created:** add PR link.

**After PR merged:** update "Merged" column with date — item stays in "Ready to Ship" until `/release`.

**After merge, update task document:**
```markdown
> **Status:** MERGED
> **Merged:** {Date}
> **PR:** #{number}
```

---

## PR Checklist

| Category | Checks |
|----------|--------|
| Code Quality | Build passes, no TypeScript errors, lint passes, no console.log, no commented-out code |
| Documentation | Task doc complete, test report PASS, feature doc updated, user guide updated (if user-facing) |
| Security | No hardcoded secrets, no `.env` files, no exposed API keys, proper auth checks |
| Testing | E2E tests pass, manual testing done, edge cases handled |

---

## Summary Output

**Manual mode:** Output PR URL, branch name, pre-deployment check results, and next steps (review/merge PR, then `/release`).

**Auto mode:** Output SIGNAL and exit per the format above.
