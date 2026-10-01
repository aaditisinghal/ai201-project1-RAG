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

Produced by `run_eval.py::main` (file: `results/run_2026-09-28_0037_before.md`).
Retrieval is `store.py::search` over chunks from `chunker.py::split_documents`.
Corpus `advice_threads`, top-k 5, cutoff 0.73, 3 runs per question, cache off.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunks contain the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Every chunk ≥100 chars, first line THREAD: | 0 failing | 0 failing | 0 failing | 0 failing | MET |
| 5. Source: line names the file with the expects phrase | 5 of 5 | 3/5 | 3/5 | 3/5 | MISSED |

Criteria 1, 3 and 4 are deterministic (retrieval, the gate and the chunker do
not change between runs), so their three columns are identical. Only criteria 2
and 5 depend on the generated text.

**Real output, criterion 1** (`run_eval.py`, question 2, run 1):

```
Best distance: 0.6295 (passed the gate)
Sources retrieved: thread_commuting.txt, thread_late_work.txt, thread_laundry_timing.txt, thread_printing.txt, thread_winter_advice.txt
```

**Real output, criterion 2 and 5** (`generate.py`, question 4, run 2 and run 1):

```
The student found that their verbal yes "didn't survive a staff change" (thread_transfer_credits.txt).
```
```
The verbal yes did not survive a staff change.

Source: `thread_transfer_credits.txt`
```

**Real output, criterion 3** (`run_eval.py::check_out_of_scope`):

```
| What is the capital of Mongolia? | 0.948 | refused |
| How do I change the oil in a diesel engine? | 0.930 | refused |
| Who won the 1994 World Cup? | 0.952 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.828 | refused |
| How do I write a for loop in Rust? | 0.871 | refused |
```

**Real output, criterion 4** (`check_chunks.py`, using chunks from `chunker.py::split_documents`):

```
<paste the exact line it printed, for example: 23 chunks, 0 failing criterion 4>
```

## Verdicts

1. **Criterion 1: MET.** The right thread was in the top 5 for all five questions in all three runs (5/5, target 4/5). Retrieval is deterministic, so three runs add no information here.
2. **Criterion 2: MET.** All 15 answers named a source file, in varying formats. Gate refusals are excluded, as my reason in `criteria.md` says.
3. **Criterion 3: MET.** The gate refused 5 of 5 out-of-scope questions (best distances 0.828 to 0.952 against a 0.73 cutoff).
4. **Criterion 4: MET.** `check_chunks.py` reports 0 failing.
5. **Criterion 5: MISSED.** Read plainly, the criterion requires the file on the `Source:` line. Only 3 of 5 answers per run had one (9 of 15 overall). I did not count inline citations like `(thread_group_project.txt)`, because the criterion names a `Source:` line. Every answer named the correct file (15 of 15), so what failed was citation format, not citation accuracy.

Two of these MET verdicts are close to free. Criterion 1 checks the top 5 of only 23 chunks, and criterion 4 checks structure, not whether a chunk can answer anything. I say more in "What I'd Do Differently."

## Diagnoses

**Criterion 5, stage: generation.** Question 5 ("What will most instructors adjust…") got its correct citation in italic parentheses in all three runs, and the other questions switched between `Source:` lines and inline citations from run to run. The retrieved chunks were right in every case, so retrieval and chunking are not the cause. The grounding instruction tells the model to name the file but does not fix the format of the citation, and the model chooses a different format from run to run. The criterion also assumed a format that nothing enforces.

No other criterion was missed.


## The Improvement

**Change:** I tightened `GROUNDING_INSTRUCTION` and the final line of
`build_prompt()` in `generate.py` to require the answer to end with exactly
one line reading `Source: <filename>`, with no other citation anywhere in the
answer. Previously the instruction only said "name the document," with no
required format.

**Why this change and nothing else:** criterion 5 was MISSED (3/5 in every
"before" run) because the model cited the correct file in inconsistent
formats — a `Source:` line in some runs, inline parentheses or italics in
others, and no citation line at all for question 5 in all three runs. The
diagnosis pointed at the generation stage, specifically the prompt, since
retrieval returned the correct chunk every time in both runs. This is the only
change I made. Chunking, retrieval, the 0.73 cutoff, questions.py and
criteria.md are untouched.

**Predicted risk (asked Claude "I'm going to tighten the grounding prompt to
require a Source: line to fix inconsistent citations — tell me why that might
not work"):** <paste what it actually said, or summarize in one sentence:
e.g. "it warned the model might still vary wording around the Source: line,
or might add a Source: line even to a refusal where there's nothing to cite."
Note which of these did or didn't happen below.>

### Run Log — After

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunks contain the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Every chunk ≥100 chars, first line THREAD: | 0 failing | 0 failing | 0 failing | 0 failing | MET |
| 5. Source: line names the file with the expects phrase | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |

**Real output** (`generate.py::answer_from_chunks`, question 2, run 1 of the
after log, `results/run_2026-10-01_1137_after.md`):

```
It gets dark by 4:30 in December.

Source: thread_winter_advice.txt
```

**Did it help?** Yes. Criterion 5 went from 3/5, 3/5, 3/5 (before) to 5/5,
5/5, 5/5 (after) across all three runs. Every one of the 15 answers in the
after run ends with an exact `Source: <filename>` line naming the correct
file, compared to 9 of 15 before. No other criterion moved: criteria 1, 3 and
4 are deterministic (unaffected by a prompt-only change), and criterion 2 was
already MET in both runs. Distances across all five questions are identical
between the before and after runs (e.g. 0.1829, 0.6295, 0.2649, 0.5144,
0.3704), confirming retrieval was not affected.

## What's Still Broken

- **Citation format fixed, not citation accuracy.** The improvement made the
  model format its source consistently, but nothing checks that the named
  file is actually where the fact came from. If the model ever misattributes
  a fact to the wrong chunk, the Source: line would still look correct. I
  would add a check that compares the cited file against which chunk the
  answer's key phrase actually appears in.
- **Multi-source answers aren't handled.** The instruction assumes one
  source per answer. If a future question genuinely needs two documents, the
  format has nowhere to put a second file.
- **The December question only clears the gate because I moved the cutoff to
  0.73.** Its own thread, `thread_winter_advice.txt`, ranked second in
  retrieval (0.660) behind an unrelated printing thread (0.630). Whole-thread
  chunking mixes topics in multi-reply threads, and I did not test per-reply
  chunking as an alternative, because this unit allows only one change.
- **The gate is only tested against far-off-topic questions** (best distances
  0.828–0.952). Near-miss questions, like ones about tuition or campus
  services not covered in this corpus, are untested, so I don't know how the
  cutoff behaves closer to the boundary.
- I stopped here because the unit's rule is one change, measured properly,
  and I wanted to measure this one cleanly rather than confound it with a
  second change.

## What I'd Do Differently

- **Criterion 5** assumed a citation format ("the Source: line") without
  first confirming the system produced one consistently. I would write it as
  "the answer text names the file containing the expects phrase," and treat
  citation *format* as a separate criterion, since format and accuracy turned
  out to be two different failure modes.
- **Criterion 1** is close to a given: the correct thread only has to appear
  anywhere in the top 5 of 23 total chunks (22% of the corpus), and it did in
  all three before-runs with no improvement needed. I'd tighten it to
  "the correct thread is ranked first or second," which the December
  question (ranked second behind an unrelated thread) would actually test.
- **Criterion 4** checks chunk structure (length, a THREAD: title) but not
  whether a chunk can answer a question on its own. I would add a criterion
  like "for each test question, the chunk holding the answer contains it
  within a single reply," which the diluted multi-topic chunks (like the bike
  thread) might fail even though they pass the current structural check.

