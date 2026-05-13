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
