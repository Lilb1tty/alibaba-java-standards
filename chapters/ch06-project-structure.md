# Chapter 6: Project Structure and Dependencies

**Source:** Printed pp. 44-47: Application layers 44; Internal libraries 45-46; Servers 46-47. Terms also drawn from Appendix 2, p. 53. The layer diagram on printed p. 44 was inspected visually. **Levels:** M = mandatory, R = recommended, F = reference.

## Core Idea

Assign each layer a clear responsibility and keep shared-library behavior reproducible. Dependency upgrades are behavioral changes whose transitive resolution and runtime compatibility must be examined.

## Frameworks Introduced

### Application Layering — R, Layers Rule 1

Use when locating behavior, identifying reuse, and tracing failure ownership. Upper layers depend on lower layers; the source does not require an empty intermediary for every call. The open API layer may reach Web or Service, and Service can use DAO directly or through a useful Manager.

| Layer | Responsibility |
|---|---|
| Terminal display | Render templates/client views |
| Open API | Expose RPC/HTTP service interfaces and gateway control |
| Web | Access routing/control, basic parameter checks, small nonreusable handling |
| Service | Concrete business logic |
| Manager | Adapt third-party services, share caching/middleware logic, combine reusable DAO work |
| DAO | Data access to persistent stores |
| Third-party service | Remote organizational/external service dependency |
| External data interface | External storage interface, often for migration |

How: trace the requested action, place business decisions in Service, factor actual shared/adapter work into Manager, and keep data access in DAO. Why it works: responsibilities reveal where to change behavior and handle failures. A Manager that merely delegates every operation creates an extra hop without the source's intended shared responsibility.

### Layered Failure Handling — F, Layers Rule 2

1. DAO wraps relevant failures with cause in DAOException; avoid logging there when an upper colocated handling layer owns the event.
2. A colocated Manager follows the same no-duplicate-log guidance; a separately deployed Manager follows Service-style boundary logging.
3. Service records the useful safe context and stack at the designated failure-handling boundary.
4. Web converts failures to a friendly response/page instead of letting them escape to the user.
5. Open APIs return error codes/messages with appropriate safe semantics.

The source proposes `catch (Exception e)` at DAO because of varied lower-level failures. This does not supersede ch02's targeted handling or make every catch recoverable. Coordinate logging once per handling boundary and preserve the original cause.

### Minimal, Traceable Internal Libraries — F, Dependencies Rule 12

Keep only needed service APIs, domain models, utilities, constants, and enums; remove unnecessary APIs/dependencies. Use a logging facade rather than bundling its implementation. Where appropriate, let consumers choose provided dependencies. Record changes, maintainer, and source location. Behavior should not change unless a consumer upgrades explicitly.

## Domain Model Reference

Layers rule 3 and Appendix 2 define these terms. Keep data purpose distinct from persistence-framework convenience.

| Type | Source meaning |
|---|---|
| DO | Data Object matching a table, transported upward through DAO |
| DTO | Data Transfer Object exported from Service/Manager |
| BO | Business Object containing business-level semantics |
| Query | Typed query request between layers; source says wrap queries with more than two parameters rather than Map |
| VO | View Object passed toward display/template rendering |
| AO | Application Object for reusable code close to Service business behavior |
| POJO | In this manual, simple getter/setter/toString object category including DO/DTO/BO/VO |

DO and DTO are not automatically interchangeable. Exposing persistence fields as an external contract can couple API consumers to schema changes.

## Library and Maven Rules

**M, rules 1-2:** GAV uses GroupId `com.<company-or-BU>.<business>[.<subbusiness>]`, at most four segments, and ArtifactId `<product>-<module>` without duplicate/missing meaning. Internal-library versions use major.minor.revision, beginning at 1.0.0. Major signals direction/large API or architecture incompatibility; minor adds significant features with relative compatibility (the source permits small API incompatibility); revision remains fully compatible with fixes/minor additions. This is the edition's scheme, not a promise of strict semantic versioning.

**M, rule 3:** Production does not depend on SNAPSHOT except the source's security-package exception. Published RELEASE versions are continuous and cannot be overwritten. Check the central repository before publication; mutable artifacts destroy reproducibility.

**M, rules 4-7:** For an internal-library addition/upgrade, keep unrelated jar mediation unchanged or explicitly assess/test differences. Do not return enums or enum-containing POJOs from internal-library APIs under this edition; enum parameters are allowed. A related library group shares one version property. Do not give the same GroupId/ArtifactId different versions in subprojects: the merged artifact contains one selected version even if local modules seemed correct.

**R, rules 8-11:** Be cautious with third-party implementations in foundational/core/data/hardware-adjacent systems. Put actual dependency declarations in dependencies and version mediation in dependencyManagement. The latter does not introduce a dependency. Minimize internal-library configuration, prefer stable compatible utilities, and inspect third-party scope/compatibility rather than adding a utility for convenience.

## Commands and Example

Commands from the source dependency-resolution guidance, adapted to Maven CLI. Run in the relevant project using its Maven/wrapper configuration.

```text
mvn dependency:resolve
mvn dependency:tree
```

Capture resolution before and after an upgrade, inspect changed transitive nodes, and use explicit exclusions/version management only after understanding the difference. The plugin commands inspect/resolve dependencies; they do not themselves prove runtime compatibility.

Illustrative source-pattern adaptation, using the project's actual version property rather than a made-up latest version:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

A child needing spring-core must still declare it under dependencies. Use the same property for related Spring artifacts and preserve the project's selected version/scope.

## Server Guidance and Runtime Caveats

**R:** Enable `-XX:+HeapDumpOnOutOfMemoryError` for diagnosable OOM and use equal Xms/Xmx in the source's production tuning context. Review current collector/container memory behavior and dump storage/privacy before selecting actual values. Raise available file descriptors when measured connection needs justify it; there is no universal safe multiplier.

**R, source claim:** The manual recommends reducing TCP TIME_WAIT and illustrates `net.ipv4.tcp_fin_timeout = 30`. **Runtime correction:** on Linux this setting concerns orphaned FIN_WAIT_2, not a general TIME_WAIT timer. Preserve the intent to investigate connection-resource pressure, but do not apply that setting as a TIME_WAIT fix without platform evidence. The quoted 240-second and 1024-descriptor defaults in the 2020 source are not universal current defaults.

**F:** Internal redirects use forward and external redirects use a unified URL Broker; ch01 contains the stronger API-level requirement, and ch04 adds destination allowlisting.

## Key Concepts

- **First-party library:** Module dependency inside the current project.
- **Second-party library:** Company-internal published library used by other applications.
- **Third-party library:** Library published outside the company, described as open source in the glossary.
- **GAV:** GroupId, ArtifactId, Version coordinates uniquely identifying a Maven artifact.
- **Mediation:** Selection of the effective version among dependency paths.
- **Release immutability:** An existing release coordinate retains its original artifact.
- **File descriptor:** OS resource handle used for files/sockets.
- **Heap dump:** Memory snapshot used to investigate allocation/retention failures.

## Mental Models

- A shared jar is a public contract to its consumers, even when it stays inside the company.
- Compilation validates one resolved graph; runtime deployment must use a compatible graph.
- Version-management entries choose versions; dependency entries introduce libraries.
- Put failures and logs where the layer has enough context to act.

## Worked Example

The source warns that modules with different versions of the same jar can each work locally but fail after packaging. Adapted upgrade review:

1. Record the current effective graph for the affected modules.
2. Change the intended library/version property.
3. Compare resolved jars, not only the edited pom line. Identify every unrelated mediation change.
4. Inspect dependency:tree for paths that caused the change; deliberately manage/exclude only confirmed conflicts.
5. Align the same G/A across subprojects and related artifacts through shared properties.
6. Check the final packaged dependency set and exercise compatibility-sensitive behavior. A runtime NoSuchMethodError is consistent with a compiled/runtime API mismatch.
7. Document remaining intentional changes and their validation; do not overwrite an already released library coordinate.

## Anti-patterns

- A pass-through Manager replaces a clear direct dependency without adding responsibility.
- DAO, Manager, and Service log the same stack repeatedly.
- Raw Map query requests erase a structured contract.
- Production SNAPSHOT artifacts change without a deliberate upgrade.
- An upgrade silently changes unrelated transitive libraries.
- dependencyManagement is assumed to import a dependency.
- Historical server tuning is applied without checking actual platform semantics.

## Key Takeaways

1. Route behavior by responsibility, not obligatory empty layers.
2. Keep error transport/log ownership consistent with deployment boundaries.
3. Review effective dependency changes and final packaging.
4. Preserve release immutability and consumer compatibility.
5. Diagnose server constraints before tuning them.

## Connects To

- [Ch01](ch01-programming.md): Interface stability and DO/DTO conventions.
- [Ch02](ch02-errors-and-logging.md): Linkage failures and boundary translation.
- [Ch03](ch03-unit-testing.md): Testable collaborator boundaries.
- [Ch07](ch07-design.md): Dependency inversion and system/module contracts.
