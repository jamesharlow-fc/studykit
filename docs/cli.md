# CLI reference

```
./study <command> [options]
```

These options work before or after the command name:

- `--date YYYY-MM-DD`: treat this date as today when working out what is due and how overdue it is. Use it for testing, or to see the queue as it was on a past date. It never changes when a measurement happened.
- `--json`: print JSON instead of the human view, for commands that have both.
- `--version`

Environment variables:

| Variable | Effect |
|---|---|
| `STUDYKIT_DATA` | where your data lives. Default `$XDG_DATA_HOME/studykit`, else `~/.studykit`. |
| `STUDYKIT_PACKS` | where the shipped packs are read from. Default `<repo>/packs`. Packs you installed are always read from the data directory as well. |
| `STUDYKIT_PYTHON` | the Python to use, if `python3` is older than 3.12. |
| `STUDYKIT_TODAY` | today's date, for working out what is due. |
| `STUDYKIT_NOW` | the timestamp put on new rows. |
| `NO_COLOR` | turn off colour. |

On an error, the command prints one line to stderr and exits `2`. There is no traceback. `doctor` and `test` exit `1` when they find problems.

If you run a command without what it needs to do anything, it prints its help and exits `0`. This applies to `./study` on its own, to commands that only group subcommands, and to commands that need an argument or a JSON payload.

---

## Getting started

### `setup`

Asks for your level, packs, default session length and default problem mode, then creates the data directory.

```
./study setup
./study setup --level senior --packs system-design,foundations --non-interactive
./study setup --force                 # overwrite an existing profile
```

Options: `--level`, `--packs`, `--budget`, `--mode`, `--no-confidence`, `--non-interactive`, `--force`.

### `config`

```
./study config get                    # everything, as JSON
./study config get level
./study config set level lead
./study config set packs system-design,design-patterns
./study config set budget 45m
./study config set mode interview
./study config set confidence_prompt false
./study config set sync_auto true
```

Every change rebuilds state and metrics.

| Key | Effect |
|---|---|
| `level` | the bar answers are judged against, which topics are in scope, and the mix of question types |
| `packs` | the packs in rotation, comma-separated |
| `budget` | default session length |
| `mode` | `coaching` or `interview` |
| `confidence_prompt` | whether to ask how confident you are before you answer |
| `sync_remote` | the git remote your data directory is backed up to. Set it with `sync init` |
| `sync_branch` | the branch on that remote. Default `main` |
| `sync_auto` | pull before each session and push after each write. Needs a remote first |

---

## Reading

### `status`

One screen: how many facets are overdue, due today and never tested, your coverage, your mean strength, and the session to do next.

### `progress`

The full report: what is due, what has never been measured, your weakest facets, your score by question type, how well your confidence matches your scores, facets with little evidence, areas with no coverage, problems, and what to do next. `--json` returns state, metrics and the recommendation together.

### `queue`

The list of facets to work on, in order. Overdue comes first, then due today. Within each group, the longest-unseen comes first, and the weaker facet wins a tie. Every third slot goes to a facet that has never been measured, so a backlog of reviews cannot stop you meeting new material. Options: `--limit N`, `--json`.

### `packs`

Lists every installed pack, marked `on` if it is in rotation and `off` if not, with coverage per topic at your level.

```
./study packs                         # what is installed, and what is on
./study packs enable foundations      # put a pack into rotation
./study packs disable design-patterns # take it out
```

Both subcommands take several names. `plan`, `questions` and `queue` draw only from packs in rotation. A pack that is off is still listed, but nothing is drawn from it. You cannot take the last pack out: an empty rotation means every pack, not none.

### `levels`

The list of levels, and the calibration brief for each enabled pack.

### `card <topic>`

Prints the notes for a topic. Use `--pack` if two packs have a topic with the same id. `--json` adds the topic's metadata.

### `problem [slug]`

```
./study problem                       # the one the scheduler picks
./study problem --list                # every problem at your level, with your history
./study problem url-shortener         # the prompt the candidate sees
./study problem url-shortener --notes # the interviewer notes
```

`--pack` picks the pack if the slug is ambiguous.

**The prompt and the notes are separate files.** You only get the notes with `--notes`, so an agent can fetch a problem mid-session without seeing them.

### `export <what>`

Prints `state`, `metrics`, `ledger` or `profile` as JSON.

---

## Running a session

These commands print JSON for an agent to read.

### `plan [budget]`

Builds a session that fits the time you have. It returns the calibration brief, the blocks in order, the questions already picked and mixed across topics, and a preview of the queue.

```
./study plan 25m
./study plan 1h
./study plan "half day"
./study plan                          # your default session length
```

Options: `--level` (for this call only), `--mode`, `--seed N` (same selection every time), `--no-problem`.

Budgets can be `15m`, `1h`, `90`, `half day`, `full day`, `quick` or `open`.

With `sync_auto` on, `plan` pulls from your data repository first.

### `questions`

Picks questions from the topics in scope at your level, in queue order, mixed across topics.

```
./study questions --count 8
./study questions --topic caching --subtopic hot-key
./study questions --qtype judgment --count 5
```

Other options: `--pack`, `--level`, `--seed N`.

Questions you answered today are left out. Questions you have never seen, or not seen for a long time, are picked before ones you answered recently.

Your level and your mean score per type decide the mix of question types. See [levels.md](levels.md#what-your-level-changes). Within a type, questions written for your level come first. The rest of the topic is used when those run out.

The response includes `author_for`: facets that have no questions left to pick.

---

## Writing

### `bank add`

Saves questions written during a session and gives them ids. Never write ids yourself.

```
./study bank add --json-text '{
  "pack": "system-design",
  "topic": "caching",
  "questions": [
    {"subtopic": "hot-key", "qtype": "judgment", "q": "...", "a": "..."}
  ]
}'
```

`pack`, `topic` and `levels` at the top level apply to every question. Any question can set its own. `levels` defaults to your level only, because a question written mid-session was written for the level it was asked at. Set it yourself if the question suits more than one level.

Questions go to `~/.studykit/bank/<pack>/<topic>.toml`. This file is yours. It is merged with the pack when loaded. Each id is `<prefix>-u<hash of the question>`. Pack authors can pass `--into-pack` to write into the pack's own directory instead, with a sequential id.

The payload can also come from `--file` or stdin. `--data` is the same as `--json-text`.

A question that states a figure needs a `derivation`. It is refused if it has none, or if a figure in it does not match any result of the derivation. If any question in a batch is refused, nothing is written. See [authoring-packs.md](authoring-packs.md#figures-are-derived-not-asserted).

With `sync_auto` on, `bank add` pushes after writing, unless `--into-pack` is set.

### `bank check`

Runs the same figure check without writing anything. Run it **before** you ask the question, not after.

```
./study bank check --json-text '{"questions": [
  {"q": "Origin is 150 ms RTT away and setup is 350 ms.", "a": "Three round trips.",
   "derivation": ["rtt_ms = 150", "setup_ms = 3 * rtt_ms"]}
]}'

error: These figures match no derivation result: 350 ms.
       The derivation gives rtt_ms = 150, setup_ms = 450.
```

It returns the figures it found and every value it computed. You can use it to see what the checker counts as a figure.

### `record`

Adds measurements to the ledger, then rebuilds `state.json` and `metrics.json`.

```
./study record --json-text '{
  "session": "quiz",
  "pack": "system-design",
  "rows": [
    {"topic": "caching", "subtopic": "hot-key", "qtype": "judgment",
     "qid": "ca-014", "measured": 2, "predicted": 4, "post": 4, "taught": true}
  ]
}'
```

`session`, `pack`, `at` and `level` at the top level apply to every row. You can also pass a plain JSON array of rows. The payload can come from `--json-text`, `--file` or stdin.

**When a row happened.** Each row has an `at` field: a full local timestamp with its UTC offset. By default it is the moment you record the row. Only set `at` yourself when writing up a past session and you know the exact time. `--date` only changes what counts as today for the rebuild. It does not change `at`.

Scheduling counts whole days, taken from `at`.

| Field | Required | Notes |
|---|---|---|
| `session` | yes | `quiz` `practice` `review` `learn` `study` `drill` |
| `pack` | yes | must be installed |
| `topic` | yes | a pack topic, or `problem:<slug>` |
| `subtopic` | yes | one the topic declares, or `overall` |
| `measured` | yes | 1-5, the score **before any teaching** |
| `at` | no | ISO 8601 local timestamp. Defaults to now |
| `qtype` | no | quiz rows only |
| `qid` | no | must already be banked |
| `predicted` | no | the user's own number. Leave it out if they did not give one |
| `post` | no | the score on a variant question after teaching. Never affects scheduling |
| `taught` | no | true if the score was 1-2 and teaching followed |
| `note` | no | free text, shown in reports |

`record` rejects unknown topics or subtopics, unknown question ids, scores outside 1-5, and any row that sets a field the engine calculates (`strength`, `interval`, `due`, `reps`). `--dry-run` checks the rows without writing them.

With `sync_auto` on, `record` pushes after writing.

### `rebuild`

Rebuilds `state.json` and `metrics.json` from the ledger. `record` does this for you. You only need it after editing the ledger by hand or changing packs.

### `dashboard`

```
./study dashboard
./study dashboard --open
./study dashboard --out ~/study.html
```

Writes a single HTML file with the CSS and SVG inline. It makes no network requests. It has light and dark themes and a toggle between them.

---

## Maintenance

### `sync`

Backs up your data directory to a git repository you own. This is separate from the studykit repository, so studykit can be public while your ledger stays private.

```
./study sync init git@github.com:you/studykit-data.git
./study sync init git@github.com:you/studykit-data.git --auto
./study sync init git@github.com:you/studykit-data.git --branch studykit
./study sync                          # commit what changed, push
./study sync -m "after the Friday session"
./study sync pull                     # fetch, rebase onto the remote, rebuild state
./study sync status
```

`init` turns the data directory into a git repository, adds the remote, writes `.gitignore` and `.gitattributes`, and pushes. `--auto` also turns on `sync_auto`. `init` fails if the remote already has history. In that case, clone it into your data directory instead:

```
rm -rf ~/.studykit && git clone git@github.com:you/studykit-data.git ~/.studykit
```

`.gitignore` leaves out the files rebuilt from the ledger (`state.json`, `metrics.json` and `dashboard.html`) and temporary files. The rebuilt files change on every write, so committing them would give you a diff after every command and a conflict on every second machine. `ledger.jsonl`, `profile.json`, `bank/` and `attempts/` are committed.

`.gitattributes` sets `merge=union` on `ledger.jsonl` and the bank files. If two machines both add rows while offline, the merge keeps both sets.

With `sync_auto` on, `plan` pulls before a session, and `record` and `bank add` push after writing. A failed pull or push never fails the command. Your measurement is already saved locally. A warning tells you to run `./study sync` later.

`sync status` shows the remote, the branch, and anything not yet pushed or pulled. `doctor` also notes anything not yet pushed, and reports a problem if a remote is set but the data directory is not a git repository.

### `test`

Runs the test suite.

```
./study test
./study test --verbose
```

### `doctor`

Checks packs and data. It reports two kinds of finding: problems and notes.

Problems:

- a topic that declares no subtopics
- a topic whose area is not in the pack's `areas`
- a problem without a prompt or notes
- a question naming an unknown topic or subtopic
- a figure in a question that matches no result of its derivation
- a ledger row that no longer validates against the current packs
- a sync remote that is set when the data directory is not a git repository

A duplicate question id stops the packs loading at all, so `doctor` exits with an error rather than listing it.

Notes:

- a topic with no card
- a level with no calibration brief
- a topic with no questions at one of its levels
- whether each level has enough questions, measured against the targets in [question-types.md](question-types.md#levels): any level where too few facets have a question of a type that level's mix will ask for, and any level with fewer than two questions per in-scope facet
- sync status, including anything not yet pushed

Notes are not problems. The question targets, for example, are a guide for authors. By default `doctor` only counts the notes. `--verbose` lists them.

```
./study doctor
./study doctor --verbose      # also list notes, e.g. topics with no questions at a level
```

Exits `1` if it finds problems.
