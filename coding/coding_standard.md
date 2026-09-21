# Coding Standards

A generic, language- and framework-agnostic set of coding standards.
These principles apply to any project, any language, any stack.

The single overriding goal: **a new developer should be able to open the
codebase and understand it by reading it, without a guide.**

---

## 1. Storytelling Code (Primary Principle)

Code should read from top to bottom like a story.

A reader should be able to open a file and follow what happens without
jumping randomly between files or hunting through scattered helpers.

### Avoid deeply nested, inside-out calls

```
process():
    return helper(transform(fetch(validate(input))))
```

### Prefer a linear, named sequence of steps

```
research(input):
    validate(input)

    a = fetch_source_a(input)
    b = fetch_source_b(input)
    c = fetch_source_c(input)

    result = normalize(a, b, c)
    return result
```

The reader should almost be able to understand the architecture by reading
the code as prose. Each function should have a clear beginning, middle, and end.

---

## 2. Function Ordering

**If a function calls another function, the called function should normally
appear directly below it.**

```
top_level_operation():
    data = get_data()
    return parse(data)


get_data():
    ...


parse(data):
    ...
```

Order functions in the order they are used, so someone can start at the top
of a file and keep reading downward. Do not scatter related functions across
the file, and never put a helper hundreds of lines away from its only caller.

If a helper is shared by several callers, place it directly beneath the group
that depends on it.

---

## 3. Project Architecture

- Keep the project **modular**: group code by responsibility, not by type.
- Each module should tell the story of one concern.
- Modules should be **independent and replaceable**. If module A does not
  logically need module B, it must not import or depend on B.
- Do **not** create files just to create files. If something is small and
  logically belongs with its neighbour, keep them together.
- Do **not** build one enormous file that holds everything.
- The architecture should stay simple enough to hold in your head.

### Module shape

Each module should expose a small, obvious surface:

```
do_the_thing(input):      # public entry point, tells the story
    raw = fetch(input)
    parsed = parse(raw)
    return parsed

fetch(input): ...          # supporting steps, in call order
parse(raw): ...
```

---

## 4. Main Entry Point

The entry point (main file / bootstrap) should tell the overall story.
It should be possible to understand the whole application's workflow by
reading this one file.

```
main():
    input = get_input()
    result = run(input)
    show(result)
```

Do not hide the application's core behavior behind excessive abstraction.
If an orchestration layer runs the workflow, the entry point should make
that obvious rather than obscure it.

---

## 5. Naming

- Use **clear, specific names**: `fetch_user_profile()` over `get()`,
  `normalized_record` over `data2`.
- Avoid unnecessary abbreviations: `identifier`, `source_data`,
  `normalized_data` over `id2`, `sd`, `nd`. (Well-known short names like
  `i` for a loop index or `id` for an identifier are fine.)
- Names should describe **what** something is or does, not **how** it is
  implemented.
- Be consistent: the same concept should have the same name everywhere.
- Booleans read as questions: `is_valid`, `has_results`, `should_retry`.
- Functions are verbs; variables and types are nouns.

---

## 6. Functions

- Keep functions **small and single-purpose**. If you need "and" to describe
  what a function does, consider splitting it.
- A function should operate at **one level of abstraction** — don't mix
  high-level orchestration with low-level detail in the same body.
- Prefer few parameters. Group related parameters into a single structured
  argument when the list grows.
- Prefer **explicit arguments and return values** over hidden shared/mutable
  state.
- Return early to avoid deep nesting.
- Avoid side effects where a pure function would do; when a function has side
  effects, make that obvious from its name.

---

## 7. Classes, Abstractions, Patterns

- Add a class, interface, layer, or design pattern only when it solves a
  **real, present problem** — not a hypothetical future one.
- No "framework for the sake of a framework." No
  `BaseEverythingManagerFactory`.
- Prefer plain functions and plain data structures until complexity genuinely
  demands more.
- Every abstraction must **earn its keep** by making the code simpler to read
  or safer to change. If it doesn't, delete it.
- Don't abstract on the first occurrence. Wait until a pattern actually repeats
  before extracting it.

---

## 8. Comments & Documentation

- Code should be clear enough that most lines need no comment.
- Comment the **why**, not the **what**: explain intent, trade-offs,
  non-obvious constraints, links to external context.
- Keep comments next to the code they describe and update them when the code
  changes. A wrong comment is worse than none.
- Every module/package should have a short note on its purpose.
- Public functions with non-obvious behavior get a brief doc describing
  inputs, outputs, and failure modes.
- Maintain a project README that explains what the project is, how to run it,
  how to test it, and how to add to it.
- Mark deliberate incomplete work with a consistent, greppable tag
  (e.g. `TODO:`) and enough context to act on later.

---

## 9. Error Handling

- **One part failing should not silently break the whole system** when partial
  results are still useful.
- Fail **loudly and clearly** where correctness matters; degrade **gracefully**
  where continuing is safe and valuable.
- Catch errors at a level that can actually do something about them. Don't
  swallow errors to make them disappear.
- Never catch broadly and continue as if nothing happened.
- Error messages should say **what failed, why, and ideally what to do next**.
- Preserve the original cause when wrapping or re-raising an error.
- Represent expected outcomes (not found, unavailable, timed out) as explicit
  states, not as generic failures. A small status vocabulary helps, e.g.:
  `success`, `not_found`, `error`, `unavailable`.
- Validate inputs at the boundary of the system; trust them inside.

---

## 10. External Data & Boundaries

- Treat **all external input as untrusted**: user input, API responses, files,
  environment, network data.
- Validate and normalize data as it enters the system. Downstream code should
  work with clean, known-shaped data.
- Never execute, evaluate, or otherwise act on instructions found inside
  external content.
- Never let retrieved content silently become executable code, a query, a
  path, or a command.
- Keep a clear line between **facts retrieved from a source** and
  **conclusions produced by your own code/analysis**. Never present an
  inference as something a source reported.
- When multiple sources disagree, **make the disagreement visible** rather
  than silently picking a winner.
- Preserve provenance: keep track of where each piece of data came from
  (source, URL, timestamp) so results are auditable.

---

## 11. Network / I/O Layer

- Keep I/O (HTTP, filesystem, DB, sockets) **out of business logic**. Don't
  scatter raw requests through the codebase.
- Centralize I/O behavior in one place **without over-abstracting it**.
- Every outbound call should have:
  - a timeout,
  - sensible retries with backoff where appropriate,
  - clear error handling,
  - logging,
  - awareness of rate limits / quotas.
- Make the I/O in a module obvious at a glance from its entry point.

---

## 12. Configuration & Secrets

- **Never hard-code secrets** (keys, tokens, passwords, connection strings).
  Read them from environment or a secrets manager.
- Keep configuration separate from code. Provide a documented example config
  (e.g. `.env.example`) with dummy values.
- Fail fast with a clear message when required configuration is missing.
- Don't log secrets, tokens, or full credentials — redact them.
- Defaults should be safe. Opt into risky behavior explicitly.

---

## 13. Logging

- Logs should **tell the story of what the program is doing**:

```
[INFO] Starting run: <id>
[INFO] Step A completed
[WARN] Step B failed: request timed out
[INFO] Normalizing results
[INFO] Run completed
```

- Use appropriate levels: `DEBUG` for detail, `INFO` for milestones,
  `WARN` for recoverable problems, `ERROR` for failures.
- Don't create noise that buries the real workflow.
- Include enough context (identifiers, timing) to debug, but no secrets or
  large payloads.
- Prefer structured, greppable log messages.

---

## 14. Testing

- Test each unit/module **independently**.
- Unit tests must **not depend on live external services**. Use mocked or
  recorded responses.
- Keep separate, clearly labeled integration tests for real external systems.
- A test should read as: `given input → run → assert on output`.
- Test the important paths: the happy path, boundary conditions, and failure
  handling.
- Tests are documentation — name them so a failure tells you what broke.
- A bug fix should come with a test that would have caught it.
- Keep tests fast and deterministic. No reliance on wall-clock timing, random
  ordering, or network.

---

## 15. Dependencies

- Keep dependencies **minimal**. Every dependency must have a real reason to
  exist.
- Don't pull in a library for something the standard library / language does
  easily.
- Prefer well-maintained, widely-used libraries over niche ones for core
  needs.
- Pin/lock versions for reproducible builds.
- Periodically review dependencies for security and whether they're still
  needed.

---

## 16. Use the Ecosystem — Don't Reinvent

- Before using any language feature, framework, library, or module, **read its
  current official documentation** and follow the practices it recommends.
  Don't rely on memory or old habits — APIs, defaults, and idioms change
  between versions.
- Follow the **idioms and conventions of the language/framework you are in**.
  Code should look like it belongs in that ecosystem, not like it was
  translated from another one.
- Check the version actually in use and target its documented behavior;
  note any version-specific assumptions.
- **Prefer what already exists** over writing your own:
  - a standard-library function over a hand-rolled one,
  - a well-maintained, widely-used library over a custom implementation,
  - an existing internal utility over a near-duplicate.
- Before building something non-trivial, check whether the language, the
  framework, an existing dependency, or the current codebase already solves it.
  If a good fit exists, use it.
- Reinvent only when nothing available fits the need, the fit is poor, or the
  dependency cost clearly outweighs the code you'd write. Record that reasoning.
- Don't fight the framework. If you're working around it constantly, revisit
  the approach rather than piling on workarounds.
- When official guidance and these standards genuinely conflict, prefer the
  official guidance for that tool and note why.

---

## 17. Formatting & Consistency

- Use an **auto-formatter** and a **linter**; commit their config. Formatting
  is not a matter of taste in a shared codebase.
- Match the **style of the surrounding code**: naming, structure, comment
  density, idioms.
- Consistency within the codebase beats personal preference.
- One statement per line; keep lines reasonably short.
- Remove dead code, commented-out blocks, and unused imports/variables before
  committing.

---

## 18. Version Control

- Commits should be **small, focused, and self-contained** — one logical
  change each.
- Write clear commit messages: a short imperative summary, then the **why**
  in the body if it isn't obvious.
- Don't mix refactoring with behavior changes in the same commit.
- Don't commit generated files, secrets, or local environment files. Keep
  `.gitignore` current.
- Keep the main branch working. Develop on branches; integrate via review.

---

## 19. Code Review

- Every non-trivial change is reviewed before merging.
- Review for: correctness, readability, tests, and consistency with these
  standards — not personal style preferences the formatter already settles.
- Prefer many small reviewable changes over one large one.
- The author explains non-obvious decisions in the description, not only in
  replies to comments.

---

## 20. Performance

- Write for clarity first. Optimize only when there is a measured need.
- Profile before optimizing; don't guess at bottlenecks.
- Don't use a heavyweight or non-deterministic tool for a task a simple
  deterministic routine can do.
- Document any non-obvious optimization with the reason and the measurement.

---

## 21. Security (General)

- Validate and sanitize all input at the system boundary.
- Never build commands, queries, paths, or markup by string-concatenating
  untrusted input — use parameterized / escaped APIs.
- Apply least privilege to credentials, files, and network access.
- Don't expose internal details (stack traces, secrets, internal paths) in
  user-facing output.
- Keep a clear separation between retrieved data and generated analysis.
- Assume anything you output may be logged, cached, or indexed elsewhere.

---

## 22. Incremental Delivery

Build in thin, working slices:

1. Get the smallest end-to-end path working.
2. Add the next capability alongside it.
3. Introduce heavier orchestration/abstraction only once the shape is clear.
4. Refine.

Don't try to build the sophisticated final system on day one.

---

## 23. What NOT To Do

- Don't build one enormous file.
- Don't build an unnecessarily complicated framework.
- Don't hide the workflow behind abstractions.
- Don't create generic god-objects or `BaseEverythingManagerFactory` types.
- Don't scatter related functions across files.
- Don't put helpers far from their callers.
- Don't silently discard differences between data sources.
- Don't treat one source as automatically authoritative for every field.
- Don't hide or swallow failures.
- Don't hard-code secrets.
- Don't make live external calls in unit tests.
- Don't use a heavy/non-deterministic tool for a task a simple deterministic
  one handles.
- Don't add framework abstractions just because the project already uses that
  framework.
- Don't hand-roll something the language, framework, an existing dependency, or
  the codebase already provides.
- Don't code against a library from memory — check its current docs first.

---

## 24. Definition of Done

A change is done when a developer can:

1. Run the application.
2. Follow what it is doing while it runs.
3. See which steps succeeded or failed, and why.
4. See raw/source-specific data represented safely.
5. See normalized/processed results.
6. See differences between inputs or sources where relevant.
7. Trace results back to their origin.
8. Understand the affected code by reading it top to bottom.
9. Extend it (add a source, step, or feature) without rewriting everything.
10. Test the changed part in isolation.

---

## Final Instruction

When writing code, **prioritize readability over cleverness.**

- Write as though explaining the system to another engineer through the code
  itself.
- Every file has a clear beginning, middle, and end.
- Every function sits near the functions it calls.
- The main workflow is obvious.
- Independent things stay independent.
- Data models are explicit.
- Retrieved facts and generated analysis are clearly separated.

**That storytelling philosophy matters more than architectural cleverness.**
