# Chapter 1: Programming Conventions

**Source:** Programming conventions, printed pp. 1-28. Subsections: Naming 1-3; Constants 4; Formatting 5-6; OOP 7-10; Date/time 11; Collections 12-16; Concurrency 17-20; Control flow 20-23; Comments 24; Frontend/backend contracts 25-26; Other 27-28.  
**Levels:** M = mandatory, R = recommended, F = reference. Compact English synthesis; examples are source adaptations.

## Core Idea

Make meaning, ownership, and limits visible in the code. Prevent failures caused by implicit numeric conversions, mutable views, shared state, and incomplete boundary contracts.

## Frameworks and Review Procedure

- **Five constant reuse scopes — R:** Cross-application, application, subproject, package, class. Choose the smallest shared scope that reaches every legitimate consumer; put cross-application constants in an internal client library. Group constants by function instead of using one giant holder. Different definitions of `YES` can silently break interoperability.
- **PECS — M:** Producer Extends, Consumer Super. Use `? extends T` for typed reads and `? super T` for writes of `T`. Runtime clarification: the latter still supports reads as `Object`; neither wildcard means literally every `add`/`get` method disappears.
- **Lock scope — M:** Prefer suitable lock-free structures, then a small locked block over a whole method, then an object lock over a class lock. Establish a common order for multiple resources. Failure mode: a correct lock used around slow RPC creates contention; inconsistent resource order creates deadlock.
- **Financial update sequence — R:** Lock, validate, update, release. Validate after acquiring the lock, while the protected state is stable. Prefer pessimistic locking for financial-sensitive updates under this edition.
- **Guard clauses — R:** Express exceptional branches with early return/throw, then show the main path. Keep if/else nesting to at most three levels; consider strategy/state patterns for real variation.

## Decision-Ready Rules

### Naming and Constants

**M:** Use English words rather than Chinese identifiers or mixed pinyin/English, with established international names allowed. Avoid discriminatory terminology in code/comments. Names must not start/end with `_` or `$`. Classes use UpperCamelCase, preserving established suffixes such as `DO`, `DTO`, and `VO`; methods/variables use lowerCamelCase. Constants use complete UPPER_SNAKE_CASE names. Packages are lowercase, singular, and one meaningful English word per segment. Avoid arbitrary abbreviations and confusing shadowed names across parent/child classes or sibling blocks; setter/getter parameters can match fields.

**M:** Use `Abstract`/`Base` for abstract class prefixes, `Exception` for exception suffixes, and `<Target>Test` for tests. Write `int[] values`, not `int values[]`. Boolean POJO fields have no `is` prefix. For exposed Service/DAO contracts under this edition's SOA convention, use interfaces and an `Impl` implementation suffix. Use named constants instead of magic values and uppercase `L` for long literals.

**R:** Use full descriptive word combinations, put type nouns last (`nameList`, `startTime`), and show a design pattern in names when it clarifies actual intent. Capability interfaces can use `-able` names. Omit redundant interface method modifiers and document contracts. Use enums for a fixed domain with additional attributes.

**F:** Enum class names end in `Enum`; members use uppercase underscores. Service/DAO method prefixes: `get` one, `list` many, `count` aggregate, `save`/`insert`, `remove`/`delete`, `update`. Use DO for table data, DTO for transfer, VO for display; POJO is a category rather than a `xxxPOJO` class name.

### Formatting and Comments

**M:** Four spaces, no tab characters; UTF-8 files with Unix line endings. Opening braces stay on the preceding nonempty statement line; closing braces go on their own line except continuation such as `} else {`. Empty blocks can be `{}`. Use spaces around binary/ternary operators, after argument commas, between control keywords and parentheses, and before nonempty opening braces. Keep parentheses tight and casts such as `(int)value` tight. Put one space after `//`. Keep lines at most 120 characters; continuation indent is four spaces, operators and method-call dots start the continuation, and parameter wrapping occurs after commas, not before opening parentheses.

**R:** Methods should not exceed 80 lines excluding comments but including signature, braces, blank lines, and invisible lines. Separate different logic with one blank line; avoid cosmetic alignment of assignment operators. Group overloaded constructors/methods together before applying public/protected, private, getter/setter order.

**M:** Javadoc for classes, fields, and methods; abstract/interface methods document purpose, parameters, result, exceptions, and implementer/caller obligations. Add class creator and creation date (`yyyy/MM/dd`); never invent an author identity. Document enum members. Place method-line comments above statements; align multiline comments with code. **R/F:** Update comments with behavior, remove dead declarations, explain retained commented code, and keep TODO/FIXME owner/date and intended action visible. Use clear English in this skill; the source's comment-language recommendation prioritizes clarity over poor English.

### Objects, Nullability, and Precision

**M:** Access static members through the class. Add `@Override`. Varargs must share type and meaning, appear last, and avoid `Object`. Do not change consumed public/internal-library method signatures; deprecate with the replacement documented. Avoid deprecated APIs in new callers.

**M:** Call `equals` on a known nonnull value or use `Objects.equals`. Boxed integral value comparisons use `equals`, not `==`, regardless of the `Integer` cache range. POJO properties and RPC parameters/results use wrappers; local variables preferably use primitives (**R**). Check every unboxing point. DO field types match columns; `bigint` cannot safely become `Integer`. An unsigned BIGINT can exceed Java's signed Long range: verify the supported domain and mapping.

**M:** Store money as integers in the smallest currency unit. Float/double equality requires an explicit error tolerance or exact decimal representation rather than direct `==`/wrapper equality. Use `BigDecimal.compareTo` for numeric equality and string construction or `valueOf` for decimal inputs. `equals` also compares scale; `new BigDecimal(double)` preserves binary approximation.

**M:** No implicit POJO property defaults; no business logic in constructors. Put initialization logic in `init`. Provide POJO `toString`, including inherited state when appropriate, while respecting sensitive-data handling. Do not expose both `isXxx` and `getXxx` for one property. Additive serializable changes keep `serialVersionUID`; incompatible changes need a deliberate version change.

**R:** Check split-array bounds; default `String.split` drops trailing empty fields. Keep getters/setters free of business logic. Append repeated loop text with a reused `StringBuilder`. Use `final` for deliberate immutability; choose the smallest visibility. Treat `clone` and bean-copy utilities as shallow unless deep copying is explicitly implemented. **M:** Avoid Apache BeanUtils property copying under this edition; alternatives still need semantic review.

### Date and Time

**M:** Calendar years use lowercase `yyyy`, not week-year `YYYY`. Distinguish month `M` from minute `m`, 24-hour `H` from 12-hour `h`. Use `System.currentTimeMillis` rather than allocating a Date for epoch milliseconds. Do not use `java.sql.Date`, `Time`, or `Timestamp` in application code under this edition. Do not hardcode a year as 365 days; use calendar-aware operations. **R:** Handle February 29 anniversaries explicitly and use named month constants with legacy zero-based calendars.

**M:** Shared `SimpleDateFormat` needs synchronization or confinement. Prefer immutable `DateTimeFormatter` on Java 8+. Runtime note: week-year behavior depends on calendar/locale settings; the stable rule is to choose the intended year field. `System.nanoTime` serves elapsed durations, not wall-clock timestamps. Time-zone conversions need explicit semantics.

### Collections

**M:** Custom value equality used in sets/maps needs consistent `equals` and `hashCode`; overriding one requires the other. Use `isEmpty`. `Collectors.toMap` must specify a merge policy and must not produce null values. `subList` is a view, not an ArrayList: no cast, and parent structural mutation can invalidate it. Map key/value/entry views do not support adding new mappings. `emptyList`/`singletonList` are immutable; `Arrays.asList` is fixed-size and array-backed, so no add/remove/clear, although `set` is supported. Check null before `addAll`. Convert with a zero-length typed array. Avoid raw-to-generic assignments; when legacy raw data is unavoidable, verify element types before use.

**M:** Do not add/remove from a collection inside enhanced-for iteration. Use iterator removal; concurrent access also needs a coherent synchronization policy. A comparator must satisfy sign antisymmetry, transitivity, and equal elements' consistent comparison to a third element. Returning only 1 or -1 violates equality.

**R:** Specify initial capacity. For HashMap, the source estimates `(expectedCount / 0.75) + 1`, default 16 when unknown; actual capacity rounding/allocation remains implementation-specific. Traverse entries directly or use `Map.forEach` when both key/value are needed. Check null support per implementation.

**F:** Decide whether order must be stable and/or sorted; HashMap supplies neither guarantee, ArrayList preserves order, TreeSet sorts. Use Set for membership/deduplication rather than repeated List scans.

### Concurrency

**M:** Singleton initialization and singleton methods must be thread-safe. Give pool threads useful names. Supply application execution through thread pools, explicitly configured with `ThreadPoolExecutor`; `Executors` fixed/single pools can accumulate unbounded queued work, cached pools can create excessive threads. A ThreadFactory creating worker threads is consistent with this rule; arbitrary request code creating unmanaged threads is not.

**M:** Remove custom `ThreadLocal` state in `finally`. Acquire a blocking lock immediately before `try`; do not put potentially throwing work between acquisition and `try`. For `tryLock`, check acquisition first and unlock only when held. Protect concurrent updates with application/cache locking or database version checks. The source explanation recommends optimistic locking below 20% conflicts and at least three retries; use bounded, justified retries. **R:** Financial-sensitive data prefers pessimistic locking. **M:** Prefer scheduled-executor handling to Timer's single-thread failure coupling. Runtime note: a failed periodic scheduled task can still suppress its own future executions; handle failures deliberately.

**R:** Put CountDownLatch `countDown` on every worker exit, normally in `finally`; worker exceptions do not propagate into the parent's try/catch. Prefer `ThreadLocalRandom` to a contended shared Random. Double-checked initialization requires a `volatile` field on Java 5+. **F:** `volatile` fixes visibility, not `count++` atomicity; use AtomicInteger/AtomicLong or LongAdder according to semantics. HashMap is not a concurrent map; the source's resize-cycle warning is historically version-specific. ThreadLocal confines references, not a shared object's mutations.

### Control Flow and Input Contracts

**M:** Traditional switch includes a final default and explicit termination or documented fall-through per case. Check nullable external String selectors first. Always use braces in if/else/for/while/do. Inspect ternary type alignment for null unboxing. For concurrent exit thresholds, avoid equality-only checks that can be skipped; fix synchronization too, since a `<= 0` check alone does not make a decrement safe.

**R:** Add a separating blank line after terminating branches in methods longer than ten lines. Name complex predicates, keep assignments out of other expressions, prefer direct positive comparisons, and move avoidable repeated allocations/DB connection acquisition out of loops. Protect public and especially batch interfaces. **F:** Revalidation inside trusted private/hot internal methods can be omitted only when a verified upstream contract guarantees valid input; this does not exempt user boundaries from ch04 validation.

### Frontend/Backend and Templates

**M:** Define protocol, domain, path, method, input, status, and response. Use HTTPS in production. Paths identify nouns, preferably plural, use lowercase/underscore separators, and omit `.json`/`.xml` suffixes; negotiate via headers. GET retrieves, POST creates, PUT updates, DELETE removes. No sensitive URL parameters; set body Content-Type. JSON keys use descriptive lowerCamelCase. Empty data-list responses use empty containers. Error responses carry HTTP status, `errorCode`, `errorMessage`, and a separate useful user message without sensitive data.

**M:** Return very large integer IDs as String for JavaScript clients; Number cannot represent every integer above its safe range. URL parameters are limited to 2048 bytes by this edition; bound request-body length against actual configured servers. Clamp frontend page values below one to the first page; backend requests above the last page return the last page. Internal forwarding uses `forward`; external destinations use the centralized URL builder and security allowlist. **R:** Declare caching policy, prefer JSON, and agree on `yyyy-MM-dd HH:mm:ss` in GMT. **F:** Version via headers rather than path under the source convention.

**M:** Precompile reused regular expressions. With Velocity, use property access and `$!{var}` to suppress unresolved placeholders. Use integer random APIs instead of scaling/truncating `Math.random`. **R:** Keep view templates simple; bound initialized data structures and remove obsolete code/configuration.

## Key Concepts

- **Value equality:** Compare business values independently of object identity.
- **Scale:** BigDecimal's decimal-place representation, considered by `equals`.
- **View:** A collection interface sharing backing storage.
- **Atomicity:** A compound operation appears indivisible to competing threads.
- **Visibility:** A thread can observe another thread's published writes.
- **Ownership:** Responsibility for cleanup and mutation of a resource.
- **Week-year:** Year assigned to a week, potentially different from calendar year.
- **Boundary contract:** Explicit permitted input, output, units, and failure behavior.

## Mental Models

- Treat boxed values as nullable data, not primitive defaults or object identities.
- Treat a collection result as a mutation contract before choosing its concrete class.
- Think of each acquired lock or request context as an obligation tied to an exit path.
- Distinguish visibility, atomicity, and resource bounds when reviewing concurrent work.

## Reference Tables

Source table: Collections rule 19, printed p. 16. TreeMap row assumes natural ordering; a custom comparator can change null-key behavior.

| Map | Null keys | Null values | Thread safety |
|---|---|---|---|
| Hashtable | No | No | Synchronized operations |
| ConcurrentHashMap | No | No | Concurrent operations |
| TreeMap, natural ordering | No | Yes | No |
| HashMap | Yes | Yes | No |

| Container/result | Structural add/remove | Shares source storage? |
|---|---|---|
| `Arrays.asList(array)` | No | Yes, backing array |
| `list.subList(a, b)` | Via view, subject to parent validity | Yes |
| `Collections.emptyList()` / `singletonList` | No | Immutable result |
| `new ArrayList<>(view)` | Yes | Independent structure; element references still shared |

## Code Examples

Source adaptations of OOP rules 9/10/12, Collections rule 14, Concurrency rules 6/9.

```java
BigDecimal first = new BigDecimal("1.0").subtract(new BigDecimal("0.9"));
BigDecimal second = new BigDecimal("0.9").subtract(new BigDecimal("0.8"));
boolean sameAmount = first.compareTo(second) == 0;

Iterator<String> iterator = values.iterator();
while (iterator.hasNext()) {
    String item = iterator.next();
    if ("obsolete".equals(item)) {
        iterator.remove();
    }
}

userContext.set(userInfo);
try {
    processRequest();
} finally {
    userContext.remove();
}

lock.lock();
try {
    updateProtectedState();
} finally {
    lock.unlock();
}
```

These are fragments requiring the surrounding variables/imports. They demonstrate decimal equality, iterator ownership, and cleanup on exceptional exits; they do not claim a complete runnable application.

## Worked Example

The source's duplicate version-key scenario adds three values for `version`: 12.10, 12.19, and 6.28. A plain `toMap` fails on the duplicate key. The adapted implementation states its business policy explicitly:

```java
Map<String, Double> versions = pairs.stream()
    .collect(Collectors.toMap(Pair::getKey, Pair::getValue,
        (previousValue, newValue) -> newValue));
```

The result retains 6.28. This is “latest input wins,” not the numerically greatest version. Decide that policy before choosing the merger. A null mapped value still fails, so validate or filter it according to the boundary contract. Pair/pairs are placeholders for the source's data type and input list.

## Anti-patterns

- Boxed `==` passes small-value tests and fails outside cached objects.
- Implicit POJO defaults overwrite fields absent from a partial update.
- Mutable views or fixed-size lists are treated as independent expandable lists.
- Unbounded pools hide overload until memory is exhausted.
- A pooled thread retains the previous request's ThreadLocal context.
- Lock release runs without acquisition, or an exception escapes before `try` begins.
- Logger arguments perform costly work before the disabled level can reject output.
- An integer transaction ID becomes a rounded JavaScript Number.

## Key Takeaways

1. Write explicit value, null, unit, and mutation contracts at boundaries.
2. Bound execution resources and clean up on every owned exit path.
3. Make concurrency invariants atomic; visibility and threshold checks are insufficient alone.
4. Preserve published signatures and use edition conventions consistently.
5. Check platform-specific behavior before treating historical explanations as guarantees.

## Connects To

- [Ch02](ch02-errors-and-logging.md): Cleanup, null returns, failure translation, and log arguments.
- [Ch03](ch03-unit-testing.md): Numeric boundaries, duplicates, nulls, and race/failure cases.
- [Ch04](ch04-security.md): Trust boundaries, user validation, redirects, and safe output.
- [Ch05](ch05-mysql.md): Unit/type alignment and `is_xxx` mappings.
- [Ch06](ch06-project-structure.md): Domain names and compatible library interfaces.
