# TakeMeter

TakeMeter is an AI201 Project 3 workflow for classifying `r/soccer` Reddit discourse into three types of soccer discussion: `analysis`, `hot_take`, and `reaction`. The goal is not just to train a classifier, but to understand whether these labels produce a meaningful, learnable distinction between substantive soccer reasoning, unsupported claims, and immediate fan emotion.

## Repository Contents

- `planning.md`: Working notes for the project spec, label definitions, edge cases, data plan, AI tool plan, and evaluation notes.
- `data/labeled_dataset.csv`: Single labeled CSV used by the notebook.
- `outputs/evaluation_results.json`: Exported baseline-vs-fine-tuned comparison from Colab.
- `outputs/confusion_matrix.png`: Fine-tuned model confusion matrix image exported from Colab.

## Community And Task

The chosen community is `r/soccer`, a large public Reddit community with match clips, transfer rumors, official club news, statistics, quotes, daily discussion threads, and opinion posts. It is a good fit for classification because the discourse is varied: the same subreddit contains tactical breakdowns, emotional match reactions, source-based transfer arguments, referee complaints, and sweeping claims about players, clubs, and managers.

The classifier predicts one label per post or comment:

- `analysis`: A post or comment that makes a structured soccer argument using specific, relevant evidence such as tactics, player roles, match context, stats, injuries, transfer fees, or source reliability.
- `hot_take`: A post or comment that makes a bold soccer claim with little support, vague support, or evidence used mainly to make the claim sound stronger.
- `reaction`: A post or comment that mainly expresses immediate emotion about a goal, miss, referee decision, quote, transfer rumor, clip, or match result without making a broader supported argument.

Label examples:

| Label | Example 1 | Example 2 |
|---|---|---|
| `analysis` | "Arsenal struggled after the 60th minute because the fullbacks stopped inverting, which left Rice isolated against two midfield runners and made every turnover turn into a counter." | "That transfer fee makes sense only if United see him as a left-sided 8, because his progressive carries and pressing numbers fit that role better than playing him as a pure winger." |
| `hot_take` | "That manager is a fraud and only wins because he has expensive players." | "This club has no ambition; every transfer window is just PR and panic." |
| `reaction` | "What a finish. I still cannot believe he scored from there." | "That red card decision is ridiculous." |

## Dataset

The dataset contains 210 public `r/soccer` examples saved in one CSV file, not pre-split. The notebook handles the 70% / 15% / 15% train, validation, and test split.

Label distribution:

| Label | Count | Share |
|---|---:|---:|
| `analysis` | 70 | 33.3% |
| `hot_take` | 70 | 33.3% |
| `reaction` | 70 | 33.3% |

No label is above 70% of the dataset, so the model is not trained on a majority-class-dominated dataset.

CSV columns:

| Column | Purpose |
|---|---|
| `text` | Post or comment text used as model input |
| `label` | Gold label string |
| `notes` | Annotation notes for borderline cases |
| `source_type` | Source type, usually `comment` |
| `source_permalink` | Public old Reddit permalink |
| `created_utc` | Reddit timestamp from the archive |

## Data Collection And Labeling

Examples came from public `r/soccer` comments collected through the PullPush Reddit archive API after direct Reddit JSON access returned a network-security block. I filtered out deleted, removed, bot-like, duplicate, extremely short, overly long, and low-information examples before creating the final CSV.

The labeling process followed the definitions above. `analysis` required specific soccer reasoning or evidence, `hot_take` required a broad unsupported claim, and `reaction` required immediate emotion without a broader supported argument. Borderline examples were marked in the `notes` column so they could be reviewed later.

Three difficult annotation examples:

| Text | Possible Labels | Final Label | Decision |
|---|---|---|---|
| "Ten Hag didn't have a great hand at his disposal but I don't think he should be completely exonerated from crisisism because of that. His tactics were suicidal and in the xg table..." | `hot_take` or `analysis` | `analysis` | It uses heated language, but it also gives soccer-specific reasoning about tactics, xG table comparison, trophies, and squad context. |
| "Again, Amad has 2 more g/a than Kulu who is supposedly your best player, in 8 less games. Amad is miles clear of Kulu..." | `analysis` or `hot_take` | `hot_take` | It cites goal/assist numbers, but the evidence is used mainly to support a sweeping "miles clear" and "crazy overrated" claim. |
| "3 goals in 5 minutes on extra time. Comeback in a European quarter final. Lord Maguire scores the final goal. Absolute scenes." | `analysis` or `reaction` | `reaction` | It mentions match context, but the purpose is immediate excitement, not a structured tactical or statistical argument. |

## Model Setup

The notebook used `distilbert-base-uncased` for fine-tuning. I kept the default hyperparameters:

- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16
- Train / validation / test split: 70% / 15% / 15%

I did not adjust hyperparameters.

## Baseline

Before fine-tuning, I ran a zero-shot Groq LLM baseline on the locked test set. The prompt included the `r/soccer` task, the three label definitions, one example per label, and an instruction to output only `analysis`, `hot_take`, or `reaction`.

Baseline results:

| Metric | Value |
|---|---:|
| Test examples | 32 |
| Parseable responses | 32 / 32 |
| Accuracy | 0.844 |
| Macro F1 | 0.84 |
| Weighted F1 | 0.84 |

Baseline per-class metrics:

| Label | Precision | Recall | F1 | Support |
|---|---:|---:|---:|---:|
| `analysis` | 1.00 | 0.70 | 0.82 | 10 |
| `hot_take` | 0.79 | 1.00 | 0.88 | 11 |
| `reaction` | 0.82 | 0.82 | 0.82 | 11 |

The baseline was strongest overall and was especially conservative about `analysis`: every predicted `analysis` was correct, but it missed 30% of true `analysis` examples.

## Fine-Tuned Results

Fine-tuning completed without error. Validation accuracy improved slightly during training, but the final test results were much worse than the zero-shot baseline.

Training log:

| Epoch | Training Loss | Validation Loss | Validation Accuracy |
|---:|---:|---:|---:|
| 1 | 1.093379 | 1.097772 | 0.516 |
| 2 | 1.094375 | 1.081472 | 0.516 |
| 3 | 1.057627 | 1.038317 | 0.548 |

Fine-tuned test results:

| Metric | Value |
|---|---:|
| Test examples | 32 |
| Accuracy | 0.469 |
| Macro F1 | 0.42 |
| Weighted F1 | 0.42 |
| Wrong predictions | 17 / 32 |

Fine-tuned per-class metrics:

| Label | Precision | Recall | F1 | Support |
|---|---:|---:|---:|---:|
| `analysis` | 0.40 | 1.00 | 0.57 | 10 |
| `hot_take` | 0.50 | 0.18 | 0.27 | 11 |
| `reaction` | 1.00 | 0.27 | 0.43 | 11 |

Baseline vs. fine-tuned comparison:

| Model | Accuracy |
|---|---:|
| Zero-shot baseline (Groq) | 0.844 |
| Fine-tuned DistilBERT | 0.469 |
| Fine-tuning change | -0.375 |

## Confusion Matrix

Rows are true labels. Columns are predicted labels.

| True \ Predicted | `analysis` | `hot_take` | `reaction` |
|---|---:|---:|---:|
| `analysis` | 10 | 0 | 0 |
| `hot_take` | 9 | 2 | 0 |
| `reaction` | 6 | 2 | 3 |

The dominant error is not a single pairwise confusion like `analysis` to `hot_take`. Instead, the fine-tuned model over-predicts `analysis` across both other classes. It correctly catches all 10 true `analysis` examples, but it also sends 9 of 11 `hot_take` examples and 6 of 11 `reaction` examples to `analysis`.

## Wrong Prediction Analysis

I used an AI tool to help look for patterns in the wrong predictions, then verified the patterns by re-reading the examples. The useful pattern was that the model overweights surface soccer evidence: numbers, player names, team names, and match context. I discarded one weaker AI-suggested explanation that the model was mainly confused by post length, because the wrong examples include both short reactions and longer player-opinion comments.

### Example 1

Text: "mate I've been saying this about foden for over 2/3 seasons now. I just don't rate him as much as I rate the other three I named one good full season didn't change my mind..."

- True label: `hot_take`
- Predicted label: `analysis`
- Confidence: 0.37

Why it failed: The model likely treated the player comparison and references to multiple seasons as analytical evidence. Under the project definition, this is still a `hot_take`: it is mostly a personal rating of Foden without a structured argument about role, tactics, stats, or match context.

### Example 2

Text: "Only 15 goals conceded, the lowest of any team in the league. Unbelievable."

- True label: `reaction`
- Predicted label: `analysis`
- Confidence: 0.37

Why it failed: The number "15 goals conceded" looks like evidence, but the post does not build an argument from it. The word "Unbelievable" shows the post is mainly reacting emotionally to an impressive stat. This is a boundary case where the model confuses a statistic with analysis.

### Example 3

Text: "has Maguire been a striker this entire time? Offside but like my word what a finish."

- True label: `reaction`
- Predicted label: `hot_take`
- Confidence: 0.34

Why it failed: The rhetorical question can look like a claim about Maguire's role, but the post is really a live reaction to a finish. This shows a different boundary problem: short, sarcastic reactions can resemble unsupported claims even when they are not trying to make a durable argument.

### Example 4

Text: "People always wanna blame the manager over the guy that score a single goal and can’t stay fit. Pep might be washed but Grealish is horrible"

- True label: `hot_take`
- Predicted label: `analysis`
- Confidence: 0.36

Why it failed: The model probably picked up manager/player discussion and references to goals and fitness. But the language "washed" and "horrible" is a broad unsupported judgment, so the correct label is `hot_take`.

## Sample Classifications

These examples come from the fine-tuned model outputs shown during evaluation.

| Text | Predicted Label | Confidence | Outcome |
|---|---|---:|---|
| "Even with it being a home fixture, Everton at home is exactly the kind of match that Newcastle specialise in getting 0-0'd from an xG of 2-3+. That they have Pickford in goal is just a bonus layer..." | `analysis` | 0.387 | Correct; true label was `analysis` |
| "Massively overrated player, always has been" | `hot_take` | 0.359 | Correct; true label was `hot_take` |
| "mate I've been saying this about foden for over 2/3 seasons now. I just don't rate him as much as I rate the other three I named..." | `analysis` | 0.368 | Incorrect; true label was `hot_take` |
| "Only 15 goals conceded, the lowest of any team in the league. Unbelievable." | `analysis` | 0.374 | Incorrect; true label was `reaction` |
| "What a finish to both the games. The last matchday promises to be exciting and nerve wrecking for both the teams and supporters." | `analysis` | 0.353 | Incorrect; true label was `reaction` |

The first correct `analysis` prediction is reasonable because the post uses xG, matchup context, home/away context, and Pickford's role to explain an expected game pattern. The correct `hot_take` prediction is also reasonable because the post is a broad unsupported judgment: it calls a player overrated without evidence or explanation.

## What The Model Captured Vs. What I Intended

The intended distinction was about discourse structure: `analysis` should require a supported soccer argument, `hot_take` should capture unsupported confident claims, and `reaction` should capture immediate emotional responses. The fine-tuned model instead appears to have learned a shallower boundary: soccer-specific details often mean `analysis`.

That explains the regression. DistilBERT learned that player names, stats, match references, fitness, and tactical vocabulary are strong signals for `analysis`, but it did not reliably learn whether those details were being used as part of an actual argument. This is why "15 goals conceded" became `analysis` even though the post was a reaction, and why "Grealish is horrible" became `analysis` because it mentioned goals and fitness.

To fix this, I would add more hard negative examples: `hot_take` and `reaction` posts that contain numbers, player names, and match context but are still not analysis. I would also add a second annotation pass focused only on the `analysis` boundary and possibly include a binary feature in the prompt or dataset notes: whether the post explains a claim or merely asserts/reacts.

## Spec Reflection

The spec helped most by forcing precise label definitions before annotation. The edge-case rule for posts that mix emotion and evidence made annotation more consistent, especially when a comment mentioned a stat but did not actually build an argument.

The implementation diverged from the spec in one important way: I planned for the fine-tuned model to improve `analysis` recall while keeping precision high, but it did the opposite of the baseline. It maximized `analysis` recall at the cost of many false positives. That divergence revealed that the dataset needs more examples that teach the model the difference between "mentions soccer evidence" and "uses evidence analytically."

## AI Usage

I used AI assistance in two specific places.

First, I used an AI tool during evaluation analysis. After the notebook printed the fine-tuned model's wrong predictions, I gave the misclassified examples to the tool and asked it to identify common patterns, such as repeated label confusions, short or low-information posts, sarcasm, and cases where soccer-specific details made a post look more analytical than it really was. The useful pattern it surfaced was that the model often over-predicted `analysis` when a post contained surface soccer evidence, such as numbers, player names, team names, match context, or manager/player discussion. I verified that pattern myself by rereading the wrong predictions and checking the confusion matrix. I also discarded a weaker suggested pattern that the issue was mainly post length, because the errors included both short reactions and longer player-opinion comments.

Second, I used AI assistance to review the final README against the submission checklist and identify missing report details. That review flagged that the README needed the label examples, difficult annotation examples, and clearer data collection notes to stand on its own. I added those sections manually and kept the final wording tied to my actual dataset, notebook output, and evaluation results.

## Reproducing The Artifact Set

1. Upload `data/labeled_dataset.csv` in the Colab notebook.
2. Use this label map:

```python
LABEL_MAP = {
    "analysis": 0,
    "hot_take": 1,
    "reaction": 2,
}
```

3. Run Sections 1 and 2 to split and tokenize the dataset.
4. Run Section 5 for the zero-shot Groq baseline.
5. Run Section 3 to fine-tune `distilbert-base-uncased`.
6. Run Section 4 to evaluate the fine-tuned model and save `confusion_matrix.png`.
7. Run Section 6 to export `evaluation_results.json`.
8. Place both exported files in `outputs/`.
