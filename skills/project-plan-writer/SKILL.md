---
name: project-plan-writer
description: Create or revise a project plan package for software or product work. Use when asked to write a plan/计划书, implementation plan, delivery plan, or project execution plan that needs coordinated decision tracking, sequenced engineering tasks, acceptance criteria, and milestone progress.
---

# Project Plan Writer

Create a reviewable plan package from the project brief, repository, existing docs, and user-provided constraints. Write in the user's language. Keep known facts separate from assumptions; use explicit `待确认` markers rather than inventing scope, owners, dates, or architecture.

## Workflow

1. Inspect the repository and existing planning conventions before creating files. Reuse established folders, naming, terminology, and document formats. If none exist, create a plan directory under `docs/plans/<project-or-feature>/` (or a similarly conventional location already present).
2. Gather the goal, scope, non-goals, stakeholders, constraints, target date, existing architecture/contracts, and known dependencies from the request and workspace. Ask only for information that blocks a coherent plan; otherwise proceed with labeled assumptions and open questions.
3. Create or update the four deliverables below. Use the matching templates in `assets/templates/` as a starting point, adapting them to repository conventions:
   - `plan.md` — plan body: context, goals, scope, approach, risks, dependencies, rollout/operations, and success measures.
   - `decisions.md` — pending decision questions. Start with a clean status: mark unresolved entries `待决策`; do not carry forward a resolved state as if it were current. If there are no known questions, state `当前无待决策问题` and leave the table ready for additions.
   - `tasks.md` — sequenced task table with shared contract design, common/setup orchestration (`C0`, `C1`...), backend (`B0`, `B1`...), frontend (`F0`, `F1`...), and test (`T1`, `T2`...) tasks. Include dependencies, owner or `待分配`, deliverable, and verifiable acceptance criteria. Include the test acceptance criteria explicitly.
   - `progress.md` — project execution overview with milestone records, target/actual dates, status, evidence or outcome, and blockers/next step.
4. Make ordering and dependencies explicit. Put cross-team prerequisites before dependent backend, frontend, and test work. Use stable task IDs and refer to those IDs from dependencies and milestones.
5. Check consistency across all four files: scope matches tasks; decisions are not silently treated as settled; every task has a concrete output and completion condition; milestones cover delivery; dates and owners are either sourced or marked pending.
6. Summarize created/updated paths and call out assumptions, unanswered decisions, and the first milestone. Do not claim execution progress that has not happened.

## Task planning rules

- Put shared API/schema/type definitions under a clearly named shared-contract task (for example `C0: 定义 @shared/contracts`); specify consumers, compatibility/versioning expectations, and review/approval needed when known.
- Use the `C` group for common work that must be decided or completed before implementation, such as scope alignment, architecture, environment setup, and contract design. Use `B`, `F`, and `T` for backend, frontend, and test work respectively. The prefixes are configurable if the repository already has an established convention.
- Express each task as an actionable verb plus object. Split work when it has different owners, dependencies, or independently verifiable outcomes.
- Capture dependencies as task IDs, and distinguish hard blockers from parallel work. Do not invent estimates; mark unknown estimates `待评估`.
- Write acceptance criteria as observable checks, including relevant API behavior, UI states, error paths, compatibility, and regression coverage. Avoid vague criteria such as “works as expected.”
- In the progress overview, record planned milestones initially as `未开始`; update actual status only from repository evidence or user-provided updates.

## Templates

Read and adapt these files when creating the package:

- [Plan body template](assets/templates/plan.md)
- [Decision questions template](assets/templates/decisions.md)
- [Task table template](assets/templates/tasks.md)
- [Milestone overview template](assets/templates/progress.md)
