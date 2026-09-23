# The Unofficial Guide

<!-- Replace this line with your name and which corpus you picked. -->

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

This system answers questions about campus life using student-written posts
and dining hall guides. It uses RAG to search through the campus_life corpus
and return answers grounded in real documents. Questions it handles include
dining hall hours, wait times, and course workload. It will not answer
questions outside this corpus.

## Chunking Strategy

**Chunk size:** paragraph-based (no fixed size)
**Overlap:** none

The campus_life documents are short student posts, each containing one or two
complete thoughts. Fixed 800-character windows would rarely split anything,
so I switched to paragraph splitting. Each paragraph becomes one chunk and
can answer a question on its own.

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline
```

**Chunk 2** — source: `course_cs_210_workload.txt#2` — produced by: `chunker.py::split_documents`

```
It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```


**Chunk 3** — source: `course_phys_130.txt#3` — produced by: `chunker.py::split_documents`

```
The one piece of advice: the lab practical is worth 20% and almost nobody prepares for it.
```


**Chunk 4** — source: `dining_verrill_street_grill.txt#1` — produced by: `chunker.py::split_documents`

```
I'm a junior and I've done this twice now. Wait times: up to 30 minutes on Friday evenings, otherwise under 10. The thing worth going for is the burger, which is the only late-night hot food on campus. The thing to know is that one register, so the queue is a single line no matter how busy.
```


**Chunk 5** — source: `housing_morrow_house.txt#2` — produced by: `chunker.py::split_documents`

```
The good: cheapest housing tier by about $900 a year, and the singles are real singles.
```

## Sample Answer

**Question:** What time does Halden Hall close?

**Answer:**

```
Halden Hall closes at 7:00pm.

Source: dining_halden_hall_followup.txt
```

**My relevance cutoff:** 0.6

| Question | In corpus? | Best distance |
|---|---|---|
| What time does Halden Hall close? | Yes | 0.270 |
| What is the capital of Mongolia? | No | 0.799 |
| How do I change the oil in a diesel engine? | No | 0.850 |

## How I Used AI

**1.** I asked Claude to write the paragraph-based chunking function. It returned a working version but did not include any minimum chunk length check. I reviewed the output and accepted it as-is since campus_life posts are already short enough.

**2.** I asked Claude to suggest test questions for my corpus. It gave me five questions based on the documents I shared. I kept the questions but wrote the `expects` keywords myself after checking the actual document content.
**3.** I asked Claude to help diagnose why Question 5 failed. It suggested the answer was split across chunks. I verified this by checking the corpus files directly, then implemented the merge fix myself.
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

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks read as complete thoughts | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 5. Answer returns in under 10 seconds | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |

**Real output sample (Criterion 1, Run 1):**

Question: How long is the wait at Halden Hall during lunch?
Answer: The wait at Halden Hall is rarely more than 8 minutes.
Source: dining_halden_hall_followup.txt

Question: How many midterms does STAT 150 have and is there a final exam?
Answer: Based on the provided documents, there is no information about the number of midterms or a final exam for STAT 150.
Source: course_stat_150.txt

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | 4 of 5 questions had the answer in retrieved chunks across all 3 runs. Question 5 (STAT 150 midterms) missed all 3 runs. |
| 2 | Every answer names a source | MET | All 5 answers named a source file in every run. |
| 3 | Gate stops out-of-corpus questions | MET | Gate refused all 5 out-of-scope questions. |
| 4 | Chunks read as complete thoughts | MET | Sampled chunks were complete paragraphs with no cut sentences. |
| 5 | Answer returns in under 10 seconds | MET | All answers returned within a few seconds. |

## Diagnoses

Question 5 (STAT 150 midterms) missed criterion 1 in all 3 runs.

**Stage: chunking**

The answer "three equally weighted midterms, no final" exists in the corpus but was split across two chunks during paragraph splitting. Neither chunk alone contained the complete exam structure, so retrieval returned chunks that mentioned STAT 150 but not its exam format. The model correctly said it didn't have enough information rather than guessing.

## The Improvement

**What I changed:** Added overlap to the paragraph-based chunker so that short paragraphs that belong together get merged into one chunk, preventing the exam format answer from being split.

**Why I picked it:** The diagnosis pointed at chunking — the STAT 150 exam answer was split across two paragraphs. Merging short adjacent paragraphs keeps the answer in one chunk.

### Run Log — After

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks read as complete thoughts | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Answer returns in under 10 seconds | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |



**Did it help?** Yes. Question 5 went from 0/3 to 3/3. Merging short paragraphs kept the STAT 150 exam format in one chunk, so retrieval could find the complete answer.

## What's Still Broken

All five criteria were met after the improvement. However, criterion 1 was originally set at 4 of 5, which was easy to clear. If I were continuing, I would tighten it to 5 of 5 and test whether the system holds — especially for edge cases where the answer spans multiple documents.

## What I'd Do Differently

I would write criterion 1 as "5 of 5" instead of "4 of 5" from the start. Setting it at 4 of 5 left too much room — a system that misses one question consistently is still passing. A tighter target would have pushed me to fix the chunking problem sooner.
