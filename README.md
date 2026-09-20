# The Unofficial Guide

Munsakad — `campus_life` corpus.

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
> or remove them.

---

# Unit 1

## What This Does

This is a question-answering system over `campus_life`, a corpus of 88 short
posts about student life at a university — dining halls, dorms, courses, and
the administrative rules nobody explains properly. Ask it something like "is
the housing lottery random?" or "how much does laundry cost in Aldridge
Hall?" and it retrieves the post(s) closest in meaning to the question, then
has a model write a short answer grounded only in that text, naming the file
it came from. Ask it something the corpus doesn't cover — the capital of
Mongolia, a Rust for-loop — and it refuses instead of guessing.

## Chunking Strategy

**Chunk size:** 700 characters (a cap, not a target)
**Overlap:** 100 characters (only used if a document is split)

campus_life's documents are one-topic notes — "On the housing lottery,"
"Aldridge Hall — what it's actually like" — one to three short paragraphs
each. I read through the admin, housing, course, and dining posts in
Milestone 1 and measured every document in the corpus: the longest is 550
characters, the shortest 179, and none comes close to the starter's default
800-character window. That's why the starter's fixed-size chunker turns 88
documents into exactly 88 chunks without ever cutting one — there's nothing
in this corpus for an 800-character window to cut.

Rather than just accepting that as a coincidence of the default number, I
rewrote `split_documents` in `chunker.py` to group each document's paragraphs
up to a 700-character cap, splitting only between paragraphs and carrying
the last paragraph forward (via `CHUNK_OVERLAP`) if a split does happen. On
this corpus the effect is the same — one document, one chunk — but now it's
because I decided a whole post is the right unit of retrieval, not because a
round number happened to be bigger than everything I fed it. A sentence like
"The good: closest building to the science quad, four minutes to a 9am lab"
means nothing without knowing which building it's about, so keeping each
post's paragraphs together preserves the antecedent that makes the chunk
answerable on its own. The 700-character cap and 100-character overlap exist
so the same function does something sensible — split on paragraph
boundaries with continuity across the cut — if a future document (or a
different corpus) doesn't fit in one chunk.

I did not change my mind partway through; the paragraph-grouping approach
was the first thing I tried, because Milestone 1's read-through made clear
before I wrote any code that these documents were already atomic.

## Sample Chunks

Printed with `python app.py chunks -n 5`, which samples spread across the
corpus.

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_biol_160.txt#0` — produced by: `chunker.py::split_documents`

```
BIOL 160 Cell Biology

I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved.

Expect 9 to 11 hours a week, the heaviest first-year course by reputation.

The one piece of advice: the unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.
```

**Chunk 3** — source: `course_hist_118_workload.txt#0` — produced by: `chunker.py::split_documents`

```
Workload for HIST 118 Modern World History

People keep asking so: a lot of reading, about 120 pages a week, but no problem sets. That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 4** — source: `dining_pellew_dining_hall_followup.txt#0` — produced by: `chunker.py::split_documents`

```
Re: Pellew Dining Hall

Adding to what people have said about Pellew Dining Hall. The wait figure of 12 to 18 minutes at peak matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

Also worth saying: the furthest hall from anywhere, next to the athletics centre. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_innisfree_hall.txt#0` — produced by: `chunker.py::split_documents`

```
Innisfree Hall — what it's actually like

Transferred in last year, so take this with a grain of salt. Built 1991, renovated 2022. Rooms are doubles arranged as pairs sharing one bathroom between two rooms.

The good: the shared-bathroom-between-two-rooms arrangement is the best compromise on campus.

The bad: no air conditioning, which matters for the first three weeks of September.

Laundry costs $1.75 wash, $1.75 dry, app-based. On noise: moderate; the building is L-shaped and the short wing is much quieter.
```

## Sample Answer

**Question:** How much does laundry cost in Aldridge Hall?

**Answer:**

```
Laundry in Aldridge Hall costs $1.75 to wash and $1.50 to dry (housing_aldridge_hall.txt and housing_aldridge_hall_laundry.txt).

Sources retrieved: housing_aldridge_hall.txt, housing_aldridge_hall_laundry.txt, housing_calder_annexe.txt, housing_innisfree_hall.txt, housing_innisfree_hall_laundry.txt
```

**My relevance cutoff:** 0.6 (the course default — measuring my own corpus
confirmed rather than moved it).

I ran my five test questions and the five in `OUT_OF_SCOPE` through
`python app.py retrieve` and recorded the best (lowest) distance for each.
The two groups didn't just have a gap, they didn't come close to touching:

| Question | In corpus? | Best distance |
|---|---|---|
| Is the housing lottery actually random? | yes | 0.248 |
| What time does Halden Hall dining hall close? | yes | 0.266 |
| How much does laundry cost in Aldridge Hall? | yes | 0.247 |
| When do I have to declare my major? | yes | 0.292 |
| How many hours a week should I expect to spend on CS 210 outside of class? | yes | 0.249 |
| What is the capital of Mongolia? | no | 0.825 |
| How do I change the oil in a diesel engine? | no | 0.934 |
| Who won the 1994 World Cup? | no | 0.886 |
| What is the recommended dosage of ibuprofen for a headache? | no | 0.844 |
| How do I write a for loop in Rust? | no | 0.896 |

In-corpus questions topped out at 0.292; out-of-scope questions bottomed out
at 0.825. The starter's default of 0.6 sits well inside that gap, so I kept
it rather than moving it — there was nothing close enough to either edge to
argue for a different number.

## How I Used AI

<!-- REVIEW BEFORE SUBMITTING: these two entries describe what actually
     happened in the session where this was built with Claude. Replace or
     edit them so they honestly reflect what you reviewed and decided
     yourself — this section is graded as your own account, and submitting
     it unedited would misrepresent that. -->

**1.** After reading through a sample of `campus_life` documents in
Milestone 1 and noticing they were short, single-topic posts, I asked Claude
to replace the starter's fixed-size chunker with a strategy that fit that
shape. It measured every document's length first (179-550 characters, all
under the 800-character default), then wrote a paragraph-grouping chunker
that only splits between paragraphs and carries the last paragraph forward
on a split. I ran `python app.py chunks -n 5` myself and read the output
before accepting it, to check the chunks actually stood alone the way the
milestone asks.

**2.** For Milestone 4's relevance cutoff, I asked Claude to measure it
empirically rather than pick a number — run all five test questions and the
five `OUT_OF_SCOPE` questions through `python app.py retrieve` and report the
best distance for each. It came back with in-corpus distances of
0.247-0.292 and out-of-scope distances of 0.825-0.934, and suggested keeping
the default 0.6 since it already sat in that gap. I checked the raw numbers
in the table above myself rather than taking the recommendation on its own.

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
