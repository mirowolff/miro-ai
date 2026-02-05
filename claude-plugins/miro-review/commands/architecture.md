---
description: Architecture-focused review analyzing structural changes and design patterns
argument-hint: "<board-url> <pr|owner/repo#pr|PR-URL|--local|--branch name>"
allowed-tools: Bash(gh:*), Bash(git:*), mcp__miro__*, mcp__glean_default__*
---

# Architecture Review

Generate an architecture-focused visual code review on a Miro board. Analyzes structural changes, design patterns, dependencies, and architectural impact.

## Arguments

Same as `/miro-review:review`:
- `board-url` (required): Miro board URL
- Source: PR number, external PR, PR URL, `--local`, or `--branch <name>`

## Architecture Analysis Focus

### Structural Concerns

| Aspect | What to Analyze |
|--------|-----------------|
| **Dependencies** | New dependencies, circular deps, coupling changes |
| **Layering** | Layer violations, proper separation of concerns |
| **Patterns** | Design pattern usage, anti-patterns introduced |
| **Interfaces** | API contracts, breaking changes, versioning |
| **Modularity** | Component boundaries, cohesion, coupling |
| **Scalability** | Performance implications, bottlenecks |
| **Testability** | Test coverage, mockability, isolation |

### Impact Assessment

| Impact Level | Criteria |
|--------------|----------|
| **Breaking** | API changes, contract modifications, migration required |
| **Significant** | New patterns, architectural decisions, major refactoring |
| **Moderate** | Component changes, interface updates |
| **Minor** | Internal changes, no external impact |

## Workflow

### 1. Extract Changes
Get PR diff and identify structural changes:
- New files/modules/packages
- Moved or renamed components
- Interface/API changes
- Dependency modifications

### 2. Architecture Context Gathering

**If Glean MCP is available**, search for:
- Architecture Decision Records (ADRs)
- System design documents
- Component ownership and dependencies
- Technical debt tracking
- Previous architecture reviews

Use queries like:
- "ADR [component name]"
- "architecture [service name]"
- "design document [feature area]"

### 3. Structural Analysis

**Dependency Analysis:**
```bash
# Check for new dependencies (example for Node.js)
git diff HEAD -- package.json | grep "^\+"

# Analyze import changes
git diff HEAD -- "*.ts" "*.js" | grep -E "^[+-].*import"
```

**Module Structure:**
- Identify new modules/packages created
- Check for proper encapsulation
- Verify consistent naming conventions

**Interface Changes:**
- API signature modifications
- Type/schema changes
- Breaking vs. non-breaking changes

### 4. Create Architecture Review Artifacts

#### Component Impact Table
Use `mcp__miro__table_create`:

| Column | Type | Options |
|--------|------|---------|
| Component | text | Name/path |
| Change Type | select | New (#00FF00), Modified (#FFA500), Removed (#FF0000), Moved (#0088FF) |
| Impact | select | Breaking (#FF0000), Significant (#FFA500), Moderate (#FFFF00), Minor (#00FF00) |
| Dependencies | text | Affected components |
| Notes | text | Architecture concerns |

#### Architecture Summary Document
Position at x=-2000, y=0:

```markdown
# Architecture Review: [PR Title]

**Review Date:** [date]
**Impact Level:** [Breaking/Significant/Moderate/Minor]

## Summary
[Overview of architectural changes]

## Structural Changes

### New Components
- [List new modules, services, packages]

### Modified Components
- [Components with significant changes]

### Dependency Changes
- [New, removed, or updated dependencies]

## Design Pattern Analysis
- [Patterns introduced or modified]
- [Anti-patterns identified]

## Breaking Changes
- [API changes requiring consumer updates]
- [Migration requirements]

## Architecture Decision Points
- [Decisions that should be documented as ADRs]
- [Trade-offs made]

## Technical Debt
- [New debt introduced]
- [Debt addressed]

## Related ADRs
[Links from Glean if available]

## Recommendations
- [Architectural improvements]
- [Documentation needs]
- [Follow-up items]
```

#### Architecture Diagrams

**Component Diagram** (x=2000, y=0):
Use `mcp__miro__diagram_create` with `uml_class`:
- Show affected components and their relationships
- Highlight new vs. modified components
- Show dependency directions

**Before/After Comparison** (x=2000, y=1500):
For significant refactoring, create side-by-side:
- Previous structure
- New structure
- Changed relationships

**Data Flow Diagram** (x=4000, y=0):
Use `flowchart` to show:
- How data moves through changed components
- New data paths introduced
- Removed data flows

**Sequence Diagram** (x=4000, y=1500):
For API/interaction changes, use `uml_sequence`:
- Show updated interaction patterns
- Highlight changed message flows

## Example Usage

```
# Architecture review of PR
/miro-review:architecture https://miro.com/app/board/abc123= 42

# Architecture review of feature branch
/miro-review:architecture https://miro.com/app/board/abc123= --branch feature-new-api
```

## Output

After completion, provide:
1. Link to the Miro board
2. Overall architectural impact assessment
3. List of breaking changes (if any)
4. Components requiring documentation updates
5. Recommended ADRs to create
