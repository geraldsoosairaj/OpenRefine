# Course Context

This file documents why and how this OpenRefine fork is used for UC San Diego's CSE 190 "Large Codebases" course. It is the canonical, up-to-date summary of that decision — it replaces the "Course Context" section that used to live in `CLAUDE.md`.

## Course structure

The 10-week course is split into two five-week halves, each covering the same skill areas (code navigation, debugging, unit testing/coverage, code review, code quality tooling) but with a different codebase and a different constraint on tooling:

- **Weeks 1–5 — traditional software engineering, no AI or agents.** Codebase: Python's **idlelib**. Students work by hand.
- **Weeks 6–10 — the same skills, agentic/AI-assisted.** Codebase: **this OpenRefine fork**. Students apply AI/agent tooling (Claude Code) to a codebase substantial enough to carry real architectural tradeoffs.

The course runs guided homeworks first, then a 4-person group project: add a substantial feature/modification with test coverage, diagrams, and git-workflow discipline (branching, PR, code review).

## Why idlelib for the first half, OpenRefine for the second

An earlier plan used OpenRefine across all 10 weeks, replacing a prior offering that ran on idlelib for its full duration. That prior idlelib-only offering had become too small for a full-term agentic course: students reached a complete mental model too easily, there were too few real architectural tradeoffs to analyze, and idlelib's modules interact too thinly to support meaningful sequence diagrams.

Under the current two-half plan, that critique applies specifically to the AI-assisted half, where students need a codebase big enough to require real navigation and carry genuine tradeoffs. idlelib remains well suited to the first half, where the point is to build traditional skills by hand on a small, fully graspable codebase before AI tooling enters the picture in week 6.

## Why OpenRefine (for the second half)

OpenRefine was chosen over Zulip and Gitea, audited at the source level (not just docs/README summaries) before deciding, because it's the lightest to run at class scale (single JVM process, no external services), the smallest ramp for an instructor with a Java-teaching-but-no-professional-web-experience background, and it already ships a maintainer-authored `CLAUDE.md`. Zulip had richer distributed-systems architecture (Django/Tornado split) but meant a fully new stack plus heavier Vagrant/Docker dev-environment setup for ~30-100 students. Gitea had the lightest single-binary setup and its own `AGENTS.md`, but Go would have been a brand-new language for the instructor and nearly all students.

## Teaching hooks in this codebase

- **Onboarding artifacts** (published for the instructor):
  - Architecture walkthrough (module dependency diagram, request-lifecycle diagram, plugin-registration diagram, suggested reading order through real files): https://claude.ai/code/artifact/1fca58dc-7118-4c42-819c-3a4deb774f0d
  - "OpenRefine Blueprint" — five verified UML/architecture diagrams with file:line citations (module dependency graph, request-lifecycle sequence diagram, Command/Operation/History class diagram, core data model class diagram, plugin-registration flow): https://claude.ai/code/artifact/b4e8d204-33f3-472c-94c9-ae21fddfb8ad
- **Good bounded team-project targets**: the four bundled extensions (`extensions/database`, `extensions/jython`, `extensions/pc-axis`, `extensions/wikibase`) are self-contained and each registers into the same `Command` registry as core via its own `MOD-INF/controller.js` — a real, precedented pattern for "add a feature without touching core," analogous to Zulip's `zerver/webhooks/` or Gitea's `services/webhook/`.
- **Sequence-diagram-worthy flow** (good material for the early weeks of the OpenRefine half): Browser → `RefineServlet.service()` (verb dispatch by URL) → `Command` → `Operation` → mutates `Project`/`Row`/`Cell` and is recorded in `History` (undo/redo). Trace it in `modules/core/src/main/java/com/google/refine/RefineServlet.java`, `modules/core/src/main/java/com/google/refine/history/History.java`, and e.g. `main/src/com/google/refine/operations/column/ColumnRenameOperation.java`.
- **Architectural-decision teaching point**: undo/redo isn't bolted on — it's a structural consequence of every mutation being forced through the `Operation → History` path. Good raw material for the "how architectural decisions affect maintainability" learning goal.
- **Live good-first-issue count fluctuates** — as of 2026-08-31 there were 3 open (`gh issue list --repo OpenRefine/OpenRefine --search 'label:"good first issue" is:open'`); re-check close to the term start rather than relying on a stale count.
