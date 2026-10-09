# Technique menu

These are the blocks `./study plan` builds a session from. Each one targets a specific weakness and has a rough cost in minutes, so a session can be fitted to your time. A block is chosen because your scores call for it, not by preference. The research behind each is in [research.md](research.md).

| Block | Targets | Minutes | Chosen when |
|---|---|---:|---|
| **Quiz set** | retrieval, discrimination | about 2 per question | anything is due |
| **Learn** | building a facet you do not have yet | 30 | a facet at strength 1-2 has had at most one rep, or its last re-test after teaching scored 1-2 |
| **Faded worked example** | rebuilding a weak facet | 25 | a facet is at strength 1-2. Also the cheaper fallback when **Learn** does not fit |
| **Contrasting cases** | conflation | 10 | a facet is at strength 2 (confident and wrong) |
| **Estimation drill** | quantification | 5 | your average `numeric` score is lower than every other type's |
| **Teach-back** | illusion of explanatory depth | 12 | your `predicted` scores average more than 0.5 above `measured` |
| **Diagnostic inversion** | symptom to cause | 10 | a facet is at strength 3 |
| **Full problem** | far transfer, integration | the problem's length, 30-60 | there is time for it and an unattempted problem at your level fits |
| **Cold re-attempt** | durability | the problem's length, 30-60 | a problem you have attempted before is due again |
| **Card writing** | consolidation | 20 | a never-measured facet's topic has no card, or 20 minutes or more are left over |

No two targeted blocks in one session work on the same facet.

## The less obvious ones

**Learn.** Runs the `learn` protocol on one facet: test it cold, teach only what the test showed was missing, then re-test on variants. It is for a facet with nothing to retrieve yet, where quizzing it again would only measure the same gap.

**Contrasting cases.** Two or three things that look alike on the surface but work differently underneath. You are asked what separates them before anything is explained. This is the direct fix for a score-2 conflation. It works better than a correction, because a correction fixes the wrong answer but not the missing distinction.

**Teach-back.** You explain the concept to a competent engineer who has not met it. People often believe they can explain something until they try. You only find the gaps by trying.

**Faded worked example.** A complete worked example, then a variant with parts removed for you to fill in.

**Full problem.** The only block that tests whether you can combine facets. It is also the only place the **transfer gap** shows up: the difference between how well you know a facet on its own and how well you use it when you have to spot it inside a larger problem.

**Cold re-attempt.** The same problem again, without seeing your previous attempt first. It tests whether you can still do it weeks later, not whether you did it once.

## How a budget is filled

`./study plan <budget>` fills the time from the due queue, weakest first, mixing topics and packs:

| Budget | Typical session |
|---|---|
| 10m | one quiz set of about 5 questions |
| 25m | a quiz set of 8 to 14 questions, plus one targeted block if one fits |
| 45-60m | a quiz set, plus a full problem or up to two targeted blocks (three at 60m) |
| half day | a quiz set, up to three targeted blocks, up to two problems, and card writing if time is left |
| full day | as a half day, with up to three problems |

Some fixed rules:

- The last two minutes are kept for recording.
- A session can run up to 10% over the time you asked for (at least two minutes), because every block cost is an estimate. `plan` reports the planned total.
- A quiz set has at most 14 questions (20 from two hours), and at most 8 when the session includes a problem.
- About 10 minutes are kept for the quiz set before targeted blocks or problems can use the time. A targeted block that does not fit is dropped, and `notes` says so and suggests a longer session. The quiz set always keeps this time because it is how the next weak facet is found.
- If a long session runs out of things to do, `notes` says how many minutes are unallocated and why.

The queue rotates through topics rather than finishing one first, so a 12-question set can come from up to 12 different topics. Every third slot is kept for a facet you have never been tested on, however long the overdue list is. Without that, the weak spots found in your first few sessions would be the only ones ever worked on.

## Overriding it

The same ledger on the same day always gives the same plan. You can change what it picks:

```
./study plan 25m --no-problem       # never include a full problem
./study plan 25m --seed 7           # reproducible selection
./study questions --topic caching   # ignore the queue, drill one topic
./study problem <slug>              # a specific problem
```

Drilling one topic is blocked practice. It feels more effective than mixed practice, but you remember less of it. Use it when you are learning something new, not when you are keeping up something you already know.
