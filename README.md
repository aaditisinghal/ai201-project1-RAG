# The Unofficial Guide

<!-- Aaditi Singhal -->

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them

---

# Unit 1

## What This Does

The Unofficial Guide answers questions about student life from the
`advice_threads` corpus: 23 threads of student replies covering laptops,
roommates, transfer credits, deadlines, winter and more. You ask a plain
question, it retrieves the closest thread, and an answer is written from that
thread with the source file named. Questions the threads don't cover are
refused by a distance cutoff before the model runs.

## Chunking Strategy

**Chunk size:** one whole thread per chunk. The threads in this corpus run
from about 250 to 830 characters. A thread longer than 1,200 characters
(`MAX_CHARS` in `chunker.py`) is split at reply boundaries, never mid-reply,
and a piece shorter than 100 characters (`MIN_CHARS`) is merged into its
neighbour.

**Overlap:** none for whole threads. If a thread is split, its `THREAD:` title
is repeated on every piece and the last reply carries over into the next piece
as overlap.

**Why:** the replies are short and depend on each other ("Both true.",
"Counterpoint, I sold mine."), so the thread is the smallest unit that makes
sense on its own. The starter's fixed 800-character window cut three threads
mid-reply and produced 26 chunks from 23 documents, including fragments such as
`t.` (2 characters), `nd it's the only reason I got mine back after it was
taken.`, and `) ---`. My chunker produces 23 chunks, one per thread, and
`check_chunks.py` reports 0 chunks failing criterion 4 (at least 100
characters, first line starts with `THREAD:`).

**Trade-off:** a thread like the bike one mixes storage, salt, cost and
registration in one chunk, so it may match a narrow question less sharply. One
chunk per reply with the title prepended is a candidate improvement for unit 2.

**Testing the split branch:** the split-and-overlap logic is implemented and
tested (with the cap lowered to 400 it produced 59 chunks, with titles
repeated), but it never triggers at the real 1,200 cap on this corpus.

## Sample Chunks

Five of the 23 chunks, all produced by `split_documents` in `chunker.py`.

**Chunk 1** — source: `thread_group_project.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: How do you handle a group project where someone disappears?

--- reply 1 (29 votes) ---
Document early. Not to be difficult — because if you go to the instructor in week 10 with nothing written down, there's nothing they can do.

--- reply 2 (22 votes) ---
Most instructors here will adjust individual grades if you raise it before the deadline rather than after. After is too late, consistently.

--- reply 3 (16 votes) ---
Split work into pieces that can be handed off. If one person's part is load-bearing for everyone else, one disappearance sinks it.
```

**Chunk 2** — source: `thread_roommate_conflict.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: Roommate situation isn't working. What now?

--- reply 1 (28 votes) ---
Talk to your RA early, and frame it as 'we need help sorting this out' rather than 'move me'. Room changes are possible but the process starts with mediation and skipping that step slows it down.

--- reply 2 (14 votes) ---
Room changes happen at the semester boundary almost always, and mid-semester only in fairly serious cases.

--- reply 3 (33 votes) ---
Write down specifics before the meeting. 'It's not working' is hard to act on; 'guests four nights a week past 2am' is not.
```

**Chunk 3** — source: `thread_printing.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: Is the printing quota enough?

--- reply 1 (17 votes) ---
For most people yes. $30 is about 600 pages black and white. It's the colour printing that eats it — eight times the cost per page.

--- reply 2 (11 votes) ---
Doesn't roll over between semesters. Print your readings in December rather than losing it.
```

**Chunk 4** — source: `thread_meal_plan_tier.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: Which meal plan tier is right?

--- reply 1 (24 votes) ---
Depends entirely on whether your building has a kitchen. Fenwick has kitchenettes, so people there go down a tier and cook two or three nights. Everywhere else, get the middle tier.

--- reply 2 (19 votes) ---
The highest tier only makes sense if you eat three meals a day in the halls every single day, which basically nobody does past October.

--- reply 3 (11 votes) ---
Remember you can only change it once and only in the first ten days. I waited and got stuck on a plan I didn't use.

--- reply 4 (7 votes) ---
Declining balance rolls within the semester but not between them. Spend it in December or lose it.
```

**Chunk 5** — source: `thread_first_year_regret.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: What do you wish you'd known in first year?

--- reply 1 (41 votes) ---
That the add/drop deadline and the withdrawal deadline are different dates and only one of them is on the calendar everyone reads.

--- reply 2 (28 votes) ---
That you can take a course pass/fail and declare it late — up to week eight. I carried a grade I didn't need to.

--- reply 3 (35 votes) ---
That the writing centre will read a draft for any course, not just writing courses. Free, and the appointments go unbooked.

--- reply 4 (52 votes) ---
Honestly: that nobody is watching as closely as you think. I spent a year worried about looking like I knew what I was doing.

--- reply 5 (17 votes) ---
That your adviser's job is partly to know the exceptions to rules. Ask before assuming a deadline is fixed.
```
 

## Sample Answer

**Question:** How much RAM do students say is worth paying for in a laptop for CS courses?

**Answer:**
```
Students say that 16GB of RAM is the one number worth paying for in a laptop for CS courses.

Source: thread_laptop_specs.txt

```

**My relevance cutoff:** 0.73

The 5 in-corpus best distances range from 0.183 to 0.630. The 5 out-of-scope
best distances range from 0.828 to 0.952. There's a clean gap between 0.630
and 0.828 with no overlap, so 0.73 sits safely in the middle of it.

| Question | In corpus? | Best distance |
|---|---|---|
| How much RAM do students say is worth paying for in a laptop for CS courses? | Yes | 0.183 |
| What time does it get dark on campus in December? | Yes | 0.630 |
| What does the process for a room change start with when a roommate situation isn't working? | Yes | 0.265 |
| What happened to the student who relied on a verbal yes about transfer credits? | Yes | 0.514 |
| What will most instructors adjust if you raise a disappearing group member early enough? | Yes | 0.370 |
| What is the capital of Mongolia? | No | 0.948 |
| How do I change the oil in a diesel engine? | No | 0.930 |
| Who won the 1994 World Cup? | No | 0.952 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.828 |
| How do I write a for loop in Rust? | No | 0.871 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.** "I used Claude to recalculate threshhold and best distance amongst my questions and out of scope."

**2.**"I asked Claude to help debug a GitHub push rejection caused by an exposed API key in .env.example. It walked me through amending the commit, but the amend didn't actually remove the old commit from history — I had to reset back to the last clean commit and recommit. I learned, amend doesn't rewrite older commits in the range being pushed."

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
