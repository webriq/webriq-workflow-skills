---
name: release
description: Create versioned releases with consolidated changelogs. Gathers shipped items, generates CHANGELOG.md entry, creates git tag, and GitHub Release. Use after multiple /ship tasks are merged.
---

# /release - Release Management Agent

## Invocation

```bash
/release             # Auto-determine version from task docs (RECOMMENDED)
/release auto        # Same as above
/release patch       # Force patch increment (v1.1.21 → v1.1.22)
/release minor       # Force minor increment (v1.1.21 → v1.2.0)
/release major       # Force major increment (v1.1.21 → v2.0.0)
/release v1.1.22     # Explicit version
```

---

## Workflow

```
/release [version|auto]
       ↓
1. Get current version from git tags
2. Read "Ready to Ship" items from TASKS.md
3. Filter: only include items with Merged = ✅
4. Read each task document for Type & Version Impact
5. Auto-calculate version (if not explicit)
6. Categorize changes by type
7. Generate changelog entry
8. Update CHANGELOG.md
9. Commit changelog
10. Create git tag
11. Push to remote
12. Create GitHub Release
13. Move only merged items to "Shipped" with release version
        ↓
Output: release summary
```

---

## Pre-Release Checklist

- [ ] All PRs in "Ready to Ship" are merged
- [ ] Main branch up to date (`git pull`)
- [ ] No uncommitted changes (`git status`)

---

## Step 1: Get Current Version

```bash
git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0"
git tag --list --sort=-v:refname | head -5
```

---

## Step 2: Determine New Version

**Explicit:** `/release v1.1.22` → use directly.

**Semantic:** `/release patch|minor|major` → increment accordingly.

**Auto-calculate** (recommended): Read `Version Impact` from each task document.

**Priority order:**
1. `major` — any major item → major bump
2. `minor` — any minor item, no major → minor bump
3. `patch` — all patch items → patch bump

**Fallback if Version Impact missing:**

| Type | Impact |
|------|--------|
| `feature` | `minor` |
| `bugfix` | `patch` |
| `enhancement` | `patch` |
| `documentation` | `patch` |
| `chore` | `patch` |

---

## Step 3: Filter Merged Items

Parse TASKS.md "Ready to Ship" section:
- `✅` or date in Merged column → Include in release
- `No` or empty → Skip (not ready)

---

## Step 4: Categorize Changes

| Type | Changelog Section |
|------|-------------------|
| `feature` | New Features |
| `bugfix` | Bug Fixes |
| `enhancement` | Enhancements |
| `documentation` | Documentation |
| `chore` | Other Changes |

---

## Step 5: Generate Changelog Entry

```markdown
## [v1.1.22] - Month DD, YYYY

### New Features
- **Scope:** Description (#PR)

### Bug Fixes
- Description (#PR)

### Enhancements
- Description (#PR)

### Breaking Changes (if any)
- Description and migration path
```

**Good entries:** Start with verb, include scope, reference PR, be user-focused.
**Bad entries:** Internal refactors users don't see, overly technical descriptions, duplicate entries.

---

## Step 6: Update CHANGELOG.md

**Location:** `docs/changelogs/CHANGELOG.md`

Insert new version at the top (after header). If CHANGELOG.md doesn't exist, create with header and new entry.

---

## Step 7: Commit Changelog

```bash
git add docs/changelogs/CHANGELOG.md
git commit -m "chore(release): v{version}

Release v{version} with:
- {item 1}
- {item 2}

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

---

## Step 8: Create Git Tag

```bash
git tag -a v{version} -m "Release v{version}

New Features:
- {feature} (#{PR})

Bug Fixes:
- {fix} (#{PR})"
```

---

## Step 9: Push to Remote

```bash
git push origin main
git push origin v{version}
```

---

## Step 10: Create GitHub Release

```bash
gh release create v{version} \
  --title "v{version}" \
  --notes "## What's New

### New Features
- **Web:** {description} (#{PR})

### Bug Fixes
- {description} (#{PR})

---

**Full Changelog:** https://github.com/owner/repo/compare/v{previous}...v{version}"
```

---

## Step 11: Update TASKS.md

Move only merged items from "Ready to Ship" to "Shipped". Unmerged items stay in "Ready to Ship" for next release.

**Shipped section:**
```
| ID | Task | PR | Release | Shipped |
|----|------|-----|---------|---------|
```

---

## Edge Cases

**No merged items:**
- Verify PRs created via `/ship`, merged (Merged column shows ✅), and TASKS.md "Ready to Ship" has ✅ items
- List current unmerged items and stop

**Tag already exists:**
- Use a different version or delete existing tag: `git tag -d v{version}`

**Missing task Type field:**
- Default to "enhancement"
- Warn user to add types for better categorization

---

## Summary Output

```
Release v{version} created!

Changes: {N} New Features, {N} Bug Fixes, {N} Enhancements
Files updated: docs/changelogs/CHANGELOG.md, TASKS.md
Tag: v{version}
GitHub: {release URL}

Next release will be: v{patch} (patch) / v{minor} (minor) / v{major} (major)
```
