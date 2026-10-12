# Alibaba Java Standards: Songshan Edition

An agent skill generated from *Alibaba Java Development Manual, Songshan edition (1.7.0, 2020-08-03)* by Alibaba Group technical teams and community contributors with [book-to-skill](https://github.com/virgiliojr94/book-to-skill). It helps agents apply the edition's Java, testing, security, MySQL, dependency, and design rules while coding and reviewing. All instructional prose is English. Mandatory, recommended, and reference levels remain distinct, with explicit technical caveats for historical claims.

## Install

```text
npx skills add https://github.com/Lilb1tty/alibaba-java-standards --skill alibaba-java-standards
```

For a private repository, authenticate Git access with an account that can read the repository before installing.

## Use

Ask your agent:

```text
Use alibaba-java-standards to review the current Java changes.
Use alibaba-java-standards to check this MyBatis mapping.
Use alibaba-java-standards for ch03 and derive BCDE test cases.
```

The master skill routes topics to chapter files. Chapters load on demand rather than loading the whole knowledge base for every task. Explicit user instructions and project requirements take precedence over edition conventions.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | Application workflow, core rules, chapter/topic indexes, provenance |
| `chapters/ch01-programming.md` | Naming, precision, dates, collections, concurrency, APIs |
| `chapters/ch02-errors-and-logging.md` | Error codes, exception handling, logging |
| `chapters/ch03-unit-testing.md` | AIR, BCDE, fixtures, coverage |
| `chapters/ch04-security.md` | Authorization, validation, safe output, replay/abuse controls |
| `chapters/ch05-mysql.md` | Schema, indexes, SQL, typed ORM mapping |
| `chapters/ch06-project-structure.md` | Layers, library contracts, dependencies, server caveats |
| `chapters/ch07-design.md` | Design reviews, diagram thresholds, change isolation |
| `glossary.md` | Alphabetical terminology with chapter pointers |
| `patterns.md` | Implementation recipes and trade-offs |
| `cheatsheet.md` | Decision rules, thresholds, and review smells |
| `LICENSE` | Apache License 2.0 |
| `NOTICE` | Source attribution and adaptation notice |

## Source, Changes, and Limits

The content is synthesized guidance and compact adapted examples, not the book text. The source PDF is not included. The manual's version history and the [official P3C project](https://github.com/alibaba/p3c) identify Apache 2.0 licensing; attribution is retained in NOTICE.

This adaptation translates and reorganizes the material, condenses it into decisions and procedures, reconstructs selected examples, and adds labeled runtime clarifications. It is not an exhaustive transcription, the latest Alibaba manual, a legal retention-policy opinion, or a substitute for project-specific requirements. Chapter references use the original manual's printed page numbers.

Before publication, the generated skill passed book-to-skill's advisory scan for known injection/authority patterns, local link/structure checks, English-prose checks, and supporting-file token budgets. Such checks validate packaging and known patterns; they do not establish universal correctness of every recommendation.

## License

Apache License 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
