# Evidence base

The research behind each rule in the engine. A finding is only listed here if it changes what the engine does.

## Retrieval before instruction

**Finding.** Trying to recall an answer and failing, before being taught, helps people remember more later than studying the material directly. Richland, Kornell & Kao (2009) called this the pretesting effect. It is part of Bjork's desirable difficulties (1994) and the wider testing effect (Roediger & Karpicke 2006).

**What the engine does.** It always tests first. `measured` is the score before any teaching. It is the only field that affects scheduling.

**The limit.** In these studies, teaching comes *straight after* the pretest. The failed attempt prepares the learner for a lesson. It does not replace one.

## Some prior knowledge is needed first

**Finding.** Two sets of studies disagree. Kapur's work on productive failure (2008; Kapur & Bielaczyc 2012) shows that learners who try problems before being taught do better than those taught first, especially at applying it to new problems. Kirschner, Sweller & Clark (2006) argue that giving beginners little guidance overloads them, and that worked examples do better.

What separates the two is prior knowledge. Kapur says productive failure only works when learners already know enough to come up with possible answers.

**What the engine does.** It never skips the first attempt, but what comes next depends on how badly it went. A score of 1-2 means there was nothing to build on, so the agent gives a worked example rather than a paragraph of correction. See the table of what follows an answer in [scoring.md](scoring.md).

**This is also why there are levels.** The same question is a useful challenge for a senior engineer and an impossible one for a graduate. Filtering questions by level applies this finding to what gets asked, not just to feedback.

## Worked examples stop helping as you improve

**Finding.** Beginners learn more from worked examples than from solving problems unaided (Sweller & Cooper 1985). Once the learner understands the topic, worked examples make them perform worse, because reading guidance they do not need uses up working memory. This is the expertise reversal effect (Kalyuga, Ayres, Chandler & Sweller 2003).

**What the engine does.** It picks the block by strength: a worked example at 1-2, a diagnostic question at 3. Guidance drops off as strength rises. At 4-5 the agent gives **one line** of feedback, not a long confirmation.

## Feedback needs a second attempt

**Finding.** Feedback after a failed attempt helps. It helps much more if the learner then recalls the corrected answer themselves, rather than only reading it. Across the testing-effect research, repeated recall with feedback beats feedback alone (Butler, Karpicke & Roediger 2008; Pashler et al. 2005).

**What the engine does.** A 1-2 is re-tested later in the same session on a **different version** of the question, recorded as `post`. Reading a good explanation feels like learning. Only a second attempt shows whether anything stuck.

The re-test must be a different version, never the same question. With the same question, the learner can recognise the answer and think "I knew that" without recalling anything.

## Interleaving and spacing

**Finding.** Mixing topics within a session (interleaving) beats practising one topic at a time, both for remembering and for solving problems. It is what teaches people to tell which idea applies (Rohrer & Taylor 2007; Rohrer, Dedrick & Stershic 2015). Spreading practice out beats cramming. The best gap between sessions grows with how long you need to remember it (Cepeda et al. 2006).

**What the engine does.** Sessions mix topics and never say which topic a question is from. The queue rotates through topics rather than finishing one first, so there is something to mix. Intervals grow with each good score. Only `measured` sets them.

## Feeling fluent is not the same as learning

**Finding.** Learners taught with active methods think they have learned less, while measurably learning more (Deslauriers et al. 2019). Separately, people overestimate how well they can explain how something works until they are asked to do it. This is the illusion of explanatory depth (Rozenblit & Keil 2002).

**What the engine does.** How confident the user says they are never affects scheduling. `predicted` is only used to work out calibration error. When `predicted` runs more than 0.5 above `measured` on average, `./study plan` adds a teach-back block.

## Recognising an answer is not recalling it

**Finding.** At this level, the problem is rarely missing knowledge. It is not noticing that the knowledge applies, which is known as the inert knowledge problem. Multiple choice lets a learner recognise the right answer without recalling it.

**What the engine does.** Questions are free recall by default. Multiple choice is only used where telling the listed options apart is the skill being tested. See [question-types.md](question-types.md).

## Contested

- **Does retrieval practice help with new kinds of problem?** It helps with problems like the ones practised (near transfer). Whether it helps with different ones (far transfer) is disputed. Van Gog & Sweller (2015) argue that the testing effect shrinks as material gets more complex. Karpicke & Aue (2015) disagree. The engine allows for both views. It weights questions towards `judgment`, and treats full problems as the only test of combining ideas.

- **Productive failure or direct instruction first?** This is unresolved. The prior-knowledge limit above is where the two sides come closest, but it is not proven. Results vary with the subject and with how much structure the failure phase has.

- **The best spacing intervals.** The SM-2 style multipliers in [scoring.md](scoring.md) are common practice, not taken from this research. Cepeda et al. support intervals that grow and that scale with how long you need to remember. The specific numbers (1.6 / 2.2 / 3.0) are a guess. If you have data, change them.

## Where each rule is implemented

| Rule | Implemented in | Documented in |
|---|---|---|
| Score first, three fields kept separate | `studykit/ledger.py` | [scoring.md](scoring.md) |
| Intervals, `reps` limit, same-day scores averaged | `studykit/schedule.py` | [scoring.md](scoring.md) |
| Interleaving, block by strength, fitting a session to the time | `studykit/select.py` | [techniques.md](techniques.md) |
| What follows an answer, what `post` means | the skills | [scoring.md](scoring.md) |
| Question types, free recall by default | pack question banks | [question-types.md](question-types.md) |
| Topics filtered by level | `levels` in `pack.toml` | [levels.md](levels.md) |
