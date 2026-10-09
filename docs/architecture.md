# Architecture

## The split

studykit has three parts:

| Part | Does | Where |
|---|---|---|
| **Engine** | scheduling, scoring, selection, metrics | `studykit/`, Python standard library only |
| **Content** | topics, cards, questions, problems | `packs/` and `~/.studykit/packs/`, TOML and markdown |
| **Teaching** | asking, judging, explaining | `.agents/skills/`, run by an agent |

Each part does what the others cannot. An agent does not give the same result every time. Content stored in code is hard to share. Code cannot judge a free-text answer.

**Anything that must be the same every time is code.** Interval arithmetic, queue order, question selection, coverage, metrics.

**Anything that needs judgement is the agent.** Whether an answer named the tradeoff, whether a worked example worked, what the gap actually was.

## Docs and skills

**`docs/` describes.** What the system is, why it works this way, and how to write content. A person reads it, outside a session.

**A skill instructs.** What the agent should do during a session. The agent reads it every session, so every word costs tokens every time.

A rule about running a session belongs in the skill and nowhere else. A doc can explain why the rule exists. Instructions for a single block are a third case: `select.py` sends them out with the block.

## Agent harnesses

There is one copy of every instruction, symlinked into the places each harness looks:

```
AGENTS.md             the instructions
CLAUDE.md         ->  AGENTS.md
.agents/skills/       the session protocols, one directory per skill
.claude/skills/   ->  ../.agents/skills
```

Codex looks in `.agents/`. Most agents read `AGENTS.md`. Claude Code looks for `CLAUDE.md` and `.claude/skills/`, and follows both symlinks. `AGENTS.md` names each skill by path, so a harness that does not find skills on its own still finds them.

To support a new harness, add a symlink. Never copy. Two copies drift apart.

## Keeping sessions cheap

The agent never reads pack files.

`./study plan 25m` returns a whole session: the calibration brief, the blocks in order, and the questions already picked, filtered by level and mixed. That is a few kilobytes. Reading the manifest, three cards and four question banks instead costs about a hundred times as much. It also repeats the selection logic every time.

The rest of the CLI works the same way:

- `plan` points to cards and problems **by command** instead of including them. Most sessions never open one.
- `record` takes a compact payload with shared defaults at the top, so a twelve-row session is twelve short objects.
- `progress` returns a finished report, not raw data for the agent to interpret.

When you add to the CLI: **if the agent would have to read a file to decide something, the CLI should decide it and return the answer.**

## Data flow

```
pack.toml ─────┐
               ├──► select.py ──► plan / questions  ──► agent runs the session
ledger.jsonl ──┤                                              │
               │                                              ▼
               └──► schedule.py ──► state.json ◄──── record (append + rebuild)
                    metrics.py  ──► metrics.json ──► report.py   ──► terminal
                                                  └► dashboard.py ──► HTML
```

`ledger.jsonl` is the only source of truth. Rows are only ever appended, never edited. `state.json` and `metrics.json` are built from the ledger and the pack taxonomy on every `record`. Deleting them loses nothing.

This means a scheduling change applies to all past data. Fix the interval function, run `./study rebuild`, and every due date is recalculated from the same history. If state were edited in place, a scheduler bug would stay in the data for good.

## Modules

| Module | Does |
|---|---|
| `config.py` | paths, levels, profile, date arithmetic |
| `packs.py` | loads packs and merges in the user's own bank |
| `ledger.py` | validates rows, reads and appends to the ledger |
| `schedule.py` | the interval algorithm. **The only implementation of it** |
| `select.py` | queue order, question drawing, session composition |
| `balance.py` | the target question-type mix per level, and the `doctor` check against it |
| `derive.py` | runs a question's `derivation` and checks the figures in its text |
| `metrics.py` | everything in `metrics.json` |
| `report.py` | terminal output. Read-only |
| `dashboard.py` | the single-file HTML dashboard |
| `tomlwrite.py` | just enough TOML writing to append a banked question |
| `sync.py` | backs the data directory up to its own git remote |
| `cli.py` | argument parsing and command handlers |

Imports go one way. `cli` imports everything. `select` imports `schedule`, `balance`, `packs` and `ledger`. `schedule` imports `ledger` and `packs`. `config` imports nothing. No module imports `cli`.

## Rules the code enforces

- **A score must name a facet the pack declares.** `ledger.validate` rejects anything else, so a typo cannot create a facet that is never scheduled.
- **Derived fields cannot be written.** Passing `strength`, `interval`, `due`, `reps` or `area` to `record` is an error. It is not silently overridden.
- **A problem row must name a problem.** `topic` must be `problem:<slug>` with `subtopic: overall`, so a problem score cannot be recorded as a topic score.
- **A question must be banked before a row can reference its id.**
- **Several scores for one facet on one day count as one rep.** `compute_items` does this, so a session cannot inflate its own evidence.

## Why not a database

After a year of daily use the ledger is a few hundred lines of JSONL. You can grep it, diff it and copy it anywhere. A database would add a dependency, migrations and a file you cannot read, to speed up queries that are already fast enough. For the same reason, packs are TOML and the dashboard is a generated file.

## Extending it

**A new pack.** See [authoring-packs.md](authoring-packs.md). No code changes.

**A new session type.** Add a skill in `.agents/skills/`. Add a session name to `ledger.SESSIONS` if you want reports to tell it apart.

**A new technique block.** Add it to `TECHNIQUES` in `select.py` with its cost, then give it a trigger in `_targeted_blocks`. Without a trigger based on your scores, the block is never chosen.

**A different interval function.** `next_interval` in `schedule.py` is pure and tested. Change it and run `./study rebuild`. History is recalculated, not migrated.

**A different dashboard.** `metrics.json` is a stable public format. Read it with anything.

## Tests

```
./study test --verbose
```

The tests focus on the parts you cannot check by reading: the interval function and its caps, collapsing a day's scores into one rep, an `overall` score being dropped once its topic has sub-topic scores, level filtering, ledger validation and what it rejects, and the full record-then-rebuild path. For the content packs, `./study doctor` is the more useful check.
