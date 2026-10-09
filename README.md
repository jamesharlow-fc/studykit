# studykit

A spaced-repetition study kit for engineering knowledge. You run it through a coding agent.

You say how much time you have. The agent picks what to work on, asks questions pitched at your level, and scores your answers before teaching you anything. The code then decides when each topic comes back. Scheduling, scoring and metrics are Python. The questions and material are data files. The agent does the teaching.

```
git clone <this repo> && cd studykit
./study setup
```

Then tell your coding agent **`study 25m`**. It works with Claude Code, Codex, and any other agent that reads `AGENTS.md`.

You need Python 3.12+ and a shell. There is nothing to install.

---

## What it does

| Say | Get |
|---|---|
| `study 25m` / "I've got an hour" | a session built from what is due, sized to the time you have |
| `quiz` | quick questions with short answers, mostly judgment |
| `practice` | a full design problem, in coaching or interview mode |
| `learn <topic>` | a test first, then teaching aimed at what you got wrong |
| `progress` | what is due, what is weak, and what you have never been tested on |

If you are not sure, use `study`.

## What ships with it

| Pack | Topics | Questions | Problems | For |
|---|---:|---:|---:|---|
| **foundations** | 10 | 186 | 4 | HTTP, SQL, concurrency, testing, security, encoding, the OS |
| **design-patterns** | 8 | 112 | 3 | SOLID, GoF patterns, DI, coupling, refactoring, boundaries |
| **system-design** | 16 | 263 | 12 | caching through multi-region, graduate to staff |
| **quantitative-reasoning** | 6 | 60 | 0 | estimation, tails, queueing, coincidence, capacity, measurement |

## Levels

`graduate` · `mid` · `senior` · `lead` · `staff`

Your level sets three things:

- **The mix of question types.** Graduates get more recall. Seniors and above get more judgment.
- **Which topics and problems you see.** A graduate is not asked about multi-region failover.
- **The bar for each score.** At graduate, explaining the mechanism correctly scores well. At staff, a claim with no numbers and no failure mode does not.

The mix changes as you go. Question types you answer well come up less often. Questions written for other levels come up less often too, but are never hidden.

```
./study levels                    # see the bar for every level
./study config set level lead     # change your level
```

## How a session goes

```
you:   study 25m
agent: 12 questions across 12 topics, weakest first. Starting.
       > What is the boundary defect in a fixed-window counter?
you:   burst at the end of one window plus the start of the next
agent: 4. You didn't say it's twice the limit in under one window - that's the number
       that makes it concrete.
       > p99 tripled, error rate flat, cache hit rate unchanged. Most likely cause?
...
agent: Recorded. caching/hot-key 2 -> 3, next 7 Aug. Two facets came back 2;
       both got a worked example and a variant re-test.
```

Run `./study dashboard --open` afterwards to see charts.

## Why it works this way

Past junior level, knowing a fact is rarely the problem. Spotting when it applies is. So there are five question types, and the higher your level, the more of them test judgment rather than recall.

The other rules:

- Topics are mixed together, and the agent does not say which topic a question is from.
- You are scored before you are taught. Only that first score decides when a topic comes back.
- Scores are kept per sub-topic. A good score on one never counts towards another.
- You can say how confident you are. That shows how well you judge your own knowledge. It never affects scheduling.

[docs/research.md](docs/research.md) gives the source for each rule, and lists where the evidence is disputed.

## Your data

Your data is stored in `~/.studykit`, not in this checkout, so you can delete or re-clone the repo without losing it.

```
profile.json      your level and preferences
ledger.jsonl      every score you have recorded, append-only
state.json        current strength and due dates, built from the ledger
metrics.json      metrics, built from the ledger
bank/             questions written during your sessions
attempts/         your problem attempts
packs/            packs you installed
dashboard.html    charts
```

If `XDG_DATA_HOME` is set, studykit uses it. `STUDYKIT_DATA` overrides both. Set it to a different directory to keep a separate profile.

Nothing leaves your machine unless you ask it to.

### Syncing between machines

The ledger is the only file you cannot rebuild, so back it up. Put it in a private git repository:

```
./study sync init git@github.com:you/studykit-data.git --auto
```

This turns `~/.studykit` into a git repository. With `--auto`, `./study plan` pulls before each session and `./study record` pushes after it. Files built from the ledger are not committed.

To sync by hand, run `./study sync` to push and `./study sync pull` to pull. `./study sync status` shows what has not been synced yet.

If you study on two machines while offline, both sets of scores are kept when you next sync. They do not conflict. To set up a second machine, clone your data repository into `~/.studykit`.

## Commands

```
./study setup                     first-run setup
./study status                    what is due and what is next
./study progress                  full report
./study plan 25m                  build a session (JSON, for the agent)
./study questions --count 8       pick questions (JSON, for the agent)
./study card caching              print the notes for a topic
./study problem --list            problems at your level
./study record --json-text '...'  save scores and rebuild state
./study dashboard --open          HTML charts in one file
./study packs                     installed content and how much you have covered
./study sync                      push your data to its repository
./study doctor                    check packs and data for errors
./study test                      run the tests
```

Every command and flag: [docs/cli.md](docs/cli.md).

## Documentation

| File | What it covers |
|---|---|
| [docs/scoring.md](docs/scoring.md) | how answers are scored and scheduled |
| [docs/research.md](docs/research.md) | the research behind each rule |
| [docs/question-types.md](docs/question-types.md) | the five question types |
| [docs/techniques.md](docs/techniques.md) | the kinds of block a session is built from |
| [docs/levels.md](docs/levels.md) | what your level changes |
| [docs/authoring-packs.md](docs/authoring-packs.md) | writing your own pack |
| [docs/cli.md](docs/cli.md) | every command and flag |
| [docs/architecture.md](docs/architecture.md) | how the code is organised |

## Requirements

Python 3.12 or newer, and a POSIX shell. If the `python3` on your PATH is older, set `STUDYKIT_PYTHON=/path/to/python3.12`.

Run the tests with `./study test --verbose`.

If you are working on studykit, run `git config core.hooksPath .githooks`. The tests and `doctor` will then run before every push. To skip them once, use `git push --no-verify`.

## Licence

MIT. See [LICENSE](LICENSE).
