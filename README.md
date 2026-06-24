# TakeMeter — r/MarvelStrikeForce Comment Classifier

Classifies subreddit posts into three intent categories.

## Labels
- `roster_strategy` — actionable in-game team/build/clear advice
- `monetization_critique` — claims about the game economy, pricing, F2P fairness, publisher conduct
- `reaction` — emotional response (hype, venting, jokes) with no specific advice or economic claim

## Dataset
- 221 labeled examples from r/MarvelStrikeForce
- Split: 70/15/15 train/val/test, stratified

## Results (test set, n=34)

| Model | Accuracy | monetization F1 | reaction F1 | roster F1 |
|-------|----------|-----------------|-------------|-----------|
| Zero-shot LLM baseline (Llama-3.1-8b) | 0.94 | 0.92 | 0.92 | 1.00 |
| Fine-tuned DistilBERT (3 epochs) | 0.44 | 0.53 | 0.00 | 0.53 |

## Reflection
The fine-tuned DistilBERT reached only 44% test accuracy versus the 94% LLM baseline, and critically scored F1=0.00 on `reaction` — it never correctly identified that class, collapsing toward `monetization_critique`. The near-flat training loss (1.10→1.05) and steadily rising but low validation accuracy (0.42→0.48→0.61) confirm the model was severely undertrained at 3 epochs on
~155 examples. This reflects a known pattern: with a small dataset, a pre-trained LLM's broad language knowledge decisively beats an undertrained from-scratch fine-tune. With more epochs (loss was still falling) and more data, I'd expect the fine-tuned model to recover, especially on the collapsed `reaction` class.

## Demo Video

[![Watch the demo](https://cdn.loom.com/sessions/thumbnails/aa5f9b9dc2384dc7b28a647da5d6fbe8-82f3886daad76703-full-play.gif)](https://www.loom.com/share/aa5f9b9dc2384dc7b28a647da5d6fbe8)
