---
description: Generate visual code review on Miro board from GitHub PR or local changes
argument-hint: "<board-url> <pr|owner/repo#pr|PR-URL|--local|--branch name>"
allowed-tools: Bash(gh:*), Bash(git:*), mcp__miro__*, mcp__glean_default__*
---

# Visual Code Review

Generate a comprehensive visual code review on a Miro board from GitHub PRs, local changes, or branch comparisons. Optionally enriches the review with related documentation from enterprise knowledge bases.

## Arguments

- `board-url` (required): Miro board URL where the review will be created
- Source (one of):
  - `<pr-number>`: GitHub PR number in current repo (e.g., `42`)
  - `<owner/repo#number>`: PR from external repo (e.g., `facebook/react#12345`)
  - `<PR-URL>`: Full GitHub PR URL (e.g., `https://github.com/owner/repo/pull/42`)
  - `--local`: Review uncommitted local changes
  - `--branch <name>`: Compare branch against main

## Workflow

### 1. Parse Arguments

Extract the board URL and determine the source type:
- If argument is a number → GitHub PR in current repo
- If argument matches `owner/repo#number` → External GitHub PR
- If argument is a GitHub URL → Extract owner, repo, and PR number from URL
- If argument is `--local` → Local uncommitted changes
- If argument is `--branch <name>` → Branch comparison

### 2. Extract Changes

**For GitHub PR (current repo):**
```bash
gh pr view $PR_NUMBER --json title,body,author,files,additions,deletions
gh pr diff $PR_NUMBER
```

**For GitHub PR (external repo):**
```bash
gh pr view $PR_NUMBER --repo $OWNER/$REPO --json title,body,author,files,additions,deletions
gh pr diff $PR_NUMBER --repo $OWNER/$REPO
```

**For Local Changes:**
```bash
git status --porcelain
git diff HEAD
```

**For Branch Comparison:**
```bash
git log main..HEAD --oneline
git diff main...HEAD
```

### 3. Gather Context (Optional Enhancement)

**If Glean MCP is available**, search for related documentation:
- Design documents related to the changed components
- Past reviews of similar areas
- Architecture Decision Records (ADRs)
- Related incident reports or bug tickets

Use `mcp__glean_default__search` with queries based on:
- File paths being changed (e.g., "authentication service")
- PR title and description keywords
- Component/module names

This context helps identify:
- Relevant documentation to link in the review
- Subject matter experts to tag
- Historical context for the changes

**If Glean is NOT available**, proceed with code-only analysis.

### 4. Analyze Changes

For each changed file, determine:
- **Status**: Added, Modified, or Deleted
- **Risk Level**:
  - High: Security-related, authentication, database migrations, core business logic
  - Medium: API changes, configuration, shared utilities
  - Low: Tests, documentation, styling, localization
- **Change Summary**: Brief description of what changed
- **Review Notes**: Key points for human reviewer to verify
- **Related Docs**: Links to relevant documentation (from Glean if available)

### 5. Create Miro Board Content

Scale the review based on PR complexity. Create multiple elements as needed.

**Positioning Notes:**
- Tables do not support x/y positioning and are created at the board center (0,0)
- Create tables first, then position other elements around them
- Use a grid layout for large PRs: increment x by 2000 for horizontal spacing, y by 1500 for vertical spacing

#### Scaling Guidelines

| PR Size | Files | Recommended Elements |
|---------|-------|---------------------|
| Small | 1-5 | 1 summary doc, 1 table, 1 diagram |
| Medium | 6-15 | 1 summary doc, 1 table, 2-3 diagrams (by subsystem) |
| Large | 16+ | Multiple summary docs (by area), 1 table, multiple diagrams |

#### File Changes Table (created first, at board center)
Use `mcp__miro__table_create` with columns:

| Column | Type | Options |
|--------|------|---------|
| File | text | - |
| Change | text | Brief description |
| Status | select | Added (#00FF00), Modified (#FFA500), Deleted (#FF0000) |
| Risk | select | Low (#00FF00), Medium (#FFA500), High (#FF0000) |
| Notes | text | Review points |

Then use `mcp__miro__table_sync_rows` to populate rows with file data.

For very large PRs (30+ files), consider creating separate tables:
- High-risk files table (requires detailed review)
- Standard changes table (routine changes)

#### Summary Documents
Use `mcp__miro__doc_create` with markdown content.

**Main Summary (x=-2000, y=0):**
```markdown
# Code Review: [PR Title or Branch Name]

**Author:** [author]
**Files Changed:** [count]
**Lines:** +[additions] / -[deletions]

## Overview
[Brief description of what this change does]

## Key Changes
- [Bullet points of significant changes]

## Related Documentation
- [Links to relevant docs from Glean, if found]

## Review Checklist
- [ ] Logic correctness verified
- [ ] Edge cases handled
- [ ] Error handling appropriate
- [ ] No security concerns
- [ ] Tests adequate

## Questions for Author
- [Any clarifying questions based on the diff]
```

**Additional docs for large PRs** (positioned at y=1500, y=3000, etc.):
- Per-subsystem deep dives (e.g., "API Changes", "Database Migrations", "UI Updates")
- Security-focused analysis for high-risk changes
- Breaking changes documentation

#### Architecture Diagrams
Use `mcp__miro__diagram_create`. Create multiple diagrams for complex changes.

**Diagram type selection:**
- **Feature additions** → `flowchart` showing component interactions
- **Refactoring** → `uml_class` showing structural changes
- **API/integration changes** → `uml_sequence` showing interaction flow
- **Database changes** → `entity_relationship` showing schema modifications
- **Bug fixes** → `flowchart` showing fix location in flow

**For large PRs, create separate diagrams:**
- Overall architecture impact (x=2000, y=0)
- Per-subsystem flows (x=2000, y=1500; x=2000, y=3000; etc.)
- Data flow diagram if multiple services affected (x=4000, y=0)

Each diagram should show:
- Which components/modules are affected
- How data or control flows through the changed code
- Dependencies between changed files

## Example Usage

```
# PR in current repo
/miro-review:review https://miro.com/app/board/abc123= 42

# PR from external repo (short format)
/miro-review:review https://miro.com/app/board/abc123= facebook/react#12345

# PR from external repo (full URL)
/miro-review:review https://miro.com/app/board/abc123= https://github.com/vercel/next.js/pull/789

# Local uncommitted changes
/miro-review:review https://miro.com/app/board/abc123= --local

# Compare branch against main
/miro-review:review https://miro.com/app/board/abc123= --branch feature-login
```

## Output

After completion, provide:
1. Link to the Miro board
2. Summary of what was created
3. Highlight any high-risk files that need careful review
4. List of related documentation found (if Glean was used)
