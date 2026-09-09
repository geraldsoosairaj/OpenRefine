# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

**Requirements**: JDK 17+, Maven 3.8+, Node.js 18+

```bash
./refine            # Run the application (builds if needed, serves at localhost:3333)
./refine build      # Compile only
./refine clean      # Clean compiled classes
./refine lint       # Reformat source code — MUST run before submitting a PR
```

## Testing

```bash
./refine test                           # Run all tests
mvn test -f main                        # Run main module tests only (same as ./refine server_test)
mvn test -f extensions                  # Run extension tests only
./refine e2e_tests                      # Run Cypress end-to-end tests

# Run a single test class
mvn test -f main -Dtest=MyClassTests

# Run a single test method
mvn test -f main -Dtest=MyClassTests#myMethod
```

Test class names must end in `Tests.java`. The test JVM is configured with the French locale (`fr_FR`) to catch locale-sensitivity bugs.

## Architecture

OpenRefine is a **Maven multi-module Java project** with an embedded Jetty server. The UI is a single-page JavaScript/jQuery application served via Velocity templates.

### Module layout

| Module | Purpose |
|--------|---------|
| `modules/core` | Data model: `Project`, `Row`, `Column`, `Cell`, `History`, commands, importers, exporters, `RefineServlet` |
| `modules/grel` | GREL expression language parser and evaluator |
| `main` | Concrete importers/exporters/operations, the webapp (`main/webapp`), integration tests |
| `server` | Jetty server bootstrap |
| `extensions/` | Optional plugins: `database`, `jython`, `pc-axis`, `wikibase` |
| `benchmark` | JMH benchmarks |

### Backend request flow

`RefineServlet` (in `modules/core`) routes HTTP requests to registered `Command` implementations. All data mutations are modeled as `Operation` objects appended to a `History`, enabling undo/redo. The plugin/extension system is built on the **Butterfly** framework, which handles dynamic discovery of commands, importers, exporters, and reconciliation services.

Key packages under `com.google.refine`:
- `commands/` — HTTP command handlers (one class per endpoint)
- `model/` — core data types (`Project`, `Row`, `Column`, `Cell`, `Record`)
- `operations/` — undoable data transformations
- `importers/` / `exporters/` — format adapters
- `history/` — undo/redo infrastructure
- `expr/` — expression evaluation (GREL, Jython, Clojure)
- `process/` — async process execution

### Frontend

Static assets live in `main/webapp/`. The frontend is plain JavaScript (no build step beyond dependency copying via `copy-dependencies.js`). UI modules are organized by feature: `project.js`, `facets/`, `reconciliation/`, `dialogs/`, `views/`, `widgets/`. Velocity templates handle server-side rendering of the initial HTML shell.

### Extensions

Extensions follow the Butterfly plugin pattern: each extension has a `controller.js` (MOD-INF) that registers commands, importers, exporters, and overlay models at startup. To add behavior, register it there rather than modifying core.

## Code Conventions

- Run `./refine lint` before every PR — CI will fail if skipped. This runs `mvn formatter:format impsort:sort`.
- Import order is enforced: `java.*`, `javax.*`, third-party (`*`), `com.google.refine.*`, `org.openrefine.*`.
- Eclipse formatter config: `IDEs/eclipse/Refine.style.xml`.
- Branch names should include the issue number: `issue-1234-short-description`.

## Course Context

This fork is used for UC San Diego's CSE 190 "Large Codebases" course, replacing a prior offering built on Python's idlelib (which had become too small: full mental model reached too easily, too few real architectural tradeoffs to analyze, and thin interaction between modules for sequence diagrams). Each week pairs a traditional software-engineering skill (code navigation, debugging, unit testing/coverage, code review, code quality tooling) with an agentic/AI-assisted version of the same skill. The 10-week arc runs guided homeworks first, then a 4-person group project: add a substantial feature/modification with test coverage, diagrams, and git-workflow discipline (branching, PR, code review).

OpenRefine was chosen over Zulip and Gitea (both audited at the source level before deciding) because it's the lightest to run at class scale (single JVM process, no external services), the smallest ramp for an instructor with Java-teaching-but-no-professional-web-experience background, and it already ships a maintainer-authored `CLAUDE.md` (this file). Zulip had richer distributed-systems architecture (Django/Tornado split) but meant a fully new stack plus heavier Vagrant/Docker dev-environment setup for ~30-100 students; Gitea had the lightest single-binary setup and its own `AGENTS.md`, but Go would have been a brand-new language for the instructor and nearly all students.

- **Onboarding artifact**: an architecture walkthrough (module dependency diagram, request-lifecycle diagram, plugin-registration diagram, suggested reading order through real files) was published for the instructor at https://claude.ai/code/artifact/1fca58dc-7118-4c42-819c-3a4deb774f0d.
- **Good bounded team-project targets**: the four bundled extensions (`extensions/database`, `extensions/jython`, `extensions/pc-axis`, `extensions/wikibase`) are self-contained and each registers into the same `Command` registry as core via its own `MOD-INF/controller.js` — a real, precedented pattern for "add a feature without touching core," analogous to Zulip's `zerver/webhooks/` or Gitea's `services/webhook/`.
- **Sequence-diagram-worthy flow** (good Week 2 material): Browser → `RefineServlet.service()` (verb dispatch by URL) → `Command` → `Operation` → mutates `Project`/`Row`/`Cell` and is recorded in `History` (undo/redo). Trace it in `modules/core/src/main/java/com/google/refine/RefineServlet.java`, `modules/core/src/main/java/com/google/refine/history/History.java`, and e.g. `main/src/com/google/refine/operations/column/ColumnRenameOperation.java`.
- **Architectural-decision teaching point**: undo/redo isn't bolted on — it's a structural consequence of every mutation being forced through the `Operation → History` path. Good raw material for the "how architectural decisions affect maintainability" learning goal.
- **Live good-first-issue count fluctuates** — as of 2026-08-31 there were 3 open (`gh issue list --repo OpenRefine/OpenRefine --search 'label:"good first issue" is:open'`); re-check close to the term start rather than relying on a stale count.
