---
description: Security-focused code review with vulnerability analysis
argument-hint: "<board-url> <pr|owner/repo#pr|PR-URL|--local|--branch name>"
allowed-tools: Bash(gh:*), Bash(git:*), mcp__miro__*, mcp__glean_default__*
---

# Security Code Review

Generate a security-focused visual code review on a Miro board. Analyzes code changes for potential vulnerabilities, security anti-patterns, and compliance concerns.

## Arguments

Same as `/miro-review:review`:
- `board-url` (required): Miro board URL
- Source: PR number, external PR, PR URL, `--local`, or `--branch <name>`

## Security Analysis Focus

### Vulnerability Categories

Analyze changes for these security concerns:

| Category | What to Look For |
|----------|-----------------|
| **Injection** | SQL injection, command injection, XSS, template injection |
| **Authentication** | Weak auth logic, hardcoded credentials, session handling |
| **Authorization** | Missing access controls, privilege escalation paths |
| **Data Exposure** | Sensitive data in logs, unencrypted storage, PII handling |
| **Cryptography** | Weak algorithms, improper key management, insecure random |
| **Configuration** | Debug mode enabled, insecure defaults, missing security headers |
| **Dependencies** | Known vulnerable packages, outdated libraries |

### Risk Scoring

Assign security risk scores based on:

| Risk Level | Criteria | Action Required |
|------------|----------|-----------------|
| **Critical** | Direct path to exploit, affects auth/data | Block merge, immediate fix |
| **High** | Potential vulnerability, security-sensitive area | Require security review |
| **Medium** | Security anti-pattern, needs hardening | Recommend changes |
| **Low** | Minor concern, best practice deviation | Informational |

## Workflow

### 1. Extract Changes
Same as standard review - get PR diff and metadata.

### 2. Security Context Gathering

**If Glean MCP is available**, search for:
- Security policies and guidelines
- Past security incidents in affected components
- Compliance requirements (SOC2, GDPR, HIPAA)
- Security review templates and checklists

Use queries like:
- "security policy [component name]"
- "authentication requirements"
- "data handling guidelines"

### 3. Security Analysis

For each file, analyze:

**Code Patterns:**
- Input validation and sanitization
- Output encoding
- Error handling (information disclosure)
- Logging (sensitive data exposure)
- Authentication/authorization checks

**Configuration:**
- Security headers
- CORS settings
- TLS/SSL configuration
- Secret management

**Dependencies:**
- Check for known vulnerabilities (if package files changed)
- Outdated security-critical libraries

### 4. Create Security Review Artifacts

#### Security Findings Table
Use `mcp__miro__table_create`:

| Column | Type | Options |
|--------|------|---------|
| Finding | text | Description of issue |
| File | text | Location |
| Severity | select | Critical (#FF0000), High (#FF6600), Medium (#FFA500), Low (#FFFF00) |
| Category | select | Injection, Auth, AuthZ, Data, Crypto, Config, Dependencies |
| Status | select | Open (#FF0000), Mitigated (#00FF00), Accepted (#808080) |
| Remediation | text | Recommended fix |

#### Security Summary Document
Position at x=-2000, y=0:

```markdown
# Security Review: [PR Title]

**Review Date:** [date]
**Reviewer:** AI-assisted
**Risk Score:** [Critical/High/Medium/Low]

## Executive Summary
[One paragraph overview of security posture]

## Critical Findings
[List any critical/high severity issues]

## Compliance Impact
- [ ] PII handling changes
- [ ] Authentication changes
- [ ] Authorization changes
- [ ] Data retention impact
- [ ] Third-party data sharing

## Security Checklist
- [ ] Input validation present
- [ ] Output encoding applied
- [ ] Authentication verified
- [ ] Authorization checks in place
- [ ] Sensitive data protected
- [ ] Error handling secure
- [ ] Logging appropriate
- [ ] Dependencies secure

## Recommendations
[Prioritized list of security improvements]

## Related Security Docs
[Links from Glean if available]
```

#### Attack Surface Diagram
Use `mcp__miro__diagram_create` with `flowchart`:

Show:
- Entry points (APIs, user inputs)
- Data flows through changed components
- Trust boundaries crossed
- Potential attack vectors

## Example Usage

```
# Security review of PR
/miro-review:security https://miro.com/app/board/abc123= 42

# Security review of local changes
/miro-review:security https://miro.com/app/board/abc123= --local
```

## Output

After completion, provide:
1. Link to the Miro board
2. Overall security risk score
3. Count of findings by severity
4. Critical issues requiring immediate attention
5. Recommended next steps
