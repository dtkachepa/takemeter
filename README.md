# TakeMeter

## Project Overview

TakeMeter is a fine-tuned text classifier that evaluates discourse quality in the Messi vs Cristiano Ronaldo GOAT debate community. It classifies comments from r/soccer and related football forums (redcafe.net, bigsoccer.com) into three categories: `analysis`, `hot_take`, and `reaction`. The model is built on `distilbert-base-uncased` fine-tuned on 200 labeled examples collected from public football forum threads.

---

## Community

**Community:** r/soccer and related football forums (redcafe.net, bigsoccer.com)

The Messi vs Cristiano Ronaldo GOAT debate has played out across these communities for over 15 years. This community is a strong fit for a classification task because the debate naturally produces all three label types in every thread — the same event simultaneously triggers emotional reactions, bold unsupported assertions, and stat-backed arguments. The community itself polices discourse quality, with replies like "that's not an argument, that's just a feeling" and "cite your stats" being common.

---

## Label Taxonomy

| Label | Definition |
|---|---|
| `analysis` | A claim supported by specific, verifiable evidence (stats, trophy counts, tactical observations) with explicit reasoning |
| `hot_take` | A bold opinion or assertion stated as obvious fact, with little or no supporting evidence |
| `reaction` | An emotional, visceral, or meta response — frustration, a joke, exhaustion with the debate — that makes no argument |

**Key decision rule:** If a comment cites a stat but uses it as a conversation-ender with no reasoning chain, label it `hot_take` not `analysis`. If a comment could be read as making a point, even a speculative one, label it `hot_take` not `reaction`.

---

## Dataset

- **Total examples:** 200
- **Sources:** 8 public threads across r/soccer, r/football, redcafe.net, bigsoccer.com
- **Label distribution:** hot_take 86 (43%), reaction 59 (29.5%), analysis 55 (27.5%)
- **Split:** 70% train / 15% validation / 15% test (140 / 30 / 30)
- **Collection:** Raw comments collected verbatim. Comments required to directly reference Messi, Ronaldo, or both and make a point about their comparison or GOAT status.
- **Labeling:** I labeled 70% of examples directly. I used ChatGPT to pre-label a batch of 30% before reviewing them myself. My review corrected 15 labels and removed 11 off-topic entries.

**Data sources:**
- https://www.reddit.com/r/soccer/comments/1onggjs/
- https://www.reddit.com/r/football/comments/1bs79a6/
- https://www.reddit.com/r/football/comments/zlmyuc/
- https://www.reddit.com/r/football/comments/161r51b/
- https://www.reddit.com/r/soccer/comments/1u88gb2/
- https://www.bigsoccer.com/threads/messi-vs-ronaldo.2008262/
- https://www.redcafe.net/threads/messi-vs-ronaldo-2011-2012.337782/page-2
- https://www.redcafe.net/threads/the-comprehensive-ronaldo-thread.474165/page-45

**Difficult annotation cases documented:**
- "Messi has 8 Ballon d'Ors, Ronaldo has 5. The gap speaks for itself." → `hot_take` (stat without reasoning chain)
- "Footballers who think Ronaldo is better: Ronaldo himself... pretty much everyone else." → `hot_take` (unverified, imprecise framing)
- "Ronaldo will never admit Messi is better." → `hot_take` (speculative claim, not a feeling)

---

## Model and Training

**Base model:** distilbert-base-uncased

**Training approach:** Fine-tuned for sequence classification with a classification head on top. The notebook handles tokenization, splitting, and evaluation automatically.

**Hyperparameter decisions:**

| Parameter | Default | Final | Reason |
|---|---|---|---|
| num_train_epochs | 3 | 7 | Initial run showed the model underfitting at 3 epochs; validation accuracy peaked at epoch 3 suggesting more passes needed |
| per_device_train_batch_size | 16 | 8 | Smaller batch size gives more frequent weight updates on a small dataset of 140 training examples |
| learning_rate | 2e-5 | 1e-5 | Lowered to reduce overfitting risk; smaller steps preserve more of distilbert's pre-trained knowledge |

---

## Evaluation Report

### Overall Results

| Model | Accuracy | Macro F1 |
|---|---|---|
| Zero-shot baseline (Groq llama-3.3-70b-versatile) | 0.867 | 0.87 |
| Fine-tuned (distilbert-base-uncased) | 0.733 | 0.72 |

### Per-Class Metrics

**Baseline (Groq llama-3.3-70b-versatile):**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 1.00 | 0.75 | 0.86 | 8 |
| hot_take | 0.85 | 0.85 | 0.85 | 13 |
| reaction | 0.82 | 1.00 | 0.90 | 9 |
| **macro avg** | **0.89** | **0.87** | **0.87** | **30** |

**Fine-tuned (distilbert-base-uncased):**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 1.00 | 0.50 | 0.67 | 8 |
| hot_take | 0.62 | 1.00 | 0.76 | 13 |
| reaction | 1.00 | 0.56 | 0.71 | 9 |
| **macro avg** | **0.87** | **0.69** | **0.72** | **30** |

### Confusion Matrix (Fine-Tuned Model)

| | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | 4 | 4 | 0 |
| **True: hot_take** | 0 | 13 | 0 |
| **True: reaction** | 0 | 4 | 5 |

### Success Criteria Assessment

| Criterion | Target | Result | Pass? |
|---|---|---|---|
| Macro F1 ≥ 0.70 | 0.70 | 0.72 | ✅ |
| No individual label F1 < 0.60 | all ≥ 0.60 | lowest: 0.67 | ✅ |
| Beat baseline by 0.10 | ≥ 0.97 | 0.72 | ❌ |

### Analysis of Wrong Predictions

The fine-tuned model made 8 wrong predictions, all classified as `hot_take`. Confidence scores for all wrong predictions ranged from 0.37–0.44 — all below 0.50 — indicating the model was uncertain and defaulting to the majority class rather than confidently wrong.

**Three specific failures:**

**Failure 1:**
Post: "I think they are both decent at crossing. Ronaldo's cross has more pace, Messi has good placement."
True label: `analysis` | Predicted: `hot_take` (confidence: 0.44)
Why it failed: This is a qualitative tactical comparison — two specific attributes with distinct characteristics assigned to each player. The model appears to have learned that analysis requires explicit statistics or numbers. When analysis is made through qualitative tactical observation with no numbers present, the model cannot recognize it and defaults to hot_take.

**Failure 2:**
Post: "So one more UCL puts CR7 over Messi? What about Messi's World Cup, or two Ballon D'ors he has over CR7? Or are those 'rigged'?"
True label: `analysis` | Predicted: `hot_take` (confidence: 0.41)
Why it failed: The post makes a counter-argument using specific verifiable trophies as evidence — a World Cup and a 2 Ballon d'Or gap. The rhetorical question framing confused the model — it recognized the assertive tone but missed the evidence-based reasoning underneath it.

**Failure 3:**
Post: "Ronaldo will be on his deathbed and live into his 100s, while refusing to go before Messi. With his last breath, a weak siiuuuu escapes his lips."
True label: `reaction` | Predicted: `hot_take` (confidence: 0.41)
Why it failed: This was our clear example 1 for the reaction label in planning.md — it was used as a training example. The model failing on a training example suggests the reaction class did not have enough examples with this style (elaborate creative/humorous writing) to generalize. The model may have learned reaction = short dismissive one-liner and couldn't recognize longer creative reactions.

### Error Pattern Analysis

Two systematic patterns account for all 8 wrong predictions:

**Pattern 1 — The model hasn't learned qualitative analysis (4 failures)**
All 4 misclassified analysis posts make their arguments without explicit numbers — using historical time periods, tactical comparisons, meta-arguments about the debate, or trophy references as rhetorical questions. The model appears to have learned analysis = explicit statistics. Qualitative reasoning gets pulled into hot_take.

**Pattern 2 — Longer or substantive reactions get misclassified (4 failures)**
All 4 misclassified reaction posts contain more than pure emotion — a declarative statement, a reference to real facts, a meta-claim about community behavior, or elaborate creative writing. The model appears to have learned reaction = short dismissive one-liner.

**Is this a labeling problem or a data problem?**
The labels are consistent — going back through all 8 wrong predictions against the definitions confirms 7 out of 8 are correctly labeled. This is a data distribution problem: 140 training examples is not enough for a 67M parameter model to learn subtle distinctions that even humans find genuinely difficult, particularly when the majority class (hot_take at 43%) provides a safe default prediction under uncertainty.

---

### Sample Classifications

The following posts were run through the fine-tuned model:

| Post (truncated) | True Label | Predicted | Confidence | Notes |
|---|---|---|---|---|
| "Has more goals per game, more goals per 90. Significantly fewer shots taken per goal..." | analysis | analysis | 0.82 | Correctly identified — explicit per-90 stats with comparative reasoning |
| "There was never a real debate — anyone who watched both knows Messi is on another level" | hot_take | hot_take | 0.79 | Correctly identified — bold assertion, no evidence |
| "Messi is to Cr Ronaldo what Goku is to Vegeta." | reaction | hot_take | 0.53 | Incorrect — model missed the meme/joke format, treated as a comparative claim |
| "Ronaldo would be comfortably the best player of his generation if Messi didn't exist." | hot_take | hot_take | 0.71 | Correctly identified — classic declarative assertion without evidence |
| "I think they are both decent at crossing. Ronaldo's cross has more pace, Messi has good placement." | analysis | hot_take | 0.44 | Incorrect — qualitative tactical observation without numbers, model defaulted to majority class |

---

## Reflection: What the Model Captured vs What I Intended

The model learned a narrower version of each label than I intended. For `analysis`, it learned "contains explicit numbers or statistics" rather than "contains a reasoning chain supported by evidence" — qualitative arguments with no numbers get misclassified. For `reaction`, it learned "short and dismissive" rather than "expresses a feeling rather than making a claim" — longer creative or substantive reactions get pulled toward hot_take. The model captured the surface signals of each label rather than the underlying intent.

The most revealing finding is that the zero-shot baseline (llama-3.3-70b-versatile, macro F1 0.87) outperformed the fine-tuned model (macro F1 0.72) by a significant margin. A large language model with broad language understanding handles this nuanced classification better than a small model fine-tuned on 140 examples. Fine-tuning on limited data caused the model to overfit to surface patterns — explicit numbers for analysis, short length for reaction — rather than learning the conceptual boundaries the taxonomy was designed around. More training data, particularly examples that break the surface-level patterns (qualitative analysis, long reactions), would likely close this gap.

---

## Spec Reflection

**Where the spec helped:** The hard edge case decision rules in planning.md — particularly "could this comment exist without the evidence?" for the analysis/hot_take boundary — were the single most useful piece of the specification. They gave a concrete, consistent answer to the hardest annotation decisions and likely reduced label noise in the training set.

**Where implementation diverged:** The spec was written assuming the fine-tuned model would outperform the zero-shot baseline. In practice the baseline was stronger, which means the third success criterion (beat baseline by 0.10) was not achievable with 200 examples and a 67M parameter model. The spec's success criteria were appropriate targets but did not account for the possibility that the baseline would itself perform at 0.87 macro F1 on this task.

---

## AI Usage

**Instance 1 — Annotation assistance:**
I used ChatGPT to pre-label a batch of 60 examples using the label definitions and decision rules from planning.md. I labeled the remaining 140 examples directly myself. My review of the ChatGPT batch corrected 15 mislabeled entries, primarily analysis entries that should have been hot_take. The pre-labeled batch is tracked separately from the final reviewed version.

**Instance 2 — Failure pattern analysis:**
After fine-tuning, I gave my 8 wrong predictions to Claude and asked it to identify systematic patterns in the errors. It identified two patterns: the model defaulting to hot_take when uncertain, and the model learning surface signals rather than conceptual boundaries. I verified both patterns manually by re-reading all 8 wrong predictions before including them in this report.

---

## Repository Structure

```
takemeter/
├── planning.md                          # Design thinking and label definitions
├── README.md                            # This file
├── messi_ronaldo_verified_210.csv       # Full labeled dataset
├── evaluation_results.json             # Baseline vs fine-tuned comparison
└── confusion_matrix.png                # Fine-tuned model confusion matrix
```
