# Chapter 4: Security Conventions

**Source:** Printed p. 37, rules 1-9. **Levels:** M = mandatory, R = recommended, F = reference. This short checklist has no source code example; the worked review adds application steps without inventing a named source framework.

## Core Idea

Treat user-controlled boundaries as untrusted. Check permission, allowed values, output handling, and replay/abuse limits where the protected action occurs.

## Techniques and Criteria

### Ownership and Sensitive Display — M, Rules 1-2

For an individual user's page, object, or action:

1. Identify caller and requested object.
2. Authorize that object, not merely the caller's login.
3. Protect every read/update/delete path against changing the ID to another user's object.
4. Mask displayed sensitive data. The source hides the middle four digits of a mainland-China phone number; choose masking appropriate to the actual data and audience.

Why it works: authentication does not prove ownership. A hidden UI link does not enforce backend permission.

### SQL and Parameter Validation — M, Rules 3-4

1. Bind SQL value parameters instead of concatenating user data into SQL.
2. Restrict metadata, such as column names/sort directions, to permitted choices. Value binding cannot validate query structure.
3. Validate every request parameter's shape, range, size, and meaning at the server boundary.
4. Check page/batch sizes, expensive sorts, cache keys, URL targets, shell/deserialization input, and regex behavior as applicable.

Source consequences include OOM from huge pages, slow ORDER BY, cache breakdown, SSRF, arbitrary redirects, SQL/shell/deserialization injection, and ReDoS. A nonempty check alone does not address these different threats.

### Output, CSRF, and Redirects — M, Rules 5-7

- Filter/escape user data correctly before HTML output. Runtime clarification: HTML text, attributes, scripts, and URLs require different context handling.
- Perform CSRF security validation for forms and AJAX under this edition; inspect the actual session/authentication model and existing framework mechanism.
- Allowlist external redirect destinations. Centralized URL generation does not itself make a caller-supplied destination safe.

### Replay and Abuse Controls — M Rule 8, R Rule 9

SMS, email, calls, ordering, and payments need correct anti-replay/resource-abuse controls. Source examples include quantity limits, frequency controls, and verification-code checks. Identify whether the threat is repetition, duplicate execution, or unauthorized action; select matching controls rather than assuming one captcha solves everything.

Posts, comments, and instant messaging should implement anti-spam, prohibited-content filtering, and risk controls. Preserve this as R despite strong language inside the recommendation.

## Key Concepts

- **Horizontal authorization:** Check the requested object's ownership scope, not just login/role.
- **Masking:** Hide sensitive displayed portions while preserving necessary usability.
- **Binding:** Send SQL data separately from executable statement structure.
- **Allowlist:** Explicit permitted values or destinations.
- **CSRF:** Unwanted browser action using an existing authenticated context.
- **SSRF:** Unintended server requests caused by a caller-controlled target.
- **ReDoS:** Disproportionate regex computation from crafted input.
- **Replay:** Repeating an action contrary to its intended contract.

## Mental Models

- A valid user can request an unauthorized object.
- Input validation constrains data; output encoding constrains interpretation.
- Limits protect resources as well as correctness.
- Enforce controls before the resource-consuming effect.

## Reference Table

| Boundary | Review question | Rule |
|---|---|---|
| Owned object | Can changing the ID bypass ownership? | M 1 |
| Sensitive display | Is raw sensitive data visible? | M 2 |
| SQL value/metadata | Bound values and restricted structural choices? | M 3-4 |
| Page/batch/upload | Enforced server size/range limit? | M 4 |
| User HTML | Correct output-context handling? | M 5 |
| Browser form/AJAX | CSRF validation enforced? | M 6 |
| External redirect | Destination allowlisted? | M 7 |
| SMS/payment/order | Replay and consumption controls effective? | M 8 |

## Worked Example

Source scenario: unrestricted registration-code SMS enables harassment and resource waste. Adapted review:

1. Trace request to actual SMS send; identify caller, target, and operation.
2. Validate request values before the platform call.
3. Check enforced frequency/count controls for the relevant identity/target, not only a disabled frontend button.
4. Check repeated-request and verification behavior against the intended workflow.
5. Mask displayed phone data; keep complete sensitive values out of user/diagnostic output.
6. Verify limit failures reject before send. A friendly error after unwanted SMS does not recover the cost.

The source supplies no universal rate or algorithm. Derive these from the project contract; do not invent an “Alibaba limit.”

## Anti-patterns

- Login-only checks expose other users' records by ID.
- SQL values are bound but arbitrary sorting/identifiers remain executable.
- Client validation substitutes for server resource/permission checks.
- One escaping function serves unrelated output contexts.
- A centralized redirect builder accepts any destination.
- SMS/payment requests consume resources repeatedly without limits.

## Key Takeaways

1. Authorize objects and validate requests at server boundaries.
2. Bind values and constrain metadata.
3. Match safe output to its context.
4. Verify rejection before repeated external effects.

## Connects To

- [Ch01](ch01-programming.md): API limits, redirects, and regular expressions.
- [Ch02](ch02-errors-and-logging.md): Safe error/log content.
- [Ch03](ch03-unit-testing.md): BCDE rejection and boundary cases.
- [Ch05](ch05-mysql.md): Bound parameters and integrity constraints.
