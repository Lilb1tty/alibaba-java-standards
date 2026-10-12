---
name: alibaba-java-standards
description: "Apply Alibaba Java Development Manual, Songshan edition, when writing or reviewing Java code, SQL/MyBatis mappings, API contracts, tests, dependencies, or system designs. Use for edition-specific rules and chapter lookup."
---

# Alibaba Java Development Manual: Songshan Edition

**Author:** Alibaba Group technical teams and community contributors.  
**Edition:** 1.7.0, released 2020-08-03. **Source:** 63 PDF pages; seven main sections.  
**Generated:** 2026-10-12. **Purpose:** Apply rules while coding and reviewing.

## Apply This Skill

1. Identify the changed behavior, project conventions, Java/database/framework versions, and relevant boundaries. Follow explicit user instructions and project requirements when they differ from this edition; explain the difference.
2. Read the relevant chapter before making a detailed rule claim. Use the topic index below. Load [cheatsheet.md](cheatsheet.md) for decisions, [patterns.md](patterns.md) for implementation recipes, and [glossary.md](glossary.md) for terminology.
3. For new code, apply relevant mandatory rules first. Preserve API compatibility, resource ownership, numeric precision, access control, and data integrity. Use recommendations with their stated trade-offs.
4. For reviews, identify concrete evidence in the changed code and affected call paths. Report location, rule level, source section, failure mode, and smallest useful correction. A rule violation and its production severity are separate judgments. Mark edition-only convention concerns accordingly.
5. Check relevant behavior with the project's existing checks. Report what was verified and any unresolved assumptions. Avoid unrelated rewrites.

Without a topic, use the core rules below. A chapter request such as `ch05` loads that chapter. A browse request returns the chapter index. Read the source PDF for unindexed details; this is a synthesized toolkit, not a verbatim or exhaustive rule catalog.

## Rule Levels and Source Interpretation

| Label used here | Source level | Meaning |
|---|---|---|
| **M** | Mandatory | Required by this edition within its stated scope. |
| **R** | Recommended | Preferred practice; evaluate context and trade-offs. |
| **F** | Reference | Guidance or convention; do not promote it to mandatory. |

Chapter source locators use the manual's printed page numbers, not physical PDF page numbers. Rule numbering restarts in each subsection. Worked examples are compact English adaptations of source scenarios; invented application details are labeled adaptations. Runtime notes distinguish technical clarification from the edition's policy.

## Core Rules and Mental Models

### Preserve Meaning at Boundaries

- **M:** Use wrapper types for POJO fields and RPC parameters/results. An absent value must remain distinguishable from zero or false. Check nullable values before unboxing, including mixed-type ternary expressions.
- **M:** Compare boxed integral values with `equals`, safely handling null. Compare `BigDecimal` numeric values with `compareTo`, because `equals` also compares scale. Construct decimal values from strings or `valueOf`, not `BigDecimal(double)`.
- **M:** Store money as integer counts of the smallest currency unit. SQL `DECIMAL` is the separate rule for fractional decimal columns; define unit/scale explicitly.
- **M:** Match DO fields to database types. Map database `is_xxx` columns to POJO properties without the `is` prefix. Do not give POJO fields implicit default values that can overwrite stored data.
- **M:** Send very large integer identifiers to JavaScript clients as strings. Specify API methods, headers, payloads, status codes, and errors. Empty list responses use empty containers; this does not prohibit documented null returns in internal Java methods.

### Bound Resources and Own Cleanup

- **M:** Use explicitly configured `ThreadPoolExecutor` pools with meaningful thread names; avoid the unbounded queue/thread defaults of `Executors` factories in this edition. Choose capacity and rejection behavior deliberately.
- **M:** Remove custom `ThreadLocal` values in `finally`, especially on reused pool threads. Close owned streams/resources; try-with-resources is appropriate on Java 7+.
- **M:** Acquire a blocking lock immediately before `try`, release in `finally`; for `tryLock`, enter/release only after success. Acquire multiple locks in the same order. Keep lock scope small and avoid RPC inside locks.
- **M:** Protect concurrent record updates against lost updates. `volatile` provides visibility, not atomic compound operations. Use atomics, locks, or database version checks as appropriate.
- The source's optimistic-lock heuristic recommends conflicts below 20% and at least three retries. Its financial-data recommendation prefers pessimistic locking: lock, validate, update, release. These are source guidelines, not workload guarantees.

### Make Collection Contracts Explicit

- **M:** Implement consistent `equals`/`hashCode` for custom value keys and set elements. Use `isEmpty` for emptiness checks.
- **M:** Provide a duplicate-key merge policy to `Collectors.toMap`; prevent null mapped values. Distinguish list views, fixed-size lists, immutable containers, and independent copies.
- **M:** Do not structurally mutate collections through enhanced-for iteration. Use iterator removal; coordinate all shared access when concurrency is involved.
- **M:** Use typed array conversion, `list.toArray(new String[0])`, under this edition. Respect comparator antisymmetry, transitivity, and consistent treatment of equality.
- **PECS:** Producer Extends, Consumer Super. Read typed values from `? extends T`; write `T` into `? super T`. Reading the latter yields `Object`, not a guaranteed `T`.

### Express Failure Once, at the Correct Boundary

- **M:** Prevent foreseeable NPE/bounds failures with checks; do not use exceptions as normal branching. Catch specific unstable operations and handle or propagate the failure. Preserve required rollback when catching transaction exceptions.
- **M:** No `return` in `finally`. Include safe event context and the throwable in logs. Use a logging facade, placeholders, and level guards for trace/debug/info. Avoid duplicate logging and direct production console/stack printing.
- **M:** Error codes are five-character strings: source letter `A`, `B`, or `C` plus four digits; `00000` means success when a code is required. Sources are user, current system, and external service. Reuse registered codes; do not encode versions, log levels, or organization structure.
- **F:** External HTTP APIs return error codes; cross-application RPC prefers a Result envelope; internal application failures can use exceptions. Translate errors into useful user messages without disclosing sensitive context.
- The source mandates `Throwable` interception at RPC, internal-library, and dynamically generated invocation boundaries. Keep that edition requirement boundary-specific; read ch02's fatal-error caveat before implementation.

### Test Behavior with AIR and BCDE

- **M — AIR:** **Automatic, Independent, Repeatable.** Assert results without manual inspection; tests do not call each other or depend on order, network availability, or existing database records.
- **R — BCDE:** **Border, Correct, Design, Error.** Include boundaries, valid cases, design requirements, and deliberate invalid inputs/failure paths.
- **M:** Keep unit tests in `src/test/java`, usually at method scope and at most class scope. New core behavior must pass unit tests. Cross-class/system interaction belongs to integration testing.
- **R:** Source coverage goals are 70% statement coverage generally and 100% statement plus branch coverage for core modules. Coverage complements meaningful assertions; it is not proof of correctness.

### Enforce Trust and Integrity at Their Source

- **M:** Check ownership/authorization for user-specific pages and operations. Mask displayed sensitive data. Validate every user request parameter, including sizes, sorting, URLs, uploads, and regex input.
- **M:** Bind SQL values; allowlist metadata choices such as identifiers and sort directions. Escape/filter user data before HTML output. Validate CSRF protections for forms/AJAX and allowlist external redirects.
- **M:** Apply anti-replay/abuse controls to SMS, email, calls, orders, and payments. The source lists count limits, frequency controls, and verification checks; choose controls that address the actual threat.
- **M:** Put business uniqueness in database unique indexes; application check-then-insert is insufficient under concurrency. Preview correction targets with SELECT before changes.

### Design for Inspectable Change

- **R:** Give Web, Service, Manager, and DAO clear responsibilities. Manager adapts external services and factors shared behavior; it is not a mandatory pass-through layer.
- **M:** Review and document storage/data-structure designs and changes. Use diagrams when source thresholds apply: multiple actor types and more than five use cases; more than three states; more than three objects in a call chain; more than five model classes with complex dependencies; more than two collaborating objects with complex workflow.
- **R:** Apply single responsibility, composition before inheritance, Liskov substitution when inheriting, dependency inversion, open/closed design, and DRY. Isolate real variation points; retain documents explaining dependencies, business boundaries, and nonfunctional requirements.
- **M:** Keep library releases traceable, do not overwrite released versions, avoid production SNAPSHOT dependencies except the source's security-package exception, and evaluate transitive dependency resolution changes.

## Chapter Index

| Chapter | Subject | Main tools | Printed pages |
|---|---|---|---|
| [ch01](chapters/ch01-programming.md) | Programming conventions | Naming, precision, dates, collections, concurrency, control flow, APIs | 1-28 |
| [ch02](chapters/ch02-errors-and-logging.md) | Error codes, exceptions, logging | A/B/C codes, cleanup, boundary translation, logging ownership | 29-34 |
| [ch03](chapters/ch03-unit-testing.md) | Unit testing | AIR, BCDE, test isolation, coverage | 35-36 |
| [ch04](chapters/ch04-security.md) | Security | Authorization, validation, injection prevention, CSRF, replay controls | 37 |
| [ch05](chapters/ch05-mysql.md) | MySQL and ORM | Schema, unique indexes, EXPLAIN, SQL nulls, explicit mappings | 38-43 |
| [ch06](chapters/ch06-project-structure.md) | Project structure | Layers, domain objects, GAV, dependency mediation, server diagnostics | 44-47 |
| [ch07](chapters/ch07-design.md) | Design | Diagram thresholds, design review, SOLID-related principles, DRY | 48-50 |

## Topic Index

- **Accessibility**: ch07
- **AIR / BCDE**: ch03
- **API contracts / JSON / large IDs**: ch01, ch02, ch04
- **Authorization / CSRF / SSRF / XSS**: ch04
- **BigDecimal / money / boxed values**: ch01, ch05
- **Collections / comparator / PECS / toMap**: ch01
- **Composition / dependency inversion / DRY / Liskov**: ch07
- **Dates / formatting / leap years**: ch01
- **Dependency management / GAV / versioning**: ch06
- **DO / DTO / BO / Query / VO**: ch01, ch05, ch06
- **Error codes / Result / Throwable**: ch02, ch06
- **EXPLAIN / indexes / pagination**: ch05
- **Javadoc / naming / formatting**: ch01
- **Locks / pools / ThreadLocal / volatile**: ch01
- **Logging / retention / exceptions**: ch02, ch06
- **MyBatis / resultMap / SQL injection**: ch04, ch05
- **Null / unboxing / SQL aggregates**: ch01, ch02, ch05
- **Schema / sharding / transactions**: ch05, ch07
- **Sequence / state / activity / class / use-case diagrams**: ch07
- **Unit tests / coverage / fixtures**: ch03

## Supporting Files

- [glossary.md](glossary.md): Definitions and chapter pointers.
- [patterns.md](patterns.md): Steps and trade-offs for recurring techniques.
- [cheatsheet.md](cheatsheet.md): Decisions, thresholds, and review smells.

## Scope and Provenance

This skill synthesizes this 2020 edition in English. It is not a claim about the latest Alibaba manual, current legislation, or universal Java/MySQL best practice. Edition policies such as interface implementations, log paths, SQL foreign-key restrictions, coverage targets, and server tuning require their project context. Technical clarification is labeled in chapters rather than silently altering the source.

Source file: `D:\book\阿里巴巴Java开发规范（嵩山版）.pdf`. SHA-256: `8c2a8f7c785ab57a34a45853403f694e4dc38baf8e2a7f254acdc2d5041dd04d`.

Extraction used pdfminer text fallback because Docling was unavailable. All seven main sections, glossary, version history, and error-code appendix were read. The architecture diagram, collection null table, version row, and sample error-code table were checked visually. The other embedded images are cover/end-page artwork; QR destinations were not followed. Appendix history is represented by edition metadata; glossary terms are merged into glossary.md; selected appendix codes appear in ch02. Consult the original PDF for the complete code registry and details not indexed here.
