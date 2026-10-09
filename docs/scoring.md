# Scoring and scheduling

## Three separate things

These are recorded separately and never combined.

| Field | What it is | What it affects |
|---|---|---|
| `predicted` | how well the user expects to do, in their own words, before answering | the calibration metric only, **never** scheduling |
| `measured` | the score for the first answer, before any teaching | scheduling and metrics |
| `state.json` | built from `measured` | what comes up next |

## The score scale

Scores run from 1 to 5. The score is given **before any teaching**.

| | Meaning |
|---|---|
| 1 | No usable answer |
| 2 | A fragment, **or a confident wrong answer** |
| 3 | The core idea, but missing the why or the cost |
| 4 | Correct, and names the tradeoff |
| 5 | Correct, with numbers, and raises a failure mode or limit unprompted |

A confident wrong answer scores **2, not 3**. Mixing two ideas up is worse than not knowing, because someone who is sure of a wrong answer will build on it.

The scale is the same at every level. The **bar** for each score changes with level. See [levels.md](levels.md).

## What gets scored

Everything is scored, stored and scheduled per sub-topic: `pack / topic / subtopic`. A topic's strength is worked out from its sub-topics and never stored.

Problems are scored separately, as `pack / problem:<slug>` with `subtopic: overall`. A problem score never adds to a topic's strength.

A topic can also be scored as a whole, with `subtopic: overall`. Once any of its sub-topics has a score, the `overall` row drops out of the queue.

## Strength

`strength` is the **latest** score for a sub-topic. It is not an average, so an old 5 does not cancel out a fresh 2. Older scores stay in the ledger and feed the trend chart, but do not affect current state.

If one sub-topic gets several scores on the same day (for example, several quiz questions on it), they count as **one rep**, scored as **their mean, rounded half up**. Eight questions about caching on one day is one piece of evidence, not eight.

Each row stores the full time it was recorded. Scheduling only uses the date, so all questions on the same day count as one rep, however far apart they were asked.

## How much evidence there is

Every sub-topic tracks `reps`: the number of **days** it has been scored on. This limits how far ahead it can be scheduled:

| reps | Longest interval |
|---:|---:|
| 1 | 3 days |
| 2 | 10 days |
| 3+ | no limit beyond the 120-day ceiling |

One good answer is not much evidence. Without this limit, one lucky 5 on the first day would not come back for 6 days.

## Interval

The interval is the number of days until a sub-topic is due again. It uses the **measured** score and the previous interval, in the style of SM-2:

- **1-2**: back to 1 day
- **3**: previous × 1.6, at least 2 days
- **4**: previous × 2.2
- **5**: previous × 3.0
- Multiplied intervals are rounded half up to whole days
- The first score counts the previous interval as 2 days
- The `reps` limit above applies next, then the 120-day ceiling
- `due = last_measured + interval`

Problems skip the `reps` limit, because one problem covers a lot of ground. The 120-day ceiling still applies.

**A topic is due** on the earliest due date of its sub-topics.

The multipliers are common practice, not taken from research. See the *Contested* section of [research.md](research.md).

## Calibration

`predicted` is asked per question, before the answer, on the same 1-5 scale. If the user does not give one, the field is left out. It feeds one metric, **mean signed error: `predicted - measured`**, tracked over time. A positive number means the user is overconfident.

## Test first, then teach

Every session type tests **before** it teaches. Trying and failing before being taught helps people remember more than studying the answer does. It also means the score shows what you knew before the session. Source: [research.md](research.md), *Retrieval before instruction*.

## What follows an answer

| First score | What follows | Why |
|---:|---|---|
| 4-5 | One line: a confirmation, or the limit they did not mention. | More guidance at this level gets in the way (the expertise reversal effect). |
| 3 | Name the specific gap. | The understanding is there. Only the missing piece is needed. |
| 1-2 | A **worked example**, then a re-test on a **different version** of the question later in the same session, recorded as `post`. | A 1-2 means there was nothing to build on. Reading a correction feels like learning but often is not. |

Source: [research.md](research.md), *Some prior knowledge is needed first* and *Feedback needs a second attempt*.

## What `post` means

`post` is the re-test score after teaching. It **never affects scheduling**. `measured` stays as the first score and is the only one that sets the interval.

`post` tells you whether the **teaching** worked, not how much the learner knows. A 2 that stays a 2 after a worked example means the explanation missed, or the sub-topic needs a full `learn` session rather than a quiz question. A 2 that becomes a 4 means the user had not seen it before. The next rep shows whether it stuck. If a sub-topic is at 2 or below, and has either been scored on only one day or had its last `post` at 2 or below, `./study plan` offers a `learn` block for it.

## What each session can record

| Session | Can record |
|---|---|
| `quiz` | one sub-topic score per question, with `qtype` |
| `practice` / `review` | a `problem:` score, plus sub-topic scores **only** for sub-topics the problem directly tested, up or down |
| `learn` | a score before teaching, and optionally `post` after it |
| `study` | whatever its blocks record |

**A problem score never raises a topic.** It measures how you did on that problem. It can add sub-topic scores for sub-topics the problem really tested, including low ones. It never raises a topic's strength on its own.

**Nothing is inferred from related topics.** A session only scores what it directly tested, in the direction the answer showed. A topic does not go up because a related one went well. Anything not tested gets no score.

## Which block a sub-topic gets

`./study plan` picks blocks by strength. See [techniques.md](techniques.md) for the full list. Source: [research.md](research.md), *Worked examples stop helping as you improve*.

| Strength | Block |
|---:|---|
| 1-2 | a `learn` block if it has been scored on only one day or its last `post` was 1-2, otherwise a faded worked example |
| 2 | contrasting cases, for a confident wrong answer |
| 3 | diagnostic inversion: symptoms given, cause asked for |

Full problems do not depend on strength. `./study plan` adds one when there is time. `./study status` suggests a problem session once four or more sub-topics are at 4 or above.

## Fair questions

A question may only ask for what the pack teaches: what is on the card, or what follows from it by reasoning the card supports. If a question needs material the card does not cover, its score means nothing.

If a question is worth asking and the card does not cover it, **fix the card**.
