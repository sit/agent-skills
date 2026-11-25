# Worked examples

SKILL.md's rules applied, in Python. The rules hold in any language.

## Builders hold suite invariants; tests hold the scenario

A fluent builder assembles fixture data in the form production consumes, with
defaults that satisfy the suite's invariants. Each test writes only what
makes its scenario differ.

```python
class StoryData:
    def __init__(self) -> None:
        self.data = {
            "about": {
                "slug": "sqs-story",
                "title": "Designing the SQS outbox relay",
                "human": "Ada",
                "t0": T0,
            },
            "source": {"repo": DIRECTORY, "dirs": [DIRECTORY]},
        }

    def set(self, table: str, **fields) -> "StoryData":
        self.data.setdefault(table, {}).update(fields)
        return self

    def entry(self, table: str, **fields) -> "StoryData":
        self.data.setdefault(table, []).append(fields)
        return self

    def chapter(self, id: str, sessions: list[str], **fields) -> "StoryData":
        chapter = {
            "id": id,
            "title": f"Working on {id}",
            "sessions": sessions,
            "opener": {"session": sessions[0]} if sessions else {"session": SID1},
        }
        return self.entry("chapter", **(chapter | fields))

    def build(self) -> dict:
        return copy.deepcopy(self.data)
```

```python
def test_rejects_moment_before_its_chapter_opener():
    story = (
        StoryData()
        .chapter("relay", [SID1])
        .moment("early", "relay", segs=[seg(SID1, "h:the first message")])
        .build()
    )
    with pytest.raises(StoryError, match="opener"):
        load(story)
```

The reader sees the scenario — a moment whose segment precedes its chapter's
opener — because everything else is the builder's default.

Every fixture the builder emits must be valid enough to reach the code under
test: a default that fails validation first hides the scenario.

## Shared constants are invariants; scenario values are local

```python
# tests/support/constants.py — suite-wide invariants
T0 = dt.datetime(2026, 11, 10, 20, 11, tzinfo=dt.UTC)
SID1 = "5a1b356b"
DIRECTORY = "/home/user/src/myproject"
```

`SID1` is shared because the whole suite treats it as "the session a test
builds unless it names another". A test that needs a session *different*
from `SID1` names its own values locally:

```python
def test_orders_sessions_by_first_prompt():
    later, earlier = "bbbb1111", "aaaa0000"   # names carry the scenario
```

Two tests asserting on one shared constant for different reasons break each
other when one changes it; local values cannot.

## Derived values come from a named basis

```python
expires = T0 + ttl            # derived from the threshold under test
last_year = year - 1          # derived, named
```

Independent states get independent names, never arithmetic between them:

```python
# Bad: the reader must compute 14 to learn nothing
total = base + 14

# Good: each state named for what it means
half_full = base + 7
over_limit = base + 21
```

## Build raw input through one conversion helper

When production receives values in a projected or encoded form (a wire
format, a TOML dict, a query string), tests build them through one shared
helper. Hand-assembling the wire form per test duplicates production logic,
and the copies drift:

```python
def seg(session: str, anchor: str, **fields) -> dict:
    return {"session": session, "from": anchor} | fields
```

When the wire format changes, the helper changes and the tests read as
before.

## Time: one anchor per behavior

```python
def test_reports_session_idle_after_an_hour():
    start = T0
    last = start + 59 * MINUTE          # inside the window: not idle
    boundary = start + HOUR             # at the threshold: idle
```

The threshold (`HOUR`) appears in the test unless a production constant is
the contract.

Never let a test read "now" twice: a midnight crossing makes the suite fail
one day a year — rare, real-looking, and wrong. Freeze time with a fixed
anchor or an injected clock; never `sleep`, never `time.time()` in an
assertion.

## Fake backends over mocks

For external, expensive, or independently tested collaborators, prefer a fake
with the same interface (an in-memory store, a stub server) over a mock's
call log. A mock earns its place when the interaction itself is the contract:

```python
# Interaction is the contract: the publisher must receive exactly the pack.
publisher = Mock()
publish(pack, publisher)
publisher.send.assert_called_once_with(pack)

# Collaborator is incidental: a fake keeps the logic real.
store = FakeStore({"session-1": transcript_with(3, prompts)})
digest = summarize(store, "session-1")
assert digest.prompts == 3
```

Asserting that `summarize` called `store.get` would pin an implementation
detail that survives no refactor.

## What an assertion over a shared fixture may name

Fixtures grow; assertions must not break when they do. Assert on:

- Content the scenario created (`story.moments[0].title == ...`).
- Ids and kinds the scenario set.
- Counts of what the scenario added (`len(moments) == 2` when it added
  exactly two).

Never on:

- Total counts of shared fixture content (`len(pages) == 7`).
- Positions of entries the scenario did not move.
- Text of records the scenario did not write.

The check: would adding a chapter to the shared fixture break this test? If
so, it asserts on something it does not care about, and costs the next
author a pointless edit.
