---
name: commit
description: Check working tree and staged changes for sensitive information, then generate a concise commit message. Use this when the user wants to commit code changes or asks for a commit message.
license: MIT
metadata:
  author: user
  version: "1.1.0"
---

# Commit Message Generator

Check working tree and staged changes for sensitive information, generate a one-line commit message, and wait for user confirmation before committing.

## Instructions

When this skill is invoked:

1. **Pick the change source (staged first, fall back to working tree)**
   ```bash
   git diff --staged --stat
   ```
   - **If staged has files** → use the staged area as the source. Remember this mode as `SOURCE=staged`.
   - **If staged is empty but the working tree has changes** (check with `git status --porcelain` or `git diff --stat`) → fall back to the working tree. Remember this mode as `SOURCE=working`.
     - Tell the user: "No staged changes detected, using working tree changes instead."
     - Include untracked files listed via `git ls-files --others --exclude-standard -z`. Check their actual contents in step 2 and analyze their sanitized contents as added files in step 4.
     - Staging happens right before the commit in step 6; keep the index unchanged during analysis.
   - **If both are empty** → nothing to commit. Inform the user and stop.

2. **Check both areas for sensitive information before displaying content**
   This check is required regardless of `SOURCE`. Source selection determines the commit scope, not the scan scope.

   **Collect and scan locally**
   - Capture the staged diff (`git diff --cached`) and the working tree diff (`git diff`) separately, and read non-ignored untracked files. Use `--no-ext-diff --no-textconv --no-color` for both diffs so external diff helpers and text conversions do not expose the content. Use NUL-delimited path lists when processing filenames.
   - Run this through an existing local scanner or a local script that captures content internally and emits only findings and sanitized content. Do not print raw diffs or file contents to tool output before scanning, and do not upload them to an external service. If no scanner exists, implement the local inspection without introducing a required dependency.
   - Inspect the actual index version and the actual working tree version independently, including partially staged files. A clean working tree version does not clear a secret still present in the index. Inspect enough surrounding content locally to recognize multiline secrets and sensitive data in newly added files.
   - Check added or modified content for credentials (API keys, access/refresh tokens, passwords, private keys, authorization headers, cookies, signed URLs, and connection strings containing credentials), personal data (real phone numbers, email addresses, identity numbers, payment details), and project-specific confidential data (internal endpoints, private IPs, account identifiers, or production configuration when context makes them confidential). Pay particular attention to `.env` files, credential files, logs, fixtures, and exported data.
   - Judge findings in context: field names, environment-variable references, explicit placeholders, public contact details, and clearly synthetic examples are not automatically leaks. Pattern matching alone is not a complete check; review sanitized changes for context-specific risks and inspect ambiguous values locally without displaying them.
   - Sensitive values appearing only in removed lines do not block a cleanup commit. Still redact them from diff context, deleted lines, and any other output. Retained or newly introduced sensitive content must be handled before proceeding.

   **Report and resolve findings**
   - Report only the area (`staged`, `working`, or `untracked`), file path, line number, sensitive-data category, and remediation advice. Replace sensitive content with a fixed marker such as `[REDACTED]`; do not include raw snippets, value prefixes/suffixes, or values in logs, temporary reports, commit messages, or the conversation. Mask sensitive path segments if necessary.
   - If a genuine or unresolved suspected leak is found in either area, stop before generating a commit message, staging, or committing. Explain the finding and suggest removing the data, replacing it with a placeholder, or referencing an environment variable/secret store. Do not automatically rewrite user files or change the index as part of this check.
   - Clear a false positive only with concrete context, recording the reason and location without copying the value. Ask the user only when the classification cannot be resolved from available context. A normal commit confirmation does not waive an unresolved finding.
   - Unreadable files, unsupported binary content, or truncated/skipped scans are incomplete checks. Report the affected paths and resolve coverage with an appropriate local check before proceeding; do not report a pass for unexamined content.
   - After remediation, rescan both areas and untracked files. A source-file fix must also reach the index when the staged version was affected. Summarize the check result without claiming that heuristic scanning guarantees the absence of sensitive information.

3. **Get the sanitized detailed changes for the selected source**
   - For `SOURCE=staged`, use the sanitized staged diff from step 2.
   - For `SOURCE=working`, use the sanitized working tree diff and sanitized contents of the untracked files from step 2.
   - Redact sensitive values in added lines, removed lines, and context before emitting content to tools or the conversation. Do not rerun a raw diff or file read for display after the check.
   - If the changes need splitting, follow step 8 before generating the final messages.

4. **Analyze the changes and generate a commit message**
   - Summarize the nature of changes (new feature, bug fix, refactor, docs, style, test, chore)
   - Use conventional commit format: `<type>: <description>`
   - Keep it under 72 characters
   - Focus on the "why" not the "what"
   - Use present tense, imperative mood (e.g., "add" not "added")
   - Use generic descriptions for sensitive-data cleanup; never copy a sensitive value into the message.

5. **Present the commit message to the user**
   list all the commits when finish generating the commit message
   Show the generated message and ask for confirmation using AskUserQuestion:
   - Option 1: Commit with this message
   - Option 2: Edit the message (let user provide custom message)
   - Option 3: Cancel

6. **Execute the commit if confirmed**
   - Recheck step 2 against the current changes before staging or committing. If the reviewed content or scope changed, refresh the analysis and obtain confirmation for the updated message/scope.
   - If `SOURCE=working`, stage only the specific reviewed paths or hunks with `git add -- <paths>` (or `git add -p`). Do not use `git add -A` to pick up changes that appeared after analysis.
   - If `SOURCE=staged`, preserve the reviewed index contents, especially when the same files have unstaged edits.
   - After staging or regrouping, check the actual staged diff again using step 2's local scanning and output-redaction rules. Commit only when the scan is complete and there are no unresolved findings.
   - If step 8's split flow applies, apply these checks to each individual commit.
   ```bash
   git commit -m "<message>"
   ```
   **Important:** Do NOT append any "Co-Authored-By" trailer to the commit message. e.g. Co-authored-by: Cursor <cursoragent@cursor.com> | Co-Authored-By: Claude code | 

7. **Show the result**
   Run `git log -1 --oneline` to confirm the commit was created.

8. **Divide multiple commits when staged/changed files too many**
   When the staged/changed files span unrelated concerns or the diff is too large to describe in one line, split them into several focused commits.
   - **Triggers for splitting**
     - Staged files exceed ~10, or the diff exceeds ~300 lines
     - Changes cross multiple commit types (e.g. `feat` + `fix` + `docs` + `chore` mixed together)
     - Changes touch unrelated modules / features / directories
     - A single message cannot honestly describe the "why" without using "and" / "、" to glue topics
   - **How to split**
     1. Preserve the reviewed source and scope. For `SOURCE=staged`, split from the original staged versions/hunks; do not reset the index and re-add working tree versions when they differ from the index. If the available tooling cannot preserve that boundary, keep a single commit or have the user adjust the grouping.
     2. Group files by concern in this priority order: commit type → feature/module → directory
     3. For `SOURCE=working`, stage each approved group precisely with `git add -- <paths>` (or `git add -p` for partial hunks). For either source, run steps 2–7 for each group, including checking its actual staged contents immediately before committing.
     4. Repeat until all approved groups are committed. Leave changes outside the approved scope for the user.
   - **Present the split plan first**
     Before executing, show the user the proposed groups and their commit messages via `AskUserQuestion`, and let them approve, adjust the grouping, or fall back to a single commit.
   - **Ordering**
     Commit lower-risk / independent changes first (`chore`, `docs`, `build`), then `refactor`, then `feat` / `fix` that depend on them, so history stays bisect-friendly.

## Commit Types

- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code change that neither fixes a bug nor adds a feature
- `docs`: Documentation only changes
- `style`: Formatting, missing semicolons, etc.
- `test`: Adding or updating tests
- `chore`: Build process, dependencies, or tooling changes
- `build`: Project configuration update or coding style settings likes ESLint \ StyleLint \ Prettier
- `flow`: Branches merge \ start \ close something else scence

## Examples

Good commit messages:
- `feat: add user authentication with JWT`
- `fix: resolve null pointer in payment processing`
- `refactor: extract validation logic to separate module`
- `docs: update API endpoint documentation`
- `chore: upgrade webpack to v5`
