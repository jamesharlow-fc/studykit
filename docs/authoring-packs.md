# Authoring a pack

A pack is content only. It says what there is to learn. It says nothing about scheduling, scoring or how a session runs, so you can share it without sharing anyone's history.

## Layout

```
<name>/
  pack.toml                    manifest: levels, areas, topics, problems, calibration
  cards/<topic>.md             one knowledge card per topic
  questions/<topic>.toml       one question bank per topic
  problems/<slug>/prompt.md    what the candidate sees
  problems/<slug>/notes.md     interviewer notes, never shown before an attempt
```

The file names matter. studykit finds `cards/<topic id>.md` and `questions/<topic id>.toml` by the topic's id. Nothing in the manifest points to them.

studykit looks for packs in two places:

- `packs/` in the checkout, for the packs that ship with it.
- `~/.studykit/packs/`, for packs you install.

Put a pack directory in either and it is available. Installing to `~/.studykit/packs/` keeps `git status` clean. The pack also syncs to your other machines with your data. If two packs have the same name, studykit stops with an error naming both directories.

To choose which packs are in rotation, use `./study packs enable <name>` and `./study packs disable <name>`. To replace the whole list, use `./study config set packs a,b`. `./study packs` shows where each pack was loaded from.

## pack.toml

```toml
[pack]
name = "system-design"
title = "System design"
description = "One line, shown in `./study packs`."
levels = ["graduate", "mid", "senior", "lead", "staff"]
areas = ["caching", "storage", "consistency"]

[calibration.senior]
bar = "Tradeoff named and defended, a number attached, and a failure mode volunteered."
assume = "Has operated systems in production."
push_on = ["why, not what", "what breaks at 10x", "what it costs"]
avoid = "Do not accept a mechanism with no cost attached."

[[topic]]
id = "caching"
title = "Caching"
area = "caching"                 # must be in pack.areas
prefix = "ca"                    # seeds question ids: ca-001
levels = ["graduate", "mid", "senior", "lead", "staff"]
subtopics = ["read-strategies", "write-strategies", "eviction", "hot-key"]

[[problem]]
slug = "url-shortener"
title = "URL shortener"
areas = ["storage", "caching"]
levels = ["mid", "senior", "lead", "staff"]
minutes = 45                     # what the composer budgets for it
```

A topic or problem with no `levels` gets the pack's levels.

Write a calibration brief for every level. It is optional. Without one, the agent uses its own judgement, which is less consistent. `./study doctor --verbose` lists the missing ones.

## Sub-topics

Sub-topics are what gets scored, scheduled and reported. Every topic must declare at least one.

- **Have four to seven per topic.** With fewer, the topic is too broad to schedule well. With more, each one is tested too rarely for its score to mean anything.
- **Name them after something you can know.** `stampede-penetration` is something you can be right or wrong about. `overview` is not.
- **Each one must be testable on its own.** If you cannot write a question that tests one without also testing another, they are one sub-topic.
- **Add rather than rename.** A new sub-topic starts unmeasured, so it appears in everyone's queue straight away. Renaming one breaks it: the history under the old name is orphaned.

## Cards

A card is plain markdown with no frontmatter. Start with an H1 and a one-line summary. Do not state the card's area or levels. `./study card` prints those above the card, from the manifest.

**Give each sub-topic its own section**, with the sub-topic id as the heading. A question is fair only if the card covers what it asks. With one heading per sub-topic, it is easy to see whether it does.

```markdown
# Caching

**One line:** Keep a copy of expensive-to-fetch data somewhere faster.

## Why it exists
...

## read-strategies
...

## hot-key
...

## Numbers to know
| | |

## Related
- [consistent-hashing](consistent-hashing.md)
```

A card is for answering questions from, not for reading through. So:

- **Prefer tables, named failure modes and numbers to flowing prose.** Prose is harder to be quizzed on.
- **Give the cost, not just the mechanism.** Every technique should say what it costs and when it is the wrong choice.
- **Include numbers.** Without them the card cannot support `numeric` questions, which are the best way to catch vague answers.
- **Name the things people confuse.** Write "These two get confused; what separates them is X". This is what `discrimination` questions test.
- **Use relative markdown links.** No wiki-links, vaults or plugins.

## Question banks

```toml
topic = "caching"

[[q]]
id = "ca-001"
qtype = "discrimination"
subtopic = "write-strategies"
levels = ["mid", "senior", "lead", "staff"]
q = "Cache-aside and write-back get confused constantly. What axis actually separates them?"
a = "They are on different axes entirely. Cache-aside is a READ strategy; write-back is a WRITE strategy. You can run cache-aside reads with write-back writes."
```

- `qtype` is one of `recall` `discrimination` `judgment` `diagnostic` `numeric`. See [question-types.md](question-types.md).
- `subtopic` must be one the topic declares. `doctor` reports it if not.
- `levels` defaults to the pack's levels. Set it on purpose. Each level you list is a pool the question competes in. Tagging a recall question up to `staff` adds to staff's share of recall without giving a staff candidate anything useful. `doctor` reports the mix per level.
- Ids are permanent and never reused, because the ledger refers to them. Two questions with the same id are an error.
- Numeric ids like `ca-001` are for pack authors. Questions banked during a session get `ca-u<hash>`, where the hash comes from the question text. This means a session never takes an id a pack will want later. Two machines that bank the same question offline also get the same id.
- **Do not store exposure history here.** `shown` and `reps` are worked out from the ledger.

**A stem must make sense on its own.** See [question-types.md](question-types.md#stems-are-self-contained). To test one, read it a different way and answer that. If your answer is reasonable but the banked answer would mark it wrong, the stem needs more context.

Write the answer as what a **strong** answer contains, not the minimum to pass. The agent scores against it and uses it to tell the user what they missed. "Less, -2, because the floor on overshoot is one lease per node" is better than "Less appropriate".

### Figures are derived, not asserted

If a question states a figure, it needs a `derivation`: the arithmetic, one `name = expression` per step.

```toml
[[q]]
id = "cd-u25bb12fb"
qtype = "numeric"
q = "2M daily users make 40 API calls each. Origin is 140 ms RTT, TLS 1.3, no edge termination. How much of the day is pure connection setup?"
a = "80M requests. Each is cold: 3 x 140 = 420 ms, of which 280 ms is setup. Aggregate 22.4M seconds, about 260 human-days of waiting per day."
derivation = [
  "users = 2_000_000",
  "calls_per_user = 40",
  "requests = users * calls_per_user",
  "rtt_ms = 140",
  "setup_ms = 2 * rtt_ms",
  "aggregate_s = requests * setup_ms / 1000",
  "aggregate_days = aggregate_s / 86400",
]
```

`doctor` runs the derivation and checks that every figure in `q` and `a` matches the result of some step. `bank add` refuses a new question that states figures without a derivation. If a figure is wrong, no answer can be correct. The score still goes into the ledger permanently.

- You can use numbers, the operators `+ - * / // % **`, and `abs round min max log log2 log10 exp sqrt ceil floor`. Nothing else runs.
- Include the **inputs** as steps, not just the results. `rtt_ms = 140` is what lets the check match the 140 ms in the stem.
- A figure matches if it is the result rounded to the precision it is written at, so `about 30x` matches 29.33. Scientific notation with one significant figure, such as `1e-19`, only claims the order of magnitude.
- Only figures that look like quantities are checked: a number with a unit, a scale word, a thousands separator or an exponent, a ratio like `1 in 5`, or any value of 1000 or more. Plain counts like `TLS 1.3` and `the next 3 nodes` are skipped, so a figure that looks like a plain count is not checked. You have to get it right yourself.
- A lowercase `m` after a number is rejected, because it could mean million or minutes. Write `M`, `million` or `minutes`.
- Units are not converted. If the text says both `280 ms` and `0.28 s`, derive both.

`./study bank check --json-text '{"questions": [...]}'` runs the same check without writing anything. Use it **before** a question is asked, not after.

## Problems

A problem is two files, so the notes cannot leak into the prompt.

`prompt.md` is all the candidate sees: a one-line ask and nothing else. Leave it under-specified on purpose, so they have to ask questions.

`notes.md` is never shown before an attempt. `./study problem <slug>` does not return it; you need `--notes`. Include:

- **Hidden requirements**, functional and non-functional, that a good candidate finds by asking.
- **Rough numbers** they should reach, with the arithmetic.
- **Two or three areas to go deep on**, each with what to push on.
- **Signs of a strong answer**: what makes an answer good rather than just correct.
- **Common traps.**
- **Level calibration**, if the problem covers several levels and the bar is clearly different between them.
- **Follow-up questions.**

If a candidate can start designing without asking anything, the prompt says too much.

## Validating

```
./study doctor --verbose
```

`doctor` reports these as problems:

- a topic with no sub-topics
- a topic whose area is not in `pack.areas`
- a question naming a sub-topic its topic does not declare
- a problem missing `prompt.md` or `notes.md`
- a `derivation` that fails to run, or a figure it does not account for
- a figure written with a lowercase `m`
- a ledger row that no longer matches the pack's topics and sub-topics

Duplicate question ids and two packs with the same name stop studykit loading at all.

`--verbose` also prints notes:

- a topic with no card
- a level with no calibration brief
- a question that states figures with no derivation
- a derivation step whose result is never quoted
- a question of another type that uses the `discrimination` ask
- a topic with no questions at one of its levels. This is the most common gap.
- a level that does not have enough questions, spread across enough sub-topics, to supply the mix in [question-types.md](question-types.md#levels)

Then use the pack:

```
./study packs enable <your-pack>
./study plan 25m
```

The quickest way to find a weak card is to be quizzed on it. If you cannot answer a question from the card, fix the card, not the question.
