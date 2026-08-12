# Org-Level Rules

> Framework defaults. Read in order with aidlc-team.md and
> aidlc-project.md; later layers override.

## Way of Working

We use **trunk-based development**. All work merges to `main` via
short-lived feature branches (typically resolved within 1-2 days).
Long-lived branches accumulate merge debt; we avoid them.

For Construction worktrees, the worktree base branch is `main` and the
merge target is `main`.

If our project requires multiple environments (staging, production), we
still keep one trunk and gate releases via tags or environment-specific
deployment configs — not via long-lived release branches.

We **squash-merge** Bolt branches into `main`. Each Bolt becomes one
commit on the trunk, named by the Bolt slug, with the full Bolt commit
history preserved on the source branch until the worktree is discarded.

Squash gives us a clean linear `main` history that maps 1:1 to
delivery-planning's Bolt sequence. We accept the trade-off of losing
intermediate commits on `main` because the audit log preserves the full
event sequence anyway.

## Walking Skeleton

We always run the walking-skeleton Bolt **first** when our scope is
greenfield (`mvp`, `enterprise`, `feature`, `poc`, `workshop`, `infra`).
Bolt 1 is solo, gated, and the user explicitly approves before remaining
Bolts run.

We **skip the skeleton ceremony** when our scope is incremental work on
an existing codebase (`bugfix`, `refactor`, `security-patch`). The first
Bolt runs like any other — there's nothing to bootstrap.

After Bolt 1 ships (when it runs), the orchestrator fires the **ladder
prompt**: "How should the remaining Bolts run?" Options: continue
autonomously, gate every Bolt. The team picks per project. The choice
persists as `Construction Autonomy Mode` in `aidlc-state.md`.

## Testing Posture

We treat tests as a first-class deliverable in every Bolt. Specific
methodology — TDD, BDD, ATDD, or classic test-after — is captured by the
testing-strategy stage when it ships.

Until then, our default per scope is:
- `mvp`, `enterprise`, `feature`, `infra` → tests written alongside
  code; minimum 80% line coverage; tests run in CI before merge.
- `bugfix`, `security-patch` → regression test for the specific
  bug/vulnerability; existing test suite must remain green.
- `poc`, `refactor`, `workshop` → existing test suite remains green;
  no new test floor required.

Override at `aidlc-team.md` if the team commits to a stricter posture.

## Deployment

We **deploy on merge** to staging environments. Production deploys gate
on a separate manual approval — typically tech lead + product owner
sign-off in CodePipeline or a CD platform's environment protection.

Teams that have invested in test coverage and observability sometimes
graduate to continuous deployment to production (every commit
auto-deploys); that's a team decision, not a framework default.

## Code Style

We defer to project-level configurations:
- Formatter: Prettier (JS/TS), Black (Python), `gofmt` (Go), or
  language-default. Configured in repo root (`.prettierrc`,
  `pyproject.toml`, etc.).
- Linter: ESLint, Ruff, golangci-lint, etc. Run in CI before merge;
  failure blocks the PR.
- Naming conventions: language idiomatic (camelCase for JS/TS,
  snake_case for Python, etc.). No project-wide rename rules unless
  team affirms one.

When the framework makes a code-style suggestion, agents read the
project's linter config first; the agent's suggestion only fires if the
linter doesn't already cover it.

## Forbidden

<!-- Things agents must never do -->
<!-- Example: Do not ask questions about topics already decided in previous stages -->

## Mandated

<!-- Things agents must always do -->
<!-- Example: All architecture decisions must include an ADR -->

## Corrections

<!-- Self-learning loop appends here. -->
<!-- Use aidlc-team.md to record team-wide overrides; aidlc-project.md
     to record project-specific deviations. The loaders merge org → team
     → project at session start; each layer replaces fields the layer
     above set; missing fields fall through. -->


---

# Team-Level Rules

> This team's affirmed practices and corrections. Overrides aidlc-org.md.
> Populated by practices-discovery affirmation gate. Edit at the gate,
> not directly.

## Way of Working

<!-- Affirmed during practices-discovery. Example: -->
<!-- We use GitHub Flow with feature branches. Branches live 3-5 days max. -->
<!-- Hotfixes branch from main and merge back via expedited review. -->

## Walking Skeleton

<!-- Affirmed during practices-discovery. Example: -->
<!-- We don't run a walking skeleton — our deployment pipeline is mature -->
<!-- and the slice cost outweighs the value at our maturity stage. -->

## Testing Posture

<!-- Affirmed at requirements-analysis. Example: -->
<!-- We use BDD. Specifications drive scenarios; scenarios drive code. -->
<!-- Each Unit ships with feature files in /features/. -->

## Deployment

<!-- Affirmed during practices-discovery. -->

## Code Style

<!-- Team-specific conventions beyond the linter. Example: -->
<!-- - Prefer named exports over default exports -->
<!-- - All async functions return Result<T, E>, never throw -->

## Forbidden

<!-- Team-specific forbidden patterns -->

## Mandated

<!-- Team-specific mandates -->

## Corrections

<!-- Self-learning loop appends here. -->


---

# Project-Level Rules

> Project-specific overrides and corrections. Overrides aidlc-team.md
> and aidlc-org.md. Populated by practices-discovery and the
> self-learning loop.
>
> Use sparingly: most teams don't need a project layer. Reach for it
> only when this specific project deviates from team-wide practice in a
> stable, durable way (e.g., "this monorepo project rebases even though
> our team default is squash"; "this legacy project skips the test
> floor because the existing suite is unsalvageable and we accept
> that").

## Way of Working

<!-- Project-specific override. Example: -->
<!-- This monorepo project rebases instead of squash-merging because -->
<!-- the per-package commit history is the audit trail we depend on -->
<!-- for partial-rollback decisions. Override applies to this project -->
<!-- only. -->

## Walking Skeleton

<!-- Project-specific override. Example: -->
<!-- This project skips the walking skeleton because we're rewriting -->
<!-- an existing service in-place — there's no greenfield bootstrap -->
<!-- to gate. -->

## Testing Posture

<!-- Project-specific override. -->

## Deployment

<!-- Project-specific override. -->

## Code Style

<!-- Project-specific override. -->

## Tech Stack

<!-- Technology choices locked for this project. -->

## Decided

<!-- Decisions made in earlier stages that should not be re-asked. -->
<!-- Format: DECIDED: [decision] (Stage [slug], [date]) -->

## Scope Overrides

<!-- Custom scope rules for this project. -->

## Forbidden

<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: NEVER [behavior] (affirmed [date]) -->
<!-- Example: NEVER throw exceptions across service layer boundaries (affirmed 2026-05-17) -->

## Mandated

<!-- Populated by practices-discovery affirmation gate. -->
<!-- Format: ALWAYS [behavior] (affirmed [date]) -->
<!-- Example: ALWAYS use Result<T,E> for fallible operations in service layer (affirmed 2026-05-17) -->

## Corrections

<!-- Project-specific corrections from human feedback. -->
<!-- Format: NEVER/ALWAYS [behavior] (learned [date]) -->


---

ALWAYS run the walking-skeleton Bolt first (solo, gated) for any net-new service added to this brownfield repo. A new Spring Boot microservice without existing scaffolding is greenfield-within-brownfield and requires a runnable baseline before feature Bolts proceed.

---

ALWAYS write tests using TDD (tests before or alongside code) for new microservice classes. ALWAYS enforce ≥ 80% line coverage on controller and service classes as a hard gate before dependent frontend integration proceeds. Use JUnit 5 + Mockito.

---

NEVER modify files outside the microservices/ directory during Phase 2.1. The existing JtProject monolith (AdminController, UserController, JSPs) must not be changed to prevent regressions. This is a hard constraint enforced at every Bolt gate.

---

ALWAYS scaffold microservices/catalog-service as a new standalone Spring Boot 3.x module using Java 17+ and jakarta.* namespace. NEVER import or reuse code from the monolith JtProject/ module. NEVER use javax.* in catalog-service. The monolith stays at Spring Boot 2.6.4/javax.* and is not modified by this intent.

---

NEVER modify JtProject/ (the Spring Boot 2.6.4 monolith) as part of the catalog-service error-handling intent. The scope is strictly limited to microservices/catalog-service. Do not touch AdminController.java, UserController.java, or any JSP in the monolith.

---

NEVER use System.out.println or e.printStackTrace() in production Java source code in this project. All diagnostic output must go through SLF4J (logger.error, logger.warn, logger.info, logger.debug). This was affirmed during practices-discovery 2026-08-12.

---

NEVER include exception messages, stack traces, JDBC connection strings, or internal class names in HTTP response bodies or JSP view output. This is an OWASP CWE-209 information leakage constraint affirmed during practices-discovery 2026-08-12.

---

ALWAYS use SLF4J logger.error(message, throwable) when logging caught exceptions — the Throwable must be the last argument so the full stack trace reaches the log appender. Affirmed during practices-discovery 2026-08-12.

---

ALWAYS enforce ≥80% JaCoCo line coverage on new or modified @Controller and @Service classes as a hard gate before Bolt merge. Affirmed during practices-discovery 2026-08-12.

---

# Construction Phase Guardrails

These rules apply to every stage whose `phase: construction` declaration
imports them as the matching phase rule.

## Code Completeness

- Generate complete, runnable files — no partial implementations, no placeholder stubs unless explicitly marked TODO with a rationale
- Every generated module must be independently executable or clearly document its dependencies
- Do not leave unresolved import errors, missing type definitions, or broken references

## Error Handling

- Always include error handling at integration boundaries (API calls, database operations, file I/O, external services)
- Errors must be surfaced to the caller or logged — silent failures are not acceptable
- Distinguish between recoverable errors (retry/fallback) and fatal errors (fail fast)

## Testing Standards

- Test files must cover the happy path and at least two error/edge cases
- Tests must be runnable without manual setup beyond documented prerequisites
- Do not generate tests that always pass regardless of implementation (e.g., `assert True`)

## Security

- Never hardcode credentials, API keys, or secrets — use environment variables or a secrets manager
- Validate and sanitize all inputs at system boundaries
- Flag any code that bypasses authentication or authorization checks

## Corrections
