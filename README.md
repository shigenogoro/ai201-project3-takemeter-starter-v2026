# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

**Community:** [r/hockey](https://old.reddit.com/r/hockey), Reddit's public general NHL discussion board. Its posts range from stat-backed analysis to one-line reactions in game threads.

TakeMeter sorts r/hockey comments into four kinds of take: `analysis` (a claim backed by a checkable fact), `hot_take` (an arguable position with no support), `reaction` (emotional venting), and `joke` (a pure bit). It's a DistilBERT model fine-tuned on 301 labelled comments from 21 threads. The point is to tell a stat-backed argument apart from noise in a sub where both look the same at a glance.

**Stretch feature:** `joke` is a fourth label. I added it because r/hockey has a lot of pure bits ("Larkin for Hellebuyck. The trade is one for one.") that none of the three base labels describe.

**Candidate distinctions (from reading 67 comments across three threads: the Sharks jersey-ad thread, the Larkin trade-list thread, and the MTL @ WSH game thread from Oct 14 2026):**
1. **Some comments cite a specific, checkable fact. Most don't.** *"He's legit under 32% this season"* (Jack Hughes on faceoffs) and *"Utah just traded for Trocheck, Devils top two centres are both better than Larkin"* point at something you could look up. *"None of those teams want him"* states a claim just as confidently and gives nothing to check.
2. **Some comments are pure feeling, with no claim at all.** *"Ugh cardiac caps are back at it again. Need to close out games cleanly"* and *"We stay above 50 😮‍💨"* react to what just happened without arguing anything. The game thread is almost entirely this kind of comment ("gg Habs", "GG Caps bros"). Its bot comments (Chatterstats, highlight links) aren't human posts and shouldn't be collected.
3. **Some comments are jokes, not claims or feelings.** *"Larkin for Hellebuyck. The trade is one for one."* and *"3 > 8 > 10... might get to 15 by Christmas at this rate."* exist to get a laugh, and taking them literally misreads them.


---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `analysis`

**Definition:** Makes a claim and backs it with at least one specific, checkable fact: a stat, a transaction, a roster detail, or a named source.

**Example 1:**
> One thing to note on NJ. Jack currently isn't taking faceoffs this game and Nico, Erod, or Glass are taking them. Olympics and 4 Nations he played wing and there's been multiple seasons where coaches had someone else take his faceoffs. As much as we call him our 1 or 2 C dude fucking sucks and is a liability at faceoffs. He's legit under 32% this season. There's a growing consensus amongst Devils fans for years that Jack needs to be move to wing full time since it's clear he's not getting better at it.

**Example 2:**
> Of these new teams, I feel like they're all still tough fits? Utah just traded for Trocheck, Devils top two centres are both better than Larkin, Avs are stacked down the middle, Canes probably chill to test Stankoven at 2C…..good luck buddy

### `hot_take`

**Definition:** Takes a position someone could argue against (a prediction, a ranking, a verdict on a player or trade) and offers nothing checkable to support it.

**Example 1:**
> None of those teams want him

**Example 2:**
> I blame Adam Silver and the NBA for starting this.

### `reaction`

**Definition:** Emotional venting (cheering, frustration, disgust, relief) about a play, a result, the news, or the league, with no argument behind it.

**Example 1:**
> We stay above 50 😮‍💨

**Example 2:**
> Ugh cardiac caps are back at it again. Need to close out games cleanly

### `joke`

**Definition:** A pure bit written to get a laugh (an absurd hypothetical, a fake quote, a meme format) with no real complaint or claim underneath.

**Example 1:**
> you know what be really funny? "Larkin for Hellebuyck. The trade is one for one."

**Example 2:**
> Sharks brand/marketing person when they see this post: "Guys, I have an idea"

### The hardest boundary

**Which two labels:** `reaction` vs `hot_take`

**The borderline post:**
> Habs (and Dobes) gave that game away.

**The decision rule I used every time:**
If the post is mainly venting how the poster feels, it's `reaction`, even when it contains a judgment ("Dobes gave that game away", "Jersey ads are the worst thing to happen to this league in 20 years"). It's `hot_take` when the post takes a position someone could argue against, such as a prediction, a ranking, or a verdict on a trade ("None of those teams want him", "Larkin to Minnesota is the only trade that makes sense").

**Second rule (sarcasm):** A sarcastic complaint or claim gets the label of what it's actually saying underneath. "Of course the Caps blow a two-goal lead. Never change, boys." is `reaction`. `joke` is only for pure bits with nothing underneath, like "Larkin for Hellebuyck. The trade is one for one."



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Comments from 21 public r/hockey threads posted Oct 14–20, 2026, collected with a Playwright script that read each thread's page as it loaded (about 30 top comments per thread). I picked the threads to cover every label: stat posts (5v5 xGoal share, team 5v5 records), contract and trade threads (DeBrincat's deal, the price for Larkin, Larkin's trade list, Marchenko, Hellebuyck), highlight and penalty clips (Makar's hit, Dobeš's OT penalty, Parayko "using the force"), a game thread, and lighter threads where jokes cluster (the Hellebuyck comedian bit, the Devils' train-schedule troll, the "Stamkoses" plural). From 654 comments after removing bots, deleted comments, and duplicates, I kept 301. I dropped replies that don't make sense without their parent ("Yes 👍", "He doesn't"), plain questions, non-English comments, and off-topic tangents.

**How I labelled them:** Claude labelled all 301 rows using my four definitions and both decision rules, and the `note` column says so on every row. I did not do the unaided cold pass of 20 rows the brief asks for. The 20 rows that were set aside for it are noted `labelled by Claude (no unaided cold pass was done)`. That means this file is not evidence of how consistently *I* apply my own taxonomy. The staff-set agreement check next unit is the only independent check I'll have.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `analysis` | 47 | 15.6% |
| `hot_take` | 82 | 27.2% |
| `reaction` | 85 | 28.2% |
| `joke` | 87 | 28.9% |
| **Total** | 301 | 100% |

**Three hard cases**

**1.**
> *The post:* "I'm always expecting goals, but we don't actually score them. Jack shoots more than anyone, but only has two goals."
>
> *Could have been:* `reaction` or `analysis`
>
> *Claude chose `analysis`, because:* it opens as venting, but "shoots more than anyone, only two goals" is a checkable fact doing real work in the point being made.

**2.**
> *The post:* "Imo, when comparing analytics success vs standings success, All Situations is the more appropriate metric than 5v5. Otherwise, you're removing special teams from the equation, when special teams performance absolutely contributes to standings success."
>
> *Could have been:* `analysis` or `hot_take`
>
> *Claude chose `hot_take`, because:* it's a careful, reasoned argument, but it cites nothing checkable. My `analysis` definition requires a fact, not just reasoning. This case shows a real gap in my taxonomy: a well-argued post without data lands in the same bucket as "None of those teams want him."

**3.**
> *The post:* "Stunning and brave to have all the teams that have won the cup in the 2020s on your list."
>
> *Could have been:* `joke` or `hot_take`
>
> *Claude chose `hot_take`, because:* under my sarcasm rule, a sarcastic post gets the label of what it's saying underneath. Underneath, this is a verdict that Larkin is only willing to join contenders. It isn't a pure bit.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Device:** `cpu`. My laptop has no NVIDIA GPU (Windows 11, Python 3.12.6, PyTorch 2.14.1+cpu, 63.7 GB RAM). `python test.py` passed 10/10 on 2026-10-20, and the baseline model is already downloaded.

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128 tokens, seed 42. Training took about 5 minutes on CPU.

**Anything I changed from the defaults, and why:** Nothing in the hyperparameters. The only edits to the notebook were `LABELS = ["analysis", "hot_take", "reaction", "joke"]` (to add the fourth label) and my name in section 1. One environment fix: an expired Hugging Face login token cached on my machine made the public model download fail with a 401. I ran the training with `HF_HUB_DISABLE_IMPLICIT_TOKEN=1` so that no token was sent.

**Split sizes:** train 210 · val 45 · test 46

| Label | Train | Val | Test |
|---|---|---|---|
| `analysis` | 33 | 7 | 7 |
| `hot_take` | 57 | 12 | 13 |
| `reaction` | 59 | 13 | 13 |
| `joke` | 61 | 13 | 13 |

`analysis` has only 7 examples in the test split, under the notebook's threshold of about 8, so its F1 will swing a lot between seeds.

**First run (seed 42, from `results.json`):** accuracy 0.478, macro F1 0.368. I haven't read these against `criteria.md` yet; that's next unit's work.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* For Claude to stress-test my four labels by writing eight posts that sit on the boundaries between them. I classified the posts myself without seeing any intended answers.
- *What came back:* Eight boundary posts. Four of my labels contradicted my own written definitions. I'd called "Jersey ads are the worst thing to happen to this league in 20 years" a `reaction` even though nothing had just happened, and I never picked `joke`, even for sarcastic posts.
- *What I changed:* I redefined `reaction` as emotional venting rather than a response to something that just happened, narrowed `joke` to pure bits, rewrote the `reaction` vs `hot_take` rule around venting vs taking a position, and added a second rule that sarcasm gets the label of what it's saying underneath.

**Moment 2**

- *What I asked for:* For Claude to write my five acceptance criteria and the reasons behind each number. The project brief says not to do this, and I'm disclosing that I did.
- *What came back:* A draft of `criteria.md` covering accuracy (0.60), per-label F1 (0.55, with 0.35 for `joke`), balance (40% max, 12% min), confidence (a 0.20 gap between the most- and least-confident thirds), and agreement with the staff set (0.80).
- *What I changed:* Nothing. I kept all five numbers and reasons as Claude drafted them.

**Pre-labelling disclosure:** Claude collected the comments from r/hockey with Playwright, chose 301 of them, and labelled all 301 using my four definitions and two decision rules. I did not do an unaided cold pass, and I did not correct the labels row by row. Every row's `note` column says it was labelled by Claude.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

Both columns are scored on the same 46 posts in `test_split.csv` (seed 42). The baseline is `facebook/bart-large-mnli` zero-shot, prompted with the four definitions in `label_definitions.txt`.

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.152 | 0.478 | +0.326 |
| Macro F1 | 0.066 | 0.368 | +0.302 |
| F1 — `analysis` | 0.264 | 0.000 | -0.264 |
| F1 — `hot_take` | 0.000 | 0.688 | +0.688 |
| F1 — `joke` | 0.000 | 0.485 | +0.485 |
| F1 — `reaction` | 0.000 | 0.300 | +0.300 |

**Definitions vs. bare label names:** I ran the baseline twice. With the bare label names only (no `label_definitions.txt`), it scored **accuracy 0.283, macro F1 0.191** (F1: analysis 0.000, hot_take 0.333, joke 0.000, reaction 0.432). With my definitions it scored **0.152**. So my definitions were worth **−0.13 accuracy** to the zero-shot model. The table above reports the definitions run, as the brief asks.

**What I predicted before I looked:** The trained model will beat the baseline on `hot_take` and `joke`, because those labels depend on r/hockey's tone and a general model has never seen it, and it will not beat it on `analysis`. *(Honesty note: the seed-42 `results.json` from unit 5 was already on disk when I wrote this, and it shows `analysis` F1 = 0.000. So this isn't a blind prediction.)*

**What the gap actually means:** Fine-tuning added a lot. The trained model is 33 points more accurate, and it beats the baseline on three of four labels. But the baseline is a weak opponent. With my definitions it called almost everything `analysis`: my `analysis` definition is the only one full of concrete nouns ("stat, transaction, roster detail, named source"), and an NLI model treats any hockey comment as loosely entailing that. Even with bare names it only reached 0.283, barely above the 0.25 you'd get by guessing at random. The one place the baseline wins is `analysis`, and that's the label my trained model never predicts at all.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

**Device:** `cpu` (CPU, torch 2.14.1+cpu, max length 128), recorded in `results_three_seeds_before.json`. Settings were unchanged from unit 5: 3 epochs, lr 2e-5, batch 16. Each seed took about 7 minutes. Seed 42 reproduced unit 5's `results.json` exactly.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.60 | 0.478 | 0.391 | 0.478 | |
| 2. Per-label F1 (an / ht / re / jk) | ≥ 0.55 / 0.55 / 0.55 / 0.35 | 0.00 / 0.69 / 0.30 / 0.48 | 0.00 / 0.49 / 0.50 / 0.13 | 0.00 / 0.57 / 0.46 / 0.47 | |
| 3. Balance in `labels.csv` (max / min share) | ≤ 40% / ≥ 12% | 28.9% / 15.6% | 28.9% / 15.6% | 28.9% / 15.6% | |
| 4. Confidence gap (most-sure third − least-sure third) | ≥ 0.20 | 0.267 | 0.533 | 0.067 | |
| 5. Agreement with staff set | ≥ 0.80 | not measured | not measured | not measured | |

**Spread across seeds** (notebook section 6):

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.478 | 0.391 | 0.478 | 0.087 |
| Macro F1 | 0.368 | 0.278 | 0.376 | 0.098 |
| F1 — `analysis` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `hot_take` | 0.688 | 0.486 | 0.571 | 0.201 |
| F1 — `reaction` | 0.300 | 0.500 | 0.462 | 0.200 |
| F1 — `joke` | 0.485 | 0.125 | 0.471 | 0.360 |

Accuracy moved 0.087, under the ten-point warning line. But per-label F1 moved 0.20–0.36, so the label-level numbers are much less stable than the headline one.

### Confusion matrix

Seed 42 (the same split as the baseline table):

| true \ predicted | analysis | hot_take | reaction | joke |
|---|---|---|---|---|
| **analysis** | 0 | 5 | 0 | 2 |
| **hot_take** | 0 | 11 | 0 | 2 |
| **reaction** | 0 | 2 | 3 | 8 |
| **joke** | 0 | 1 | 4 | 8 |

**My biggest off-diagonal number, and what it means:** 8 real `reaction` posts were called `joke`, and 4 went the other way. So the model hasn't learned the venting-vs-bit boundary, and on seed 42 it leans toward `joke`. That direction isn't stable, though. On seeds 7 and 2024 it flips: 8 and 6 real `joke` posts were called `reaction`. The stable fact is that `analysis` is never predicted on any seed, and 18 of its 21 test posts across the three seeds went to `hot_take`.



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** not measured. I don't have the answer key, `data/staff_labels.csv`, which the TF hands out, so `agreement.py` can't run and there is no `agreement_results.json`. My 30 labels are in `my_staff_labels.csv`, and the comparison runs as soon as the key is in `data/`.

**How my 30 were labelled:** Claude labelled them, one post at a time, under `data/staff_taxonomy.md` only (not my r/hockey definitions), before any staff label had been seen. They came out 10 `analysis` / 10 `hot_take` / 10 `reaction`, in id order (sp_001–010, sp_011–020, sp_021–030). That suggests staff grouped the file by label, which makes it a weaker blind test than it looks.

**The posts I expect to disagree on, and how I argued them in advance:**

| Post | My label | Rival label | Why mine, from the staff rules |
|---|---|---|---|
| sp_005 "Carbon plates aren't the whole story. My times dropped 2%…" | `analysis` | `hot_take` | Rule 1: the second 2% (same shoes, more mileage) *is* the argument against "shoes are the whole story". It isn't decoration. |
| sp_007 "Ran the same loop in 4C and in 24C… 90 seconds slower" | `analysis` | `reaction` | Rule 3: it's about the writer's own run, but the 90-second gap carries a general claim about heat. |
| sp_010 "Three of us ran the same session… within 10 seconds" | `analysis` | `hot_take` | The 10-second spread is the whole argument ("smaller than the noise"). |
| sp_014 "Times are down about 3% since the shoe rules changed, so obviously…" | `hot_take` | `analysis` | Rule 1 almost word for word: the 3% decorates a conclusion already reached. |
| sp_017 "Road racing peaked in about 2015…" | `hot_take` | `analysis` | "2015" is a date, but nothing is argued from it. |
| sp_018 "Track sessions are wasted on anyone running slower than 8 minute miles" | `hot_take` | `analysis` | The pace is a threshold in the claim, not evidence for it. |
| sp_027 "Wind was so bad… leaning at about 20 degrees" | `reaction` | `hot_take` | Rule 2: it reports what just happened to the writer. |

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

None can be listed until the staff key arrives. The two template slots below are left in place for that.

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:** Nothing yet, and that matters for the diagnoses below. Without this report I have no instrument that can tell a labelling problem from a model problem.



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
