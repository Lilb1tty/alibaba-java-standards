# Coding and Review Decisions

M = mandatory, R = recommended, F = reference in Songshan 1.7.0. Source policies are not universal runtime facts.

| When you see… | Decide… | Reason / chapter |
|---|---|---|
| Boxed integer `==` | M: safe equals comparison | Cache-dependent identity, ch01 |
| Decimal value equality | M: compareTo == 0; string/valueOf construction | Scale and binary-input precision, ch01 |
| Stored money | M: integer smallest unit; define currency/range | Avoid rounding ambiguity, ch01/ch05 |
| Nullable ternary/RPC result | Check type alignment before unboxing | Missing is not zero, ch01/ch02 |
| Mutable list expected | Distinguish view/fixed-size/immutable/copy | Mutation contract, ch01 |
| Stream toMap | M: explicit duplicate policy; no null mapped values | Duplicate/NPE failures, ch01 |
| Enhanced-for removal | M: iterator removal; coordinate shared access | Invalid iteration/concurrency, ch01 |
| New pool | M: explicit ThreadPoolExecutor, names, capacity/rejection | Resource exhaustion, ch01 |
| ThreadLocal request context | M: remove in finally | Reused-thread contamination, ch01 |
| Lock acquisition | M: lock immediately before try; unlock in finally | Match ownership, ch01 |
| tryLock | Enter/unlock only after success | No unlock without ownership, ch01 |
| volatile count++ | Use atomic/lock invariant | Visibility is not atomicity, ch01 |
| Catch in transaction | Verify required rollback after catch | Swallowed exception changes control flow, ch02 |
| Failure crosses boundary | Classify, preserve cause, assign one log owner | Meaning and diagnostics, ch02/ch06 |
| Error response | M: HTTP status, errorCode, errorMessage, user message | Distinct audiences, ch01/ch02 |
| User-owned ID | M: authorize that object | Login is insufficient, ch04 |
| SQL user value / sort name | M: bind value / allowlist metadata | Data versus executable structure, ch04/ch05 |
| Browser form/AJAX | M: enforce CSRF validation | Existing-session abuse, ch04 |
| Redirect/SMS/payment | Allowlist destination / enforce replay and consumption controls | Boundary/resource safety, ch04 |
| Business uniqueness | M: database unique index | Check-then-insert races, ch05 |
| Boolean column | M: is_xxx with explicit mapping to xxx | POJO convention differs, ch05 |
| Deep offset | R: narrow IDs then fetch; inspect residual scan | Reduce wider-row work, ch05 |
| Dependency upgrade | Compare effective graph and packaged jars | Runtime compatibility, ch06 |
| Shared behavioral copies | R: one definition when meaning matches | DRY, ch07 |

## Quick Choice Rules

- **Failure transport, F:** External HTTP uses error codes; cross-app RPC prefers Result; internal calls can throw. Do not promote this boundary guide into “all methods return Result.”
- **Tests:** AIR = Automatic/Independent/Repeatable (M). BCDE = Border/Correct/Design/Error (R). Check observable outcomes, not printed output.
- **Locks:** Source recommends optimistic below 20% conflicts, at least three retries; financial-sensitive updates prefer pessimistic (R). Validate measured contention and retry semantics.
- **Null containers:** Frontend data lists use empty containers (M); documented internal Java methods may return null (R). Select by boundary.
- **Logging:** Use facade, placeholders, level guards, safe context, throwable, and one owner. Broad Throwable is the edition's narrow invocation-boundary policy; consult ch02 before applying it.

## Source Thresholds and Defaults

| Source rule | Value | Level / chapter |
|---|---|---|
| Indent / line width | 4 spaces / at most 120 characters | M, ch01 |
| Method length | At most 80 lines excluding comments | R, ch01 |
| If/else depth | At most 3 levels | R, ch01 |
| URL parameters | At most 2048 bytes | M edition limit, ch01 |
| Ordinary / specified security-sensitive logs | At least 15 days / six months | M edition retention; current policy matters, ch02 |
| Statement coverage | 70% general; core 100% statement + branch | R, ch03 |
| Joins / IN elements | At most 3 tables / within 1000 elements | M / R, ch05 |
| VARCHAR | At most 5000; larger text separate | M edition policy, ch05 |
| Sharding recommendation | Beyond 5 million rows or 2 GB | R heuristic, ch05 |
| Internal-library initial version | 1.0.0 | M, ch06 |

## Diagram Selection — M, ch07

| Complexity condition | Choose |
|---|---|
| Actor types >1 **and** use cases >5 | Use-case diagram |
| States >3 | State diagram with triggers |
| Call-chain objects >3 | Sequence diagram with inputs/outputs |
| Model classes >5 **and** complex dependencies | Class diagram |
| Collaborating objects >2 **and** complex workflow | Activity diagram |

## Fast Smells

- A field default is about to overwrite absent update data.
- A prefix unique index rejects distinct full business keys.
- One compiled dependency version becomes another in deployment.
- A failure path logs/returns success without recovery.
- Historical fin_timeout tuning is presented as a TIME_WAIT fix; inspect Linux semantics first.
- A mandatory edition convention is reported as a severe bug without a concrete failure path.
