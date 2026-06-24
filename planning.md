# TakeMeter — Planning Document

**Project:** Show What You Know: TakeMeter
**Community:** r/MarvelStrikeForce
**Author:** Jaime Nunez
**Date started:** June 17, 2026

---

## 1. Community

I chose **r/MarvelStrikeForce**, the main subreddit for the mobile gacha/RPG
*Marvel Strike Force* (published by Scopely's Boundless Entertainment). I play the game, which matters: consistent annotation requires recognizing what counts as a substantive vs. shallow take in the community's own terms, and domain fluency lets me label faster and more reliably.

The community is a strong fit for a discourse-quality classifier because its posts fall into naturally distinct, community-recognized types. As a deep team-building game wrapped in a heavily monetized live-service economy, MSF discourse splits cleanly into three categories:

- **Strategy**: Players trading actionable advice on rosters, investment, and content clears.
- **monetization debate**: A famously contentious axis, since MSF is frequently cited as an example of aggressive free-to-play monetization. 
- **Reaction**: Hype and venting around new characters and events.

This variety gives a classifier a real, non-trivial distinction to learn, while the distinctions still matter to actual participants.

**Source scope note:** I will collect *only* from the main r/MarvelStrikeForce subreddit. I am deliberately excluding the secondary/satire subreddits (e.g. the circlejerk community) because their irony and sarcasm would pollute the `reaction` class and make annotation inconsistent.

---

## 2. Labels

A post is assigned **exactly one** of three labels.

### `roster_strategy`
The post delivers actionable, mechanics-grounded guidance on team building, character investment priority, gear/iso decisions, or clearing specific content that a player could directly execute in-game.

*Example A:* "For War defense, pair Hela with the rest of Asgardians and gear her to 6 Red Stars first. Her revive is what makes the team. Don't bother investing in the others past Gear Tier 13 until she's maxed."
 
*Example B:* "If you're stuck on the Doom legendary event, you don't need maxed Symbiotes. Just get all five to 6 stars and Gear Tier 12, then isolate the boss with Carnage's bleed."

### `monetization_critique`
The post makes a judgment about Scopely's pricing, paywalls, power creep, or free-to-play fairness, asserted as an opinion about the game's economy rather than as executable gameplay advice.

*Example A:* "Another $100 offer locked behind the new meta team. This is the most
  predatory the game has ever been. Power creep every two weeks just to sell the
  counter."

*Example B:* "The fact you basically can't compete in the new game mode without dropping real money tells you everything about where Scopely's prioritiesare. F2P is an afterthought now."

### `reaction`
The post is an immediate emotional response (hype, frustration, excitement) to a character, event, or news drop, with little to no argument.

*Example A:* "OMG they finally announced Doctor Doom rework, I have been waiting
  THREE YEARS for this, I'm shaking."

*Example B:* "Lost my War match by one node again. I'm so tired of this game man, what is even the point."

---

## 3. Hard Edge Cases

### Anticipated hardest boundary: roster advice blended with monetization complaint

Many MSF posts criticize Scopely's economy *through* an investment recommendation, or justify an investment recommendation *with* a monetization complaint.

Example: "Don't waste your shards on this new character. Scopely deliberately made them weak so you'll spend on the next one."

This carries both a `roster_strategy` signal ("don't invest here") and a `monetization_critique` signal ("Scopely manipulates spending").

**Decision rule — the "deletion test":**
If you delete the gameplay advice, does a *complete monetization complaint* remain? If you delete the monetization complaint, does *complete, executable advice* remain? Label the post by whichever survives as a **standalone, complete statement**. If both survive completely, default to the one stated **first / as the main clause**.

Applied to the example: deleting the advice leaves a complete complaint; deleting
the complaint leaves a vague, incomplete instruction (no actionable *why*) means it is a 
**`monetization_critique`**.

### Second anticipated boundary: emotional venting that implies a balance/monetization judgment

Example: "This new paywall meta is the worst, I'm uninstalling."

This reads as `reaction` (pure venting) but gestures at a `monetization_critique`.
**Rule:** if the post contains *no specific claim* about the economy and exists only to express a feeling, it is `reaction`. A monetization judgment must make an identifiable claim about pricing/fairness/power creep to qualify as `monetization_critique`. "Paywall meta is the worst" with no specifics → `reaction`.

*(A third documented difficult case will be added during annotation in Milestone 3.)*

---

## 4. Data Collection Plan

- **Source:** Public posts and top-level comments from r/MarvelStrikeForce, sampled across thread types to balance label representation:
  - Strategy/advice threads → expected to skew `roster_strategy`
  - Patch-note, offer, and news threads → expected to skew `monetization_critique` and `reaction`
- **Method:** Manual collection (copy-paste into a spreadsheet). Manual collection keeps me close to the data and avoids turning this into a scraping project.
- **Target volume:** At least **200 examples**, aiming for a roughly balanced split (target ≥ 25–30% per label; hard rule: no label > 70%).
- **Underrepresentation contingency:** If, after an initial pass to 200, any label is under ~20%, I will deliberately sample additional threads biased toward that label (e.g., pull more from dedicated advice megathreads to boost `roster_strategy`, or from offer-announcement threads to boost
`monetization_critique`) until each label clears 20%.

---

## 5. Evaluation Metrics

**Overall accuracy** is reported but is insufficient alone: with three classes and a
possibly uneven distribution, a model could score decent accuracy while completely failing one class.

I will therefore report, for both the baseline and the fine-tuned model:

**Per-class precision, recall, and F1.** F1 is my primary per-class number because it balances over-prediction (low precision) against missed examples (low recall). Both are failure modes I care about equally for a discourse tool.
**A confusion matrix.** This is essential for *this* task because my central design hypothesis is that the `roster_strategy` ↔ `monetization_critique` boundary is the hard one. A confusion matrix shows the *direction* of errors, which tells me exactly which boundary the model failed to learn.
- **Macro-averaged F1** as a single summary number that weights all three classes equally, so a strong majority class can't mask a weak minority class.

---

## 6. Definition of Success

Concrete thresholds (so success is objectively checkable at the end):

- **Minimum bar:** Fine-tuned model **overall accuracy ≥ 0.70** (well above the ~0.33 random-guess floor for 3 classes) **and meaningfully beats the zero-shot baseline** on the same test set.
- **"Good enough for a real community tool":** **every per-class F1 ≥ 0.65**, with **macro-F1 ≥ 0.70**. The per-class floor matters because a deployable discourse-quality tool can't be useless on one entire category (e.g., silently failing on all `monetization_critique` posts).
- **Disqualifying signal:** if any class F1 ≈ 0, or if accuracy is suspiciously high (> 0.95) on this subjective task, I will treat it as a red flag for label leakage or labels that are too easy, and investigate before reporting.

---

## 7. AI Tool Plan

This project has no implementation code to generate, so AI assistance is scoped to
three places:

**Label stress-testing (will do):** Before annotating, I will give an LLM my three label definitions and the deletion-test edge rule, and ask it to generate 8–10 posts that sit on the `roster_strategy` ↔ `monetization_critique` boundary. If I can't classify its outputs cleanly, I'll tighten the definitions before annotating 200 examples.
**Annotation assistance (decision: NOT using pre-labeling):** I will hand-label all 200 examples myself to stay close to the data and avoid importing an LLM's label bias into my training set. If I change this decision during Milestone 3, I will flag every pre-labeled example and disclose it in the README AI-usage section.
**Failure analysis (will do):** In Milestone 6 I will paste my misclassified test examples into an LLM and ask it to surface common themes (label-pair confusion, sarcasm, short/low-information posts). I will then re-read every flagged example to verify the pattern myself, and report what I had to correct or discard.

---

## 8. Prediction

The zero-shot baseline will score highest on roster_strategy (it has distinctive vocabulary like character names, 'node,' 'gear'), and will most often confuse reaction with monetization_critique, since both are angry/anti-Scopely and the only difference is whether a specific economic claim is present.