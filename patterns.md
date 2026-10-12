# Implementation Patterns

English recipes synthesized from the manual. M/R/F are edition levels, not production severity. Read linked chapters for scope and caveats.

## Five-Scope Constants

**When to use:** A named value must agree among multiple consumers. (R, [Ch01](chapters/ch01-programming.md), Constants rules 3-5)

**How:** Name its meaning; choose class, package, subproject, application, or cross-application scope according to actual consumers. Group by function. Put shared cross-application values in the internal client contract. Use an enum for a bounded domain with attributes.

**Trade-offs:** Wider scope increases coupling. Separate definitions of the same apparent value can diverge; a giant universal constant holder obscures ownership.

## Exact Value Comparison

**When to use:** Boxed integers, decimals, money, or approximate measurements meet comparisons. (M, [Ch01](chapters/ch01-programming.md), OOP rules 6-12)

**How:** Compare nullable objects safely; use equals for boxed integral values. Use string/valueOf decimal construction and compareTo for numeric decimal equality. Store money in integer smallest units with documented currency/range. For approximate floats, specify the allowed tolerance.

**Trade-offs:** Decimal equality and representation equality differ. A tolerance is a business/numeric choice, not a universal constant.

## Explicit Collection Conversion

**When to use:** Converting streams, arrays, views, or typed output. (M, [Ch01](chapters/ch01-programming.md), Collections rules 3-14)

**How:** State duplicate-key handling for toMap and reject/handle null mapped values. Use a typed zero-length array for toArray under this edition. Copy a view/fixed-size list into ArrayList when independent structure and expansion are required. Remove via the iterator instead of enhanced-for structural mutation.

**Trade-offs:** Copies cost memory and still share element references. A merge policy such as last-input-wins does not mean greatest-value-wins. Synchronize shared mutation coherently.

## Resource Ownership in Finally

**When to use:** ThreadLocal context, locks, or owned closeable resources. (M, [Ch01](chapters/ch01-programming.md), Concurrency rules 6/9/10; [Ch02](chapters/ch02-errors-and-logging.md), Exceptions rules 6/7)

**How:** Set thread context, run work, remove in finally. Acquire a blocking lock immediately before try and unlock in finally. For tryLock, branch on successful acquisition first. Use try-with-resources for suitable owned closeables.

**Trade-offs:** Cleanup must match ownership; never unlock without acquisition or return from finally. Do not close resources owned by a caller simply because they are reachable.

## Bounded Named Execution

**When to use:** Application work executes concurrently or periodically. (M, [Ch01](chapters/ch01-programming.md), Concurrency rules 1-5/12)

**How:** Configure ThreadPoolExecutor thread counts, bounded work storage, rejection behavior, and meaningful ThreadFactory names. Confirm singleton state and methods are safe. Use suitable scheduled execution and handle task failures explicitly.

**Trade-offs:** Limits expose overload and require a rejection policy. Timer has cross-task failure coupling; scheduled periodic tasks can still stop repeating after their own unhandled failure.

## Version-Checked Update

**When to use:** Concurrent callers may change the same record. (M/R, [Ch01](chapters/ch01-programming.md), Concurrency rules 11/13)

**How:** Protect the invariant with a suitable lock or version check. For optimistic updates, compare the expected version, advance it atomically, inspect affected-row success, and retry within justified bounds. The source recommends optimistic locking below 20% conflicts with at least three retries. Financial-sensitive work prefers lock, validate, update, release.

**Trade-offs:** Retrying increases load and must preserve operation semantics. The source threshold is a heuristic; pessimistic locks serialize work and need timeout/deadlock handling.

## Failure Translation with One Log Owner

**When to use:** A failure crosses DAO/Service, RPC, or HTTP boundaries. (M/F, [Ch02](chapters/ch02-errors-and-logging.md); [Ch06](chapters/ch06-project-structure.md))

**How:** Preserve the cause; choose transport by boundary. Reuse a registry code matching A/B/C origin and specific meaning. Include safe diagnostics and a separate user message. Assign a logging owner with enough context, use placeholders/guards, and avoid repeated printing through the call path. Verify rollback if a catch suppresses propagation.

**Trade-offs:** Translation can erase cause or falsely signal success. Narrow boundary Throwable handling in this edition requires an explicit fatal-error policy.

## AIR Suite with BCDE Cases

**When to use:** Adding/reviewing core behavior tests. (M/R, [Ch03](chapters/ch03-unit-testing.md))

**How:** Prepare each test's fixtures, inject local collaborators, assert outcomes, and prove repeatability/order independence. Derive Border, Correct, Design, and Error cases from the contract. Put unit tests in src/test/java and integration checks in the appropriate project suite.

**Trade-offs:** Coverage helps locate omissions but cannot establish assertion quality. Over-mocking implementation details reduces refactoring freedom.

## Bound Values, Allowlist Structure

**When to use:** Requests influence SQL, sorting, redirects, or server fetch targets. (M, [Ch04](chapters/ch04-security.md); [Ch05](chapters/ch05-mysql.md))

**How:** Validate input meaning/range/size; bind SQL values through driver/MyBatis parameters. Map metadata to permitted identifiers/directions instead of substituting arbitrary strings. Authorize object access, validate destinations, and apply output-context handling separately.

**Trade-offs:** Binding handles values, not dynamic SQL structure. Sanitizing input cannot substitute for authorization or correct output encoding.

## Integrity-First Schema and Typed Mapping

**When to use:** Persisting a domain or modifying an ORM contract. (M, [Ch05](chapters/ch05-mysql.md))

**How:** Identify business uniqueness and enforce it with unique indexes. Select required columns, check complete value ranges, define resultMap, map is_xxx to the non-is POJO property, and update only intended fields plus update_time. Preview data-correction targets before mutation.

**Trade-offs:** Unsigned BIGINT exceeds Long's complete domain. Prefix unique indexes can reject distinct values. Map these constraints explicitly rather than trusting implicit conversion/application checks.

## Query-Shaped Index and Delayed Association

**When to use:** Filtering/ordering or deep pagination needs optimization. (R/M, [Ch05](chapters/ch05-mysql.md))

**How:** Put equality conditions before ranges in a useful composite index; measure discrimination and match required ordering. Favor covering access. Inspect EXPLAIN and representative rows/latency. For deep pages, select narrow IDs then associate the needed fields, preserving order.

**Trade-offs:** Wider indexes cost space/writes. Narrow-ID pagination reduces row-fetch work but does not eliminate deep-offset scanning. A plan label alone is insufficient evidence.

## Reviewed Dependency Upgrade

**When to use:** Adding/upgrading a company-internal shared library. (M/R, [Ch06](chapters/ch06-project-structure.md))

**How:** Capture dependency resolution before/after. Inspect changed transitive mediation, align related version properties and identical G/A coordinates across modules, and check final packaging/runtime compatibility. Keep published releases immutable and document changes.

**Trade-offs:** Source compatibility does not prove deployed binary compatibility. Version exclusions can conceal issues when applied without inspecting the graph.

## Diagram-Driven Change Review

**When to use:** States, call depth, model dependencies, workflows, or storage cross source complexity thresholds. (M/R, [Ch07](chapters/ch07-design.md))

**How:** Choose the diagram answering the hard question. Express triggers, inputs/outputs, ownership, and abnormal flows. Review/document durable structures and identify actual variation points. Apply composition, stable abstractions, and DRY to shared meaning.

**Trade-offs:** Documents must match implemented behavior. Extra diagrams/abstractions without a decision to express create maintenance work rather than clarity.
