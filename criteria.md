# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:** My chunker keeps each campus_life post as one chunk
(see Chunking Strategy), so almost every fact lives in exactly one
self-contained chunk with nothing else to compete against it in the vector
store. I'd expect close to 5 of 5, but I'm leaving one of slack in case a
question's wording overlaps more with a different post than with the one
that actually answers it — two housing posts share almost identical language
("$1.75 wash"), so a laundry question could pull back the wrong hall.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:** This one is 5 of 5, not 4 of 5, because it isn't left
to the model's judgment — `GROUNDING_INSTRUCTION` in generate.py explicitly
tells it to name the filename in every answer, and the prompt itself never
includes chunks without their source. The only way this fails is the model
flatly ignoring an instruction it's given every single time, which is a
different, worse problem than an occasional miss.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

<!-- The five questions are the ones in `OUT_OF_SCOPE` at the bottom of
     `questions.py`, and `run_eval.py` puts them through the gate and writes
     what happened into your run log. Swap them for your own if you'd rather —
     just keep five of them, or the "4 of 5" above has nothing to be 4 of. -->

**Why this target:** When I measured this in Milestone 4, the gap was wide
and clean: my five in-corpus questions had best distances of 0.247-0.292,
and the five OUT_OF_SCOPE questions had best distances of 0.825-0.934 — no
overlap at all, and nothing near my 0.6 cutoff on either side. Given that
much margin I'd expect 5 of 5, but I'm keeping the target at the course
default of 4 of 5 rather than tightening it, since I only measured the gap
once and haven't seen how it holds up on questions I haven't tried yet.

---

## 4. Chunks read as complete, self-contained thoughts

For at least 4 of 5 randomly sampled chunks, a reader can answer a question
about that chunk's topic using only the text inside it — no missing
antecedent (e.g. "the building" with no building named) and no sentence cut
off at either edge.

**Why this target:** My chunker (`chunker.py::split_documents`) groups
paragraphs up to a 700-character cap rather than cutting at a fixed
character count, and every campus_life document is under 550 characters, so
in practice a chunk is a whole document. Reading the five samples in the
Sample Chunks section below, all five stand alone. I'm still setting the bar
at 4 of 5 rather than 5 of 5: a few documents (like the dorm pages) pack a
building overview, a pro, a con, and a laundry note into one chunk, and it's
plausible a stricter reader would say that's two thoughts glued together
rather than one.

---

## 5. Source attribution is correct, not just present

For at least 4 of 5 questions that name a specific dorm (e.g. "Aldridge
Hall," "Innisfree Hall"), the source(s) the answer cites are documents about
that same dorm, not a different one.

**Why this target:** Several housing documents in campus_life describe
near-identical amenities in near-identical wording — "$1.75 wash" for
laundry shows up for more than one dorm — so a question about one dorm's
laundry could plausibly retrieve a description of a different dorm's
laundry and cite it as if it answered the question asked. Criterion 2 only
checks that *a* source is named; this checks that it's the *right* one,
which matters more for a system whose whole point is "here's exactly which
document said that." I'm leaving one of five as slack because I haven't
tested how often this near-duplicate wording actually causes a mix-up in
practice, only that the risk is real.


---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
