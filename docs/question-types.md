# Question types

Each question has one of five types. Scores are kept per type, so you can tell whether you are missing the knowledge or failing to apply it. Those need different fixes.

## Why not just recall

Retrieval practice helps you answer questions like the ones you practised (near transfer). Whether it helps you apply knowledge to new, complex problems (far transfer) is disputed. See "Contested" in [research.md](research.md).

Past junior level, the usual failure is the **inert knowledge problem**: you know the thing but do not notice that a situation calls for it. So recall questions are a small part of the mix at higher levels.

## The ladder

### `recall`: what is it

Plain retrieval. One line.

> What is the boundary defect in a fixed-window counter?

Cheap to write and answer, but tells you the least. Needed at graduate level, where people are still learning the vocabulary. Of little use at staff.

### `discrimination`: which is it, and what separates them

Two or more things that are easily confused. The answer names what tells them apart.

> Cache-aside and write-back get confused constantly. What axis actually separates them?

This catches **conflation**, which recall cannot. Conflation produces confident wrong answers, which score 2. Write these as pairs that look alike on the surface but work differently underneath.

### `judgment`: does this change the answer

A design context, then a new piece of information. Does an option become more or less appropriate?

> You are leasing rate-limit tokens locally across 500 nodes. New information: this tenant's limit is 10 per minute. Does local leasing become more or less appropriate?

The answer is `-2 -1 0 +1 +2` plus a short reason, so a hard question needs little typing.

End with "More or less appropriate?" or whichever variant fits, so the candidate knows whether you want the rating, the fix, or both. See [The ask](#the-ask).

This is adapted from script concordance testing in medical education, which tests reasoning under uncertainty rather than knowledge. **This is the type aimed at far transfer.**

A good judgment question has a defensible answer that is not obvious. Sometimes the right answer is 0: "it depends, and here is what on". If the new information settles it trivially, the question is really recall.

### `diagnostic`: symptom to cause

Give what was observed and ask for the mechanism.

> p99 tripled, error rate flat, cache hit rate unchanged. Most likely cause?

This runs the reasoning backwards. Designing and diagnosing are different skills. You need both at work and in interviews.

### `numeric`: put a number on it

> 500 nodes, 10-token leases, a limit of 1,000,000. Overshoot as a percentage?

One number, no prose. It targets a common habit: reasoning correctly but never doing the arithmetic. The answer should show the working, so a wrong number can be traced to either an arithmetic slip or a wrong model.

## The ask

**End the stem with a clear ask.** The person answering cannot see the type, so the wording is the only thing telling them which of several reasonable questions you mean. If you grade an answer against a question the stem did not ask, the score is meaningless. You have marked the candidate down for answering the question they were given.

The phrasings are not interchangeable:

| What you want | The ask | Type |
|---|---|---|
| what tells confusable things apart | "what separates them?" | `discrimination` |
| the cause behind a symptom | "most likely cause?" | `diagnostic` |
| a test that tells two candidate causes apart | "what would you check to tell which?" | `diagnostic` |
| whether new information changes a decision | "more or less appropriate?" | `judgment` |
| a figure, with the working | "how much, and what is that in ...?" | `numeric` |

**"What separates them" belongs to `discrimination`.** If a `diagnostic` offers two candidate causes and wants a test to tell them apart, it has to say so. Asked as "what separates them", the correct answer is the conceptual difference between the two. That is a different question from the one you meant.

If a stem lists candidates, say what they are candidates for. "Two candidates" on its own reads as two scenarios to consider.

## Multiple choice

Use it rarely, and only where **the wrong options are the content**. For example, choosing between five named consistency models, where telling the options apart is the skill. Everywhere else, use free recall. With multiple choice you can recognise the answer and think "I knew that" without ever recalling it. Recognition is thought to be the main thing that blocks transfer.

## Levels

Every question declares which levels it suits:

```toml
levels = ["senior", "lead", "staff"]
```

Question type and level are related but not the same. A `judgment` question for graduates exists: it has a smaller design context and a clearer answer. A staff-level `recall` question can exist too, but it is usually a wasted slot.

The target mix of question types in a draw, per level:

| Level | recall | discrimination | judgment | diagnostic | numeric |
|---|---:|---:|---:|---:|---:|
| graduate | 40% | 30% | 15% | 10% | 5% |
| mid | 25% | 30% | 25% | 10% | 10% |
| senior+ | 10% | 25% | 35% | 15% | 15% |

These numbers live in `studykit/balance.py`. `select.qtype_weights` starts from them and `./study doctor --verbose` checks each level against them. If you change them there, update this table too.

The mix is only a starting point. As scores are recorded, types you answer well are asked less and weak types more. `PRIOR_STRENGTH` in `studykit/select.py` sets how much evidence that takes.

**Each level has its own pool of questions.** A question is only in the pools for the levels it lists. Each pool is checked against its own row in the table above. A pack can hit the targets overall while its graduate questions are 90% recall. This happens if you only check the overall numbers.

**Write for coverage, not for share.** A draw goes through the due facets and asks each one for a question of the type the mix wants. So what limits a type is how many facets have a question of that type at all. Twenty judgment questions on one facet raise judgment's share of the pool but give a draw nothing new. Too many of one type is not a problem either: `qtype_weights` sets the share, so extra recall questions are just drawn less often. The number to raise is how many facets have one question of each type. `doctor` flags a level when a type is in fewer than 80% of the facets its mix needs.

`senior+` is one row because the target mix is the same for senior, lead and staff. What separates those levels is the calibration brief, not the questions.

`./study progress` shows your average score per type. A high `recall` with a low `judgment` means you know the material but are not applying it.

## Stems are self-contained

The topic is never named when a question is asked. The reason is in [research.md](research.md), under interleaving and spacing.

So the stem has to carry everything needed to set the frame, written as a scenario rather than a label.

> What should an error response contain besides a status code?

This cannot be answered. HTTP requires nothing beyond the status code, so a candidate who reads it as a protocol question is right to say "nothing". A candidate who reads it as a design question gives a different answer. The two readings do not overlap.

> You are designing a JSON API. A mobile client calls it, a third-party integrator also calls it, and your support team fields tickets about it. A request fails. Beyond the status code, what goes in the response body?

This tests the same knowledge, with the frame set and no hint at the answer. "This is an API design question" would also set the frame, but it names the topic and does half the recall for them.

A `judgment` stem with a time constraint has a second frame to set: **design or lever**. "Prices must be visible within 60 seconds" reads as a standing requirement to one candidate and as an incident to another. The right answers do not overlap: change what is cached, or fire the purge and accept the thundering herd. Say which one you are asking for.

The test: if a stem needs a topic label to be answerable, it is missing a sentence.
