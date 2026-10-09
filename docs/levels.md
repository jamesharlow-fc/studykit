# Levels

`graduate` · `mid` · `senior` · `lead` · `staff`

Your level is one setting. It changes three things.

## What your level changes

**1. The mix of question types.** Your level sets a target mix, listed in [question-types.md](question-types.md#levels). Graduates get more recall and discrimination. Seniors get more judgment.

The mix is only a starting point. Every score you record moves it: a type you answer well comes up less, and a weak one comes up more. After enough sessions the mix fits you rather than your level. `PRIOR_STRENGTH` in `studykit/select.py` sets how many scores that takes.

Each question also lists the levels it was written for. Within a topic, questions for your level come first. Once you have seen those, the other questions in the topic can come up too. No question is hidden because it was written for another level, so if there are few questions at your level you get harder or easier ones, not none.

**2. Which topics and problems you see.** Topics and problems also list their levels. This is a hard filter. `consistency-models` starts at senior. `multi-region-checkout` starts at lead. A topic outside your level does not appear in your queue, your coverage percentage or your gaps.

This filter is about relevance, not difficulty. A graduate is never asked about multi-region failover. A question written for graduates only comes up less often for a senior.

**3. The bar you are judged against.** Each pack has a **calibration brief** for each level. It says:

- the bar an answer must clear
- what to assume you already know
- what to push on
- what not to demand

`./study plan` returns it.

The 1-5 scale is the same at every level. A 4 always means "correct, with the tradeoff named". What changes is what counts as the tradeoff, and how much you are expected to say without being asked.

## Seeing the briefs

```
./study levels
```

This prints the bar at every level for each enabled pack, with its `assume`, `push_on` and `avoid`. `avoid` is what stops a graduate losing marks for not giving a cost model.

## Changing it

```
./study config set level lead
```

State and metrics rebuild straight away. The ledger does not change. Each row records the level it was scored at, so your history still makes sense after you change level.

If you move up:

- **Your coverage drops**, because topics that were out of scope are now in it.
- **Your existing scores stay the same.** They are measurements. A sub-topic scored 4 as a senior still reads 4 as a lead. Your next score, against the new bar, is what moves it.

Moving down does the opposite. Scores for topics now out of scope stay in the ledger.

## Choosing one

Pick the level you are being **assessed** at, not the one you are comfortable at. If you are preparing for an interview, use the level of the role. If you are keeping your knowledge up for your current job, use your current level.

Studying one level above your own is hard but useful. Studying one level below teaches you little.

## Authoring for levels

The TOML is in [authoring-packs.md](authoring-packs.md#packtoml).

- **Tag a question with the levels it was written for, not every level that could answer it.** "What does a load balancer do" is a graduate question. A staff engineer can answer it, but it is still a graduate question. Tagging it up to `staff` does not hide it from anyone. It just takes a staff slot it should not have.
- **A topic's lowest level is the lowest level at which its simplest sub-topic is fair.** This tag decides whether someone sees the topic at all, so it matters most.
- **`avoid` matters more than `bar`.** Saying what not to demand is what makes the lower levels usable.
- **Write enough questions for each level, not just enough for the pack.** A pack can hit its target mix overall while its graduate questions are nearly all recall. And many questions of one type on a single sub-topic do not help a session that needs that type across several. `./study doctor --verbose` reports which types each level has too few of across its sub-topics, how many questions each sub-topic has, and any topic with no questions at one of its levels.
