# Chapter 3: Unit Testing

**Source:** Printed pp. 35-36, rules 1-16. **Levels:** M = mandatory, R = recommended, F = reference. This short chapter supplies test properties and scenarios, not a prescribed testing framework.

## Core Idea

Tests must diagnose a unit's behavior automatically and reliably. Inject dependencies so a test can run alone, in any order, repeatedly, without depending on external state.

## Frameworks Introduced

### AIR — M, Rule 1

**Automatic, Independent, Repeatable.** Use AIR when deciding whether a check belongs in the unit suite.

1. **Automatic:** Assertions determine success. Execution is noninteractive; printed output requiring human inspection is insufficient.
2. **Independent:** Each test prepares its own state. Tests never call other tests or depend on their order/output.
3. **Repeatable:** Network, service, middleware availability, and existing database contents do not decide the outcome. Inject local/in-memory or mock dependencies.

Why it works: CI results become evidence about behavior rather than the environment. A test depending on a developer's existing fixture fails this criterion even if it usually passes locally.

### BCDE — R, Rule 9

**Border, Correct, Design, Error.** Enumerate cases before writing assertions.

1. **Border:** Check loop endpoints, special values/times, and ordering. Include immediately valid and invalid sides of meaningful boundaries.
2. **Correct:** Supply valid input and assert the expected result, not merely successful completion.
3. **Design:** Connect cases to documented design requirements and business rules.
4. **Error:** Deliberately submit invalid data, abnormal flows, or out-of-domain input and assert the intended failure response.

Why it works: happy-path execution misses overflow, invalid transitions, and cleanup. BCDE requires different perspectives; coverage numbers cannot replace these cases.

## Rules and Scope

**M, rules 2-7:** Assert results instead of manually inspecting System.out. Tests are independent and repeatable. Unit scope is at most a class, usually a method; cross-class/system interactions belong to integration testing. New core-business/application/module code must have passing unit tests. Put them in `src/test/java`, not the production source directory.

**R, rules 8-14:** Aim for 70% statement coverage overall and 100% statement plus branch coverage in core modules. Test DAOs, Managers, and highly reusable Services. Prepare database fixtures through programmatic insertion/import, not assumed existing rows or manual setup outside the test. Use automatic rollback or identifiable fixture prefixes/suffixes and cleanup. Refactor untestable code appropriately instead of writing brittle tests to satisfy metrics. Agree on scope with testing colleagues during design review; finish unit tests before feature testing rather than after release.

The source discusses database-related unit tests. A check requiring a real external database should follow the project's integration-test contract; keep AIR unit tests local and use isolated repeatable fixtures in the appropriate suite.

**F, rules 15-16:** Heavy constructors, global variables/static methods, many external dependencies, and complex conditionals obstruct testability. Tests are maintained developer code that helps prevent production failures, not disposable testing-team work.

## Key Concepts

- **SUT:** System under test whose observable behavior is asserted.
- **Dependency injection:** Supply collaborators externally so tests can control them.
- **Mock:** Controlled collaborator behavior isolating the SUT.
- **Fixture:** Programmatically prepared test input/state.
- **Statement coverage:** Fraction of instrumented statements executed.
- **Branch coverage:** Fraction of decision outcomes executed.
- **Integration test:** Check of interaction across components or systems.
- **Regression test:** Repeatable check preventing a known behavioral failure.

## Mental Models

- Run each test alone; suite order must add no prerequisite.
- External dependencies are seams to control, not excuses for flaky unit results.
- Coverage finds missing cases; assertions establish correctness.
- Tests are part of maintained design, not a post-release appendix.

## Reference Table

Actions adapt the source's AIR/BCDE criteria into review decisions.

| Symptom | Criterion | Correction |
|---|---|---|
| Someone must read console output | AIR Automatic | Assert results/failures |
| Another test creates its input | AIR Independent | Own setup/teardown |
| Live network/DB decides success | AIR Repeatable | Local collaborator or isolated integration suite |
| Only normal values checked | BCDE Border/Error | Boundary and invalid-input cases |
| Design rule unrepresented | BCDE Design | Case linked to requirement |
| High coverage, weak assertions | Correctness | Assert specified output/state/failure |

## Code Example

Illustrative JUnit-style adaptation; the manual does not mandate JUnit. Domain names and the limit are illustrative rather than source APIs/defaults.

```java
@Test
void rejectsBatchAboveConfiguredLimit() {
    UserLookup lookup = new UserLookup(new InMemoryUserRepository(), 2);
    List<Long> ids = Arrays.asList(1L, 2L, 3L);

    assertThrows(BatchLimitException.class, () -> lookup.listUsers(ids));
}
```

This fragment requires the project's framework/imports and example domain types. It asserts an observable contract without relying on a remote service or another test.

## Worked Example

Source scenario: a deletion test manually adds a database row that violates normal insertion rules, producing misleading results. Reconstruct a reliable check:

1. Prepare a valid entity programmatically with its required fields.
2. Invoke deletion and assert the specified postcondition: absence or logical-deletion state, according to design.
3. **Border:** Check an absent target and meaningful identifier boundary.
4. **Correct:** Delete a valid target and verify exact state.
5. **Design:** Cover physical versus logical deletion as documented.
6. **Error:** Reject a prohibited deletion and verify no unintended mutation.
7. Roll back or remove identifiable fixtures on every exit. Repeat and run alone to uncover hidden state.

These steps adapt the source fixture scenario with BCDE; actual entity and expected outcomes come from the project.

## Anti-patterns

- Tests depend on prior tests or live service availability.
- Printed output substitutes for assertions.
- Manual database preparation violates business constraints.
- Coverage goals drive fragile checks of private implementation details.
- Tests remain stale after behavior changes or appear only after release.

## Key Takeaways

1. Apply AIR to reliability and BCDE to case selection.
2. Prepare/clean fixtures through repeatable code.
3. Separate unit behavior from integration responsibilities.
4. Keep source coverage targets recommended, not mandatory.

## Connects To

- [Ch01](ch01-programming.md): Numeric, null, duplicate-key, and limit boundaries.
- [Ch02](ch02-errors-and-logging.md): Cleanup, failure classification, and rollback assertions.
- [Ch05](ch05-mysql.md): Fixtures, uniqueness, and logical deletion.
- [Ch07](ch07-design.md): Requirement documents and dependency-injection seams.
