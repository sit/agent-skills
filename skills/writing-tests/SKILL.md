---
name: writing-tests
description: Write and change tests that pin behavior, survive refactors, and fail for the right reason — TDD cycle, test layout, fixture builders, naming, determinism, mock policy, verification scope. Use when writing new tests, fixing or explaining a failing test, deciding what to test, reviewing test coverage, or changing code that has tests; also when the user asks to "add tests" or "cover this case".
---

# writing-tests

A test specifies behavior: it fails when the behavior breaks and passes when
it holds. Tests that restate the implementation add noise; tests that pin
arbitrary configuration add drag; tests that survive a real bug add nothing.

## The cycle

Develop features and bug fixes in red-green-refactor cycles:

- **Red:** write tests that specify the required behavior without
  over-constraining the implementation. Watch each fail and read the failure:
  a test that passes before the code exists specifies nothing.
- **Green:** write just the code that makes them pass.
- **Refactor:** review the code for maintainability, then the tests: what
  would a reviewer reading only the diff conclude?

## What a test asserts

Assert on public behavior: domain outcomes and state the caller can observe.

- Design each test around a realistic regression: what plausible
  consumer-visible bug does it catch? Wiring, pure echoes, source text,
  incidental defaults, bare not-throwing, and "the list grew" are not bugs.
  Never restate the implementation or pin an arbitrary fixed configuration.
- Cover the complementary outcomes of each branch under test.
- The name states the scenario and expected result, so it tells a reviewer
  what broke; the assertions fully support what the name claims.
- Assert on content, not on shape the implementation chose. Over a shared
  fixture, assert on the text or ids the test cares about, so extending the
  fixture breaks no test that does not care about the extension. Pin
  positions, counts and times only for data the test builds itself.
- Assert so a failure names what differs: compare values or project the
  properties under test, never a boolean computed from them. Use the
  assertion library's matchers to write the most expressive tests.
- Characterization tests are temporary regression protection for behavior
  that lacks a practical behavioral test boundary. Keep them to stable
  semantics tied to a concrete behavior or past regression, and mark the
  group with a short comment stating the limitation and the boundary that
  should replace them.

## What not to test

Skip argument parsing, pass-through to external tools, getters and setters,
and anything the type checker or compiler already proves. Coverage is not a
target; a module with no decisions in it needs no test file.

## Where tests live

- A module's unit tests sit at its path: `pipeline/pack.py` is tested in
  `tests/pipeline/pack/`. Moving a module moves its tests; either path finds
  the other.
- One test file per responsibility of the unit under test, named for the
  unit and that responsibility, so tools and readers pair it with its
  subject. Integration tests carry a name the runner can select apart.
- Shared helpers — builders, constants, fake backends — live once, in
  `tests/support/`, never copied between test files.
- End-to-end tests sit outside the per-module tree: they test the product,
  not a module.

## Arrange–Act–Assert, visibly

- Each test reads as arrange, act, assert: the setup, the one call under
  test, and what follows from it.
- Keep the call under test visible. Helpers may build or persist fixtures;
  they must not hide the query, command, or result being asserted, or the
  test cannot be read without reading the helper.
- One behavior per test. Two assertions on one outcome are fine; a second
  act is a second test.
- Elaborate setup — many mocks, many arguments — signals a design fault: fix
  the design, not the test.
- Comments explain what name and code cannot: non-obvious domain
  constraints, fixture techniques, test-boundary decisions.

## Fixtures and builders

- Fluent builders construct fixture data: a base builder holds suite-wide
  invariants; each test sets its scenario values visibly.
- Helpers cut noise and duplication; name them so tests read fluently. Noun
  helpers return objects for composition; helpers that persist or change
  state are verbs.
- Prefer production enums and constants when they define the tested value.
- Use real entities and value objects when setup is small. Fake or mock
  external, expensive, or independently tested collaborators, keeping the
  decision logic real. Verify interactions only when the interaction is the
  contract.
- Build values that production receives in raw or projected form through
  one shared conversion helper; per-test hand-assembled copies drift from
  what production actually sends.

## Naming values

- Name fixture values — numbers, strings, dates, identifiers — that carry
  scenario meaning, connect setup to assertion, or participate in a
  relationship. Prefer a test-local value; use a shared constant only for a
  suite invariant or shared fixture default. Once a second value competes,
  the test names its own.
- Derive related values from one named basis (`last_year = year - 1`); name
  independent states separately rather than inventing arithmetic between
  them.
- Leave self-evident structural values inline — an initial `0`, a single
  boundary `+ 1`, a length of `2` after two explicit additions; extracting
  them hides the structure. Do not extract irrelevant filler just to remove
  literals.

## Determinism and time

- One time anchor per behavior; derive scenario times from the interval or
  threshold under test. Use the production constant when it defines the
  contract; otherwise name the threshold in the test.
- When production behavior is relative to the current time, capture it once;
  otherwise use a fixed anchor or an injected clock.
- A test passes alone and in the suite, in any order and any timezone. No
  test reads the wall clock, the network, or another test's state.

## Changing code that has tests

- When a change alters what a function returns or prints, run its tests and
  its callers', and update the assertions and test data that pin the old
  output or shape.
- Update contract tests when the contract changes. Delete tests that pin
  wording or implementation details, whoever wrote them; never re-pin them.
- Run the narrowest check that covers the change: the tests of the changed
  code and its callers. Run the whole suite only when nothing narrower
  answers, and once before push — by the pre-push hook, where the repo has
  one.
- When the code does not show a reported failure's cause, reproduce it once
  before fixing, so the fix addresses the failure, not the report.

## references

- [references/examples.md](references/examples.md) — worked examples:
  builders, constants vs local values, conversion helpers, time anchors,
  fakes vs mocks, assertions over shared fixtures.
