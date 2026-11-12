---
name: code-review
description: >
  Perform a thorough code review on pending workspace changes for git or mercurial projects.
  Detects VCS type automatically, collects the full diff, then evaluates correctness, security,
  performance, maintainability, and test coverage. Classifies findings as High / Medium / Low
  and outputs a structured review report. Use when the user asks to review changes, audit a
  diff, check a branch before merging, or wants a second opinion on pending code.
license: MIT
metadata:
  author: user
  version: "1.0.0"
---

# Code Review

Review pending workspace changes as a senior engineer would before a merge — correctness first,
then security, performance, and maintainability.

## Step 1 — Detect VCS type

Check the project root:

```bash
ls .gitignore .hgignore 2>/dev/null
```

- `.gitignore` present → **git**
- `.hgignore` present → **mercurial**
- Both → prefer git
- Neither → inform the user and stop

## Step 2 — Collect the diff

**git** (gather all three views):

```bash
git diff HEAD                        # unstaged changes vs last commit
git diff --staged                    # staged changes
git diff HEAD --stat                 # summary of affected files
git log --oneline -10                # recent context
```

If the user passed a base ref (e.g., `/code-review main`), use:

```bash
git diff <base-ref>...HEAD           # branch diff against base
git diff <base-ref>...HEAD --stat
```

**mercurial:**

```bash
hg diff                              # pending changes
hg status                            # change summary
hg log -l 10 --template "{rev}: {desc|firstline}\n"   # recent context
```

If the workspace is clean (no changes), tell the user and stop.

## Step 3 — Read changed files in full

For each modified file in the diff, read the **entire** file (not just the changed hunks) so
you understand surrounding context, existing patterns, and intent. Limit to files under 1000
lines; for larger files read only the changed sections plus ±50 lines of context.

## Step 4 — Analyse the changes

Evaluate every changed file against **all** dimensions below. Flag only real issues — do not
invent problems where the code is correct.

### Correctness & Logic
- Off-by-one errors, wrong operator precedence, incorrect conditionals
- Edge cases: empty input, null/nil/undefined, zero, negative numbers, overflow
- Concurrency: race conditions, improper locking, shared-state mutation
- Error handling: swallowed errors, missing early returns, wrong error types
- API contract violations: wrong argument order, ignored return values

### Security (OWASP Top 10 + common pitfalls)
- Injection: SQL, command, LDAP, XPath, template injection
- Broken authentication: hard-coded secrets, weak tokens, improper session handling
- Sensitive data exposure: secrets in logs, plaintext passwords, unencrypted storage
- Insecure deserialization or file handling
- Path traversal, SSRF, open redirects
- Missing input validation / output encoding at trust boundaries
- Dependency with known CVE (flag the package name; do not look up CVEs online)

### Performance
- N+1 queries or unnecessary repeated I/O inside loops
- Missing indexes hinted by query patterns
- Unbounded loops or recursion without memoization
- Large allocations, memory leaks, unnecessary copies
- Blocking I/O on hot paths

### Maintainability & Design
- Single Responsibility violations: function / class doing too many things
- Deep coupling or circular dependencies introduced
- Magic numbers / strings without named constants
- Duplicated logic that should be extracted
- Misleading names that contradict behavior
- Dead code or commented-out blocks

### Test Coverage
- New logic paths with no corresponding test
- Tests that only assert the happy path, skipping error / edge cases
- Mocks that diverge from the real interface

### Documentation & Observability
- Public API / exported function with no docstring where one is expected by convention
- Missing or misleading log messages around critical state transitions
- TODO/FIXME left without a tracking reference

## Step 5 — Classify findings

| Priority | Criteria |
|----------|----------|
| **High** | Correctness bug, security vulnerability, or data-loss risk. Ship-blockers. |
| **Medium** | Performance regression, design debt that compounds, missing test for critical path. |
| **Low** | Style preference, minor readability issue, nice-to-have improvement. |

When unsure between High and Medium, choose Medium.
When unsure between Medium and Low, choose Low.

## Step 6 — Output the report

Match the report language to the codebase (Chinese comments → Chinese report, English → English).

```markdown
# Code Review — <date or branch name>

## Overview

| Severity | Count |
|----------|-------|
| High     | N     |
| Medium   | N     |
| Low      | N     |

[1–2 sentences: what the change does and the single biggest concern, or "LGTM" if clean.]

---

## High Priority

### [H1] <Issue title>
- **File**: `path/to/file.ext:line`
- **Problem**: [What is wrong and why it matters — be concrete]
- **Suggestion**: [How to fix it, with a brief code sketch if helpful]

### [H2] ...

---

## Medium Priority

### [M1] <Issue title>
- **File**: `path/to/file.ext:line`
- **Problem**: [What is wrong]
- **Suggestion**: [Improvement approach]

---

## Low Priority

- **[L1] <Issue title>** — `file:line`: [One-line description]
- **[L2] ...**

---

## Positive Notes  _(omit section if nothing notable)_

- [Highlight a well-designed part of the change — keeps review balanced]
```

## Guidelines

- **Be specific** — every finding must cite an exact file path and line number.
- **No invented problems** — if the code is correct and clean, say so.
- **Security first** — always complete the security dimension before moving on.
- **One finding per issue** — don't split a single bug into multiple items.
- **Actionable suggestions** — each High/Medium must have a concrete fix direction.
- **Positive notes are optional** — only include if genuinely noteworthy, not as flattery.
- If no changes are present or VCS cannot be detected, stop and explain clearly.
