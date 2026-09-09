# Course Context

## Course structure

The 10-week course is split into two five-week halves, each covering the same skill areas (code navigation, debugging, unit testing/coverage, code review, code quality tooling) but with a different codebase and a different constraint on tooling:

- **Weeks 1–5 — traditional software engineering, no AI or agents.** Codebase: Python's **idlelib**. Students work by hand.
- **Weeks 6–10 — the same skills, agentic/AI-assisted.** Codebase: **this OpenRefine fork**. Students apply AI/agent tooling (Claude Code) to a codebase substantial enough to carry real architectural tradeoffs.

The course runs guided homeworks first, then a 4-person group project: add a substantial feature/modification with test coverage, diagrams, and git-workflow discipline (branching, PR, code review).

## Why idlelib for the first half, OpenRefine for the second

An earlier plan used OpenRefine across all 10 weeks, replacing a prior offering that ran on idlelib for its full duration. That prior idlelib-only offering had become too small for a full-term agentic course: students reached a complete mental model too easily, there were too few real architectural tradeoffs to analyze, and idlelib's modules interact too thinly to support meaningful sequence diagrams.

Under the current two-half plan, that critique applies specifically to the AI-assisted half, where students need a codebase big enough to require real navigation and carry genuine tradeoffs. idlelib remains well suited to the first half, where the point is to build traditional skills by hand on a small, fully graspable codebase before AI tooling enters the picture in week 6.

## Artifacts for learning this codebase

- Architecture walkthrough (module dependency diagram, request-lifecycle diagram, plugin-registration diagram, suggested reading order through real files): https://claude.ai/code/artifact/1fca58dc-7118-4c42-819c-3a4deb774f0d
- "OpenRefine Blueprint" — five verified UML/architecture diagrams with file:line citations (module dependency graph, request-lifecycle sequence diagram, Command/Operation/History class diagram, core data model class diagram, plugin-registration flow): https://claude.ai/code/artifact/b4e8d204-33f3-472c-94c9-ae21fddfb8ad
