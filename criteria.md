# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy

Accuracy on the held-out test split is **at least 0.60 on each of the three seeds**.

**Why this target:** With four labels, random guessing gets 0.25. Criterion 3 caps my biggest label at 40%, so a model that always guesses that label scores at most 0.40. 0.60 sits 20 points above that floor, which is the point where I'd trust it to sort a thread better than skimming it. It isn't higher because the test split is only about 30 posts: one post is worth 0.033, and the floor has to hold on the worst of three seeds, not the average.

---

## 2. Per-label performance

`analysis`, `hot_take`, and `reaction` each have an **F1 of at least 0.55**, and `joke` has an **F1 of at least 0.35**, on each of the three seeds.

**Why this target:** The three main labels should each have about 50 or more examples, so about 8 or more land in the test split, enough for an F1 that doesn't swing wildly. 0.55 is a little below the overall target because per-label F1 is noisier than accuracy. `joke` gets a looser number because it will be my thinnest label (maybe 25 to 30 posts, so about 4 in test, where one miss moves the F1 by roughly 0.2). Under my sarcasm rule it also only covers pure bits, which makes it the hardest label to learn from text alone. I still set it well above 0, because a label the model never gets right isn't really in the taxonomy.

---

## 3. Balance

In `labels.csv`, **no label is above 40% of the rows and no label is below 12%**.

**Why this target:** r/hockey game threads are almost entirely "gg" and venting, so `reaction` will crowd out everything else unless I hold it back on purpose. The course cap is 70%, but at 70% a model can score 0.70 by always guessing one label, which would make criterion 1 meaningless. 40% keeps that shortcut below my accuracy target. The 12% floor is 24 of 200 posts, about 4 in the test split, the fewest I think can produce an F1 that means anything at all.

---

## 4. Confidence

On each of the three seeds, accuracy on the model's **most-confident third** of test posts is **at least 0.20 higher** than on its **least-confident third** (`accuracy_most_confident_third − accuracy_least_confident_third ≥ 0.20` in `results.json`).

**Why this target:** If I ever use this to sort posts, I'd only want to trust the confident predictions, so confidence has to mean something. Each third is about 10 posts, so the gap moves in steps of 0.10. 0.20 means at least two more correct posts in the confident third. That's small enough to clear if confidence works at all, and big enough that one lucky post can't fake it.

---

## 5. Consistency

When I label the 30 staff posts under the staff taxonomy, my **agreement rate is at least 0.80** (at most 6 disagreements out of 30), as reported by `agreement.py`.

**Why this target:** This is my only outside check on whether I apply a written rule the way its author meant. The staff `analysis` and `hot_take` definitions are close to mine, so those should mostly match. Their `reaction` is time-based ("just happened") and mine is about venting, a habit I'll have to unlearn for those 30 posts. I'm allowing 6 misses for that. Below 24 of 30, I'd doubt that my own 200 labels were applied consistently either.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
