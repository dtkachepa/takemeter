# TakeMeter — Planning

> **Note:** This is a living document. It will be updated as the project progresses.

## Project Overview

TakeMeter is an NLP fine-tuning assignment that builds a classifier to label comments from the Messi vs Cristiano Ronaldo GOAT debate into one of three discourse-quality categories: `analysis`, `hot_take`, and `reaction`. The model is fine-tuned on a manually reviewed dataset of 210 examples collected from public football forums.

## Community

r/soccer and related football forums (redcafe.net, bigsoccer.com) are spaces where the Messi vs Cristiano Ronaldo GOAT debate has played out for over 15 years. The distinction between analysis, hot_take, and reaction matters here because the community itself polices these — replies like "that's not an argument, that's just a feeling" or "cite your stats" are common. These labels reflect how the community itself evaluates discourse quality.

This community is a strong fit for a classification task because the Messi vs Ronaldo debate naturally produces all three label types in every thread. The same event — a goal, an interview, a pundit quote — simultaneously triggers immediate emotional reactions, bold unsupported assertions, and stat-backed arguments. This means discourse variety is structural, not incidental. A single thread of 50 comments will reliably contain examples of all three labels, making data collection efficient and the classification boundary genuinely meaningful rather than artificially constructed.

## Label Taxonomy

| Label | One-sentence definition | Unclear if... |
| --- | --- | --- |
| analysis | A claim supported by specific, verifiable evidence (stats, trophies, head-to-head records, tactical observations) with explicit reasoning. | The evidence is decorative or a conversation-ender rather than part of a reasoning chain — then it's a hot_take. |
| hot_take | A bold opinion or assertion stated as obvious fact, with little or no supporting evidence or explicit reasoning. | It references a pattern or fact but cites no source and the framing is assertive/imprecise — still a hot_take. |
| reaction | An emotional, visceral, or meta response — frustration, a joke/meme, commentary on the thread — that makes no argument. | It could be read as making a point, even a speculative one — then it leans hot_take. |

## Full Label Definitions

### analysis

The comment makes a claim supported by specific, verifiable evidence such as statistics, trophy counts, head-to-head records, or tactical observations grounded in what the player actually does on the pitch. The reasoning must be explicit — the commenter is not just asserting, they are supporting their claim.

- **Clear example 1:** "Messi has more outside-the-box goals than Ronaldo despite Ronaldo taking way more shots from distance — the efficiency gap is real"
- **Clear example 2:** "Has Ronaldo ever performed against Barcelona? I mean i know he scored that goal in the copa, but apart from that. I can't think of one stand out performance. Put it this way, when your club pays 80 mil for you, twice the amount of zidane, they should expect the kind of performances against your closest rivals that Messi puts in."
- **Uncertain example:** "Messi has 8 Ballon d'Ors, Ronaldo has 5. The gap speaks for itself." → This cites real, verifiable data but has no reasoning chain — the stat is dropped as a conversation-ender, not used to build an argument. **Decision rule:** label as hot_take. The test is whether the comment could exist without the evidence — here it could ("Messi is just better. Full stop."), so the stat is rhetorical, not analytical.

### hot_take

The comment makes a bold opinion or assertion stated as obvious fact, with little or no supporting evidence. The commenter is confident and declarative but is not backing the claim with data or explicit reasoning. Includes dismissals, strong rankings, and confident comparisons stated without proof.

- **Clear example 1:** "Ronaldo would be comfortably the best player of his generation if Messi didn't exist. Messi is far and away the best player of his generation and possibly any generation, and to even THINK of denying that now would be farcical."
- **Clear example 2:** "There was never a real debate — anyone who watched both knows Messi is on another level"
- **Uncertain example:** "Footballers who think Ronaldo is better: Ronaldo himself and some he played with. Footballers who think Messi is better: pretty much everyone else." → This feels like it could be analysis because it references a pattern among professional players. **Decision rule:** label as hot_take because the claim is unverified — no poll or source is cited, and the framing ("pretty much everyone else") is imprecise and assertive.

### reaction

The comment is an emotional, visceral, or meta response. Includes: expressing frustration or exhaustion with the debate, making a joke or meme reference, commenting on how others are behaving in the thread, or reacting to something that just happened. The comment is not making an argument — it is expressing a feeling or attitude.

- **Clear example 1:** "Messi is to Cr Ronaldo what Goku is to Vegeta."
- **Clear example 2:** "Ronaldo will be on his deathbed and live into his 100s, while refusing to go before Messi. With his last breath, a weak siiuuuu escapes his lips."
- **Uncertain example:** "Ronaldo will never admit Messi is better." → This sounds like frustration but is actually making a claim about Ronaldo's psychology and character. **Decision rule:** label as hot_take. If a comment could be argued as making a point — even a speculative one — it leans hot_take over reaction.

## Hard Edge Cases

The following are 3 specific examples that gave genuine pause during annotation, which labels they could belong to, and what I decided.

**Case 1:**

- **Post:** "Messi has 8 Ballon d'Ors, Ronaldo has 5. The gap speaks for itself."
- **Could be:** analysis (cites real, verifiable data) or hot_take (no reasoning chain)
- **Decided:** hot_take — the stat is dropped as a conversation-ender, not used to build an argument. Decision rule applied: could this comment exist without the evidence? Yes ("Messi is just better. Full stop.") — so the stat is rhetorical, not analytical.

**Case 2:**

- **Post:** "Footballers who think Ronaldo is better: Ronaldo himself and some he played with. Footballers who think Messi is better: pretty much everyone else."
- **Could be:** analysis (references a pattern among professional players) or hot_take (unverified, no source cited)
- **Decided:** hot_take — the framing ("pretty much everyone else") is imprecise and assertive. No poll or source is cited to back the claim.

**Case 3:**

- **Post:** "Ronaldo will never admit Messi is better."
- **Could be:** reaction (sounds like frustration) or hot_take (makes a claim about Ronaldo's psychology)
- **Decided:** hot_take — the comment is making a speculative point about Ronaldo's character, not just expressing a feeling. Decision rule applied: if a comment could be argued as making a point, even a speculative one, it leans hot_take over reaction.

## Mutual Exclusivity Check

You can assign exactly one label to any given post without ambiguity most of the time. The only recurring overlap is between hot_take and analysis — resolved by asking: does the commenter provide a specific, verifiable fact AND use it as part of a chain of reasoning? If yes, analysis. If the evidence is decorative or a conversation-ender, hot_take. The reaction label rarely overlaps with the others because reactions do not make claims.

## Dataset Overview

- All labels are to have a 20% minimum threshold
- Data will be collected from 8 public threads across r/soccer, r/football, redcafe.net, and bigsoccer.com
- **Train / validation / test split:** 70% / 15% / 15% (147 / 32 / 31 examples)

**Data sources:**

- https://www.reddit.com/r/soccer/comments/1onggjs/
- https://www.reddit.com/r/football/comments/1bs79a6/
- https://www.reddit.com/r/football/comments/zlmyuc/
- https://www.reddit.com/r/football/comments/161r51b/
- https://www.reddit.com/r/soccer/comments/1u88gb2/
- https://www.bigsoccer.com/threads/messi-vs-ronaldo.2008262/
- https://www.redcafe.net/threads/messi-vs-ronaldo-2011-2012.337782/page-2
- https://www.redcafe.net/threads/the-comprehensive-ronaldo-thread.474165/page-45

## Data Collection Plan

- **Sources:** Public threads from r/soccer, r/football, redcafe.net, and bigsoccer.com focused specifically on the Messi vs Cristiano Ronaldo GOAT debate.
- **Target per label:** Minimum 42 examples per label (20% of 210). If any label falls below this after collection, I will seek additional examples from threads where that label type is likely to appear naturally — stat-heavy threads for analysis, interview reaction threads for reaction, any active GOAT debate thread for hot_take.
- **Collection method:** Raw comments collected verbatim from 8 public threads. Comments which directly reference Messi, Ronaldo, or both and make a point about their comparison or GOAT status. Off-topic comments etc are to be excluded.
- **Labeling method:** I will label 70% of the examples directly using the definitions and decision rules in this document. For the remaining 30%, I will use AI to pre-label them before reviewing. I will track which examples were pre-labeled by maintaining two separate files: the raw AI-labeled batch and the final reviewed and corrected version..
- **If a label is underrepresented after 200 examples:** Target additional collection from threads where that label type is likely to appear. For analysis, seek out stat-heavy debate threads or tactical breakdown posts. For reaction, seek match threads or interview reaction threads. For hot_take, any active GOAT debate thread will surface these naturally.

## Evaluation Metrics

Accuracy alone is not sufficient for this task for two reasons. First, if the label distribution is uneven e.g hot_take is 45% of examples, so a model that predicts hot_take for every input would achieve 45% accuracy without learning anything. Second, accuracy treats all errors equally, but for this task it matters which labels get confused i.e a model that consistently misclassifies analysis as hot_take is failing at the core distinction this taxonomy was built around, even if its overall accuracy looks acceptable.

**Primary metric:** Macro F1 which averages the F1 score across all three labels equally, regardless of class size. This penalizes the model for ignoring minority classes and cannot be gamed by always predicting the majority label.

**Secondary metrics:**

- Per-class precision and recall for all three labels -> to identify which specific labels the model struggles with
- Confusion matrix -> to identify which label pairs are most commonly confused 
- Comparison against zero-shot baseline (Groq llama-3.3-70b-versatile) on the same test set -> to verify fine-tuning actually helped

**Why per-class metrics matter here:** The analysis/hot_take boundary is the hardest distinction in this taxonomy. A model might achieve strong overall F1 while systematically confusing these two labels. Per-class recall on analysis is the most important individual metric to watch, if the model rarely predicts analysis correctly, the classifier is not capturing the core distinction the taxonomy was built around.

## Definition of Success

For this classifier to be genuinely useful in a real community tool, for example, flagging low-effort hot takes in a moderated football forum or surfacing high-quality analysis comments, it needs to reliably distinguish all three label types, not just perform well on average. A classifier that cannot identify analysis comments is useless for that purpose regardless of its overall score. I will consider the model good enough for deployment if it meets all of the following thresholds on the held-out test set:

- **Macro F1 ≥ 0.70** — primary threshold
- **No individual label F1 below 0.60** — ensures the model has learned all three categories, not just the easy ones
- **Macro F1 at least 0.10 higher than the zero-shot Groq baseline** — confirms fine-tuning added real value

A result below these thresholds is still a valid project outcome — the evaluation report will analyze where the model falls short and why. 

## AI Tool Plan

**Label stress-testing:**

- I will give ChatGPT my label definitions and edge case rules from this document and ask it to generate 5–10 posts that sit on the boundary between two labels. I will then attempt to classify each generated post myself using my decision rules. If I cannot cleanly assign a label to a generated post, that is a signal my definitions need tightening before I annotate 200 examples. I expect the analysis/hot_take boundary to produce the most difficult cases and will refine the decision rule there if needed.

**Annotation assistance:**

- I will use ChatGPT to pre-label a batch of 30% of the examples before reviewing them myself. The remaining 70% I will label directly using the definitions and decision rules in this document. I will track which examples were pre-labeled by maintaining two separate files: the raw AI-labeled batch and the final reviewed and corrected version.

**Failure analysis:**

- After fine-tuning, I will export the list of wrong predictions from the test set and give them to an AI tool with the following prompt: "Here are comments my classifier got wrong. Each has the true label and the predicted label. Identify any systematic pattern in the errors — for example, does the model consistently confuse short hot_takes with reactions, or misclassify analysis when no numbers are present?" I will then verify the AI-identified pattern manually by checking whether at least 3 of the wrong predictions fit the pattern before including it in the evaluation report.
