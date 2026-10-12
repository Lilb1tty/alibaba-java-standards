# Chapter 2: Error Codes, Exceptions, and Logging

**Source:** Printed pp. 29-34; selected error codes from Appendix 3, pp. 54-59.  
**Levels:** M = mandatory, R = recommended, F = reference.

## Core Idea

Make failure origin and handling ownership explicit. An error code, diagnostic message, stack trace, and user tip serve different readers and should remain connected without exposing sensitive information.

## Frameworks Introduced

- **Rapid tracing and standardized communication — M:** An error code answers whose failure and which failure. Use a source letter plus stable registry number; reuse an existing matching meaning instead of embedding every business detail in the code. Why it works: callers and operators can compare a small stable vocabulary while diagnostics carry the variable context.
- **Boundary-specific failure transport — F:** Outside-company HTTP/API calls require error codes in the source guidance; cross-application RPC prefers Result with success status, errorCode, and errorMessage; internal calls can propagate exceptions. Choose the boundary first, then transport, then translation owner.
- **Unstable-code isolation — M:** Separate code that can fail from stable surrounding work. Catch the actual failure type at the smallest meaningful region and handle, translate, or propagate it. Broad catches around unrelated actions hide which recovery is valid.
- **Event context plus stack — M:** Log the state needed to locate the failure and preserve the throwable. Log once at the designated handling boundary; lower layers can preserve causes while propagating. Coordinate with the layering guidance in ch06.

## Error-Code Rules

**M:** Codes are five-character strings. `00000` is success when a code field is necessary. Failure codes use A/B/C followed by four digits in 0001-9999: user-origin, current-system-origin, or third-party-origin. Reserve category space in increments of 100. Do not encode product version or error severity. Allocate through a central approved registry independently of company/business architecture; approved numbers are permanent. Reuse the published registry where meaning matches. Do not display a bare code as the user message.

**R:** Put unique business context in errorMessage. A system can translate an upstream C failure into a B failure when its own outward semantics require that; retain the upstream code in the diagnostic message. This does not justify concealing the cause or reporting a false success.

**F:** Use macro codes when a finer cause is unknown; do not invent precision. The last three digits are unrelated to HTTP status. Layered category codes are a navigation scheme, not an automatic severity score.

## Reference Tables

These are selected source registry entries, not the complete appendix.

| Code | Meaning / decision |
|---|---|
| `00000` | Successful execution |
| `A0001` | User-origin failure, cause not yet more specific |
| `A0100` | Registration failure category |
| `A0111` | Username already exists |
| `A0200` | Login failure category |
| `A0230` | Login expired |
| `A0300` | Access-permission failure category |
| `A0301` | Unauthorized access |
| `A0400` | Request-parameter error category |
| `A0410` | Required request parameter absent |
| `A0420` | Parameter outside allowed range |
| `A0426` | Batch item count exceeds limit |
| `A0427` | Request JSON parsing failed |
| `A0501` | Request frequency exceeds limit |
| `A0506` | Duplicate user request |
| `B0001` | Current system execution failed |
| `B0100` | Current system execution timed out |
| `B0210` | Current system throttling |
| `B0315` | Current system thread pool exhausted |
| `C0001` | Third-party call failed |
| `C0110` | RPC service failure |
| `C0210` | RPC execution timed out |
| `C0250` | Database-service timeout |
| `C0321` | Ambiguous same-name columns in a multi-table query |
| `C0331` | Database deadlock |
| `C0341` | Primary-key conflict |

When borrowing these codes, match actual registry semantics. “Any input issue” does not imply the username-specific code; “any SQL failure” does not imply deadlock.

## Exception Rules and Runtime Notes

**M:** Guard foreseeable NPE and bounds failures instead of catching them as normal flow. Parsing malformed numeric text is an explicit example where NumberFormatException can be appropriate. Catch relevant unstable operations by type. Handle a caught failure or propagate it; the outer business consumer must turn it into understandable output. If a catch suppresses an exception that must roll back a transaction, arrange rollback explicitly through the actual transaction mechanism. Do not assume an annotation will roll back after a swallowed exception.

**M:** Close owned resources even on failures; Java 7+ try-with-resources is a suitable mechanism. Do not return in finally: it can replace a pending result or suppress a failure. Match catches to thrown types or supertypes.

**M, edition-specific:** Catch Throwable at RPC, internal-library, or dynamically generated invocation boundaries. The source motivates this with runtime linkage failures such as NoSuchMethodError, which are not Exception subclasses. **Runtime clarification:** this is a narrow boundary policy, not a general catch-everything pattern. Fatal VM errors must not be converted into ordinary success or blindly retried. Review process/runtime recovery policy before implementing it; preserve the cause and make any project exception explicit.

**R:** Internal methods may return null if the contract documents when. Callers remain responsible for null checks after RPC, database, session, nested-property access, or wrapper unboxing. This coexists with ch01's empty-container rule for frontend data lists. Define meaningful checked/unchecked business exceptions; avoid generic new RuntimeException, Exception, or Throwable as the designed domain type.

## Logging Rules

**M:** Depend on a facade such as SLF4J/JCL, preferably SLF4J, rather than implementation APIs. Use placeholders. Check enabled levels before trace/debug/info output, especially before expensive argument computation. Disable undesired additivity where appenders would duplicate entries. Do not use System.out/System.err or printStackTrace for production logs. Include safe event context and the exception object. Do not JSON-serialize arbitrary objects for logging: getters can fail or do expensive work. Select relevant safe fields or a reviewed toString.

**M:** The edition keeps ordinary logs for at least 15 days. Its network-operation, security-event, and personal-sensitive-operation records are retained for at least six months with backups across machines. These are source requirements, not a current legal opinion; apply the actual organization's retention/access/privacy policy. Its deployment convention stores today's `<application>.log` beneath `/home/admin/<application>/logs/` and historical files as `<logname>.log.yyyy-MM-dd`. Extension logs follow `appName_logType_logName.log`, such as monitor/access/stats.

**R:** Avoid production debug output; select useful info and control temporary warn volume. Use warn for ordinary invalid user parameters instead of producing repeated error alarms. Use error for actual system defects or important failures. Prefer clear English diagnostic messages and keep sensitive input out of both logs and frontend errors.

## Key Concepts

- **Error origin:** User, current system, or external service responsible for the failure.
- **Registry:** Stable approved meaning of each code.
- **User tip:** Safe actionable explanation for the end user.
- **Cause chain:** Original failure retained when wrapping exceptions.
- **Result envelope:** Cross-boundary success/failure data contract.
- **Logging facade:** API independent of the selected log implementation.
- **Additivity:** Propagation to parent appenders that can duplicate an event.
- **Linkage failure:** Runtime mismatch between compiled code and available methods/classes.

## Mental Models

- Choose transport by boundary, not by a universal preference for exceptions or Result.
- Think of a code as stable classification and a message as variable evidence.
- Treat log ownership like resource ownership: assign the owner and avoid duplicate work.
- A swallowed exception changes transaction control flow; check its rollback consequence.

## Code Examples

Adaptation of Logging rules 5, 6, and 9. Logger is an SLF4J facade; values and operation are surrounding application context.

```java
if (logger.isDebugEnabled()) {
    logger.debug("Processing trade id={} symbol={}", tradeId, symbol);
}

try {
    processTrade();
} catch (TradeException exception) {
    logger.error("Trade processing failed id={}", tradeId, exception);
    throw exception;
}
```

The example assumes this boundary owns logging and upstream layers do not log it again. If a higher layer owns the event, propagate the exception here without logging. Do not replace tradeId with complete unredacted user/transaction payloads.

## Worked Example

The source contrasts a broad registration catch with cause-specific handling. Adapted review:

1. Validate username format before service execution. Invalid format is not an NPE to catch.
2. If the username already exists, use registered `A0111`; return a safe message such as “Choose another username.” Do not invent a product-specific number.
3. If the registration service itself fails unexpectedly, classify that current-system failure rather than returning a user-validation code.
4. Return HTTP status plus errorCode, errorMessage, and user message at the HTTP boundary. Keep stack/context in the authorized diagnostic channel.
5. Ensure the failure path releases owned resources and rolls back work when necessary. Check that only the designated layer logs the event.

The steps synthesize the source scenario; the wording and transport shape are English adaptations rather than copied artifacts.

## Anti-patterns

- Empty catch blocks produce apparent success without recovery.
- Exception-based null checks obscure normal input validation.
- Returning in finally replaces the business result or exception.
- Every layer prints the same stack and exhausts log storage.
- Error codes encode department/product versions and lose stable meaning.
- Bare error codes or raw traces become user messages.
- An unsafe getter invoked by JSON logging breaks the business path.

## Key Takeaways

1. Preserve failure meaning and causes across translation.
2. Handle errors at the proper boundary and verify transaction behavior after catches.
3. Log actionable safe context once, with the throwable.
4. Match existing error-code meanings rather than guessing.
5. Treat retention and Throwable handling as scoped edition rules requiring runtime context.

## Connects To

- [Ch01](ch01-programming.md): API errors, unboxing, cleanup, and log argument evaluation.
- [Ch03](ch03-unit-testing.md): Repeatable failure/rollback tests.
- [Ch04](ch04-security.md): Sensitive output and abuse controls.
- [Ch06](ch06-project-structure.md): Layer-specific translation and logging responsibility.
