# TakeMeter Planning

## Goal

Build a small AI workflow that classifies or scores examples for the TakeMeter project, evaluates the model, and exports the final results outside the notebook for submission.

## Milestone 1: Community and Labels

### Community

TakeMeter will study discussion in `r/soccer`, a large soccer subreddit where posts and comments mix match clips, transfer rumors, official club news, statistics, quotes, daily discussion threads, and opinion pieces. I reviewed the current front page pattern on old Reddit, which included Daily Discussion, Monday Moan, media clips, official-source news, transfer posts, stats posts, and match clips. These distinctions matter because soccer fans often argue not just about which opinion is right, but whether a take is supported by tactical reasoning, credible sourcing, match context, or just immediate emotion.

Source scan: https://old.reddit.com/r/soccer/

### Label Taxonomy

The model will classify each post or comment into exactly one of three labels.

#### `analysis`

Definition: The post makes a structured soccer argument using specific evidence such as tactics, formations, chance quality, player roles, transfer fees, source reliability, injury context, or historical comparison.

Clear examples:

- "Arsenal struggled after the 60th minute because the fullbacks stopped inverting, which left Rice isolated against two midfield runners and made every turnover turn into a counter."
- "That transfer fee makes sense only if United see him as a left-sided 8, because his progressive carries and pressing numbers fit that role better than playing him as a pure winger."

Uncertain case:

- "Spain leaving out Real Madrid players shows the squad is finally being picked on form instead of reputation."

Decision rule: Label this `analysis` only if the post explains the soccer mechanism, such as player form, role fit, tactical setup, recent minutes, injuries, or selection history. If it only points to one fact and jumps to a broad conclusion, label it `hot_take`.

#### `hot_take`

Definition: The post states a bold or confident soccer opinion with little support, vague evidence, or evidence used mainly for rhetorical effect.

Clear examples:

- "That manager is a fraud and only wins because he has expensive players."
- "This club has no ambition; every transfer window is just PR and panic."

Uncertain case:

- "Bruno is overrated because United have not won the league with him."

Decision rule: If a specific stat or fact is used as part of a broader, checkable argument about role, tactics, squad quality, or match performance, label it `analysis`. If the evidence is thin, cherry-picked, or mainly there to make a sweeping claim sound stronger, label it `hot_take`.

#### `reaction`

Definition: The post is an immediate emotional response to a goal, miss, save, referee decision, quote, transfer rumor, or match result, with little to no attempt to argue a broader claim.

Clear examples:

- "What a finish. I still cannot believe he scored from there."
- "That red card decision is ridiculous."

Uncertain case:

- "This loss proves the manager has no idea how to close out big matches."

Decision rule: If the post is mostly venting about a fresh event, label it `reaction`. If it generalizes into a confident claim about a player, manager, club, or fanbase without support, label it `hot_take`; if it explains the issue with specific substitutions, shape changes, pressing choices, or chance quality, label it `analysis`.

### Mutual Exclusivity Check

These labels are mutually exclusive by the role of evidence and timing. `analysis` requires a reasoned claim with specific support, `hot_take` requires an assertive claim without enough support, and `reaction` captures immediate emotional responses that do not try to make a durable argument. When a post contains both emotion and reasoning, the evidence rule decides: specific, relevant support moves it to `analysis`; broad confidence without support moves it to `hot_take`; no broader claim keeps it as `reaction`.

### Checkpoint Summary

Labels:

- `analysis`: A post or comment that makes a structured soccer argument using specific, relevant evidence such as tactics, player roles, match context, stats, injuries, transfer fees, or source reliability.
- `hot_take`: A post or comment that makes a bold soccer claim with little support, vague support, or evidence used mainly to make the claim sound stronger.
- `reaction`: A post or comment that mainly expresses immediate emotion about a goal, miss, referee decision, quote, transfer rumor, clip, or match result without making a broader supported argument.

Concrete examples:

- `analysis`: "Arsenal struggled after the 60th minute because the fullbacks stopped inverting, which left Rice isolated against two midfield runners and made every turnover turn into a counter."
- `analysis`: "That transfer fee makes sense only if United see him as a left-sided 8, because his progressive carries and pressing numbers fit that role better than playing him as a pure winger."
- `hot_take`: "That manager is a fraud and only wins because he has expensive players."
- `hot_take`: "This club has no ambition; every transfer window is just PR and panic."
- `reaction`: "What a finish. I still cannot believe he scored from there."
- `reaction`: "That red card decision is ridiculous."

Hardest anticipated edge case: A post that starts as emotional frustration after a loss but also points to a tactical or selection issue, such as "This loss proves the manager has no idea how to close out big matches." This sits between `reaction` and `hot_take`, and it can become `analysis` if the author adds concrete reasoning. I will label it `reaction` when it is mainly fresh venting, `hot_take` when it turns into a broad unsupported judgment about the manager or club, and `analysis` only when it explains the issue with specific substitutions, formation changes, pressing choices, or chance quality.

## Milestone 2: Project Specification

### 1. Community

The chosen community is `r/soccer`, a public Reddit community focused on soccer news, results, clips, transfer rumors, statistics, quotes, and discussion. It is a good fit for classification because the same community contains very different kinds of discourse: tactical breakdowns, source-based transfer arguments, emotional match reactions, referee complaints, player debates, and sweeping club/manager judgments. That variety makes the learning task meaningful because a classifier can learn distinctions that regular readers already care about: whether a post is reasoned analysis, an unsupported hot take, or an immediate reaction.

### 2. Labels

The classifier will use three mutually exclusive labels.

`analysis`: A post or comment that makes a structured soccer argument using specific, relevant evidence such as tactics, player roles, match context, stats, injuries, transfer fees, or source reliability.

Examples:

- "Arsenal struggled after the 60th minute because the fullbacks stopped inverting, which left Rice isolated against two midfield runners and made every turnover turn into a counter."
- "That transfer fee makes sense only if United see him as a left-sided 8, because his progressive carries and pressing numbers fit that role better than playing him as a pure winger."

`hot_take`: A post or comment that makes a bold soccer claim with little support, vague support, or evidence used mainly to make the claim sound stronger.

Examples:

- "That manager is a fraud and only wins because he has expensive players."
- "This club has no ambition; every transfer window is just PR and panic."

`reaction`: A post or comment that mainly expresses immediate emotion about a goal, miss, referee decision, quote, transfer rumor, clip, or match result without making a broader supported argument.

Examples:

- "What a finish. I still cannot believe he scored from there."
- "That red card decision is ridiculous."

### 3. Hard Edge Cases

The hardest edge case will be emotional post-match criticism that gestures at tactics but does not clearly explain them, such as "This loss proves the manager has no idea how to close out big matches." This could be `reaction` because it is immediate frustration, `hot_take` because it makes a broad unsupported claim, or `analysis` if the author adds concrete evidence.

Annotation rule: label it `reaction` when the post is mainly fresh venting about the result or a single moment. Label it `hot_take` when it makes a durable broad claim about a player, manager, club, or fanbase without enough support. Label it `analysis` only when it explains the claim with specific substitutions, formation changes, pressing choices, chance quality, injury context, source reliability, or other checkable soccer evidence.

Specific difficult examples from annotation:

- "Ten Hag didn't have a great hand at his disposal but I don't think he should be completely exonerated from crisisism because of that. His tactics were suicidal and in the xg table we finished last season in a similar position to our actual position this season..." This could be `hot_take` because it calls the tactics "suicidal" and the United stint a "disaster," but I labeled it `analysis` because it gives concrete soccer reasoning using tactics, xG table comparison, trophies, and squad context.
- "Again, Amad has 2 more g/a than Kulu who is supposedly your best player, in 8 less games. Amad is miles clear of Kulu. Kulu is crazy overrated cause he scores some aesthetically pleasing goals." This could be `analysis` because it cites goal/assist numbers and games played, but I labeled it `hot_take` because the stat is used mainly to support a sweeping "miles clear" and "crazy overrated" claim without a broader role or performance argument.
- "3 goals in 5 minutes on extra time. Comeback in a European quarter final. Lord Maguire scores the final goal. Absolute scenes." This could be `analysis` because it mentions specific match context, but I labeled it `reaction` because the purpose is immediate excitement about a dramatic event, not a supported argument about tactics or player quality.

### 4. Data Collection Plan

Examples will be collected from public `r/soccer` posts and comments, especially Daily Discussion threads, match clips, post-match discussion, transfer rumor posts, official-source news posts, stats posts, and opinion-piece discussions. I will collect at least 200 total labeled examples before training, with a target of roughly 70 examples for `analysis`, 65 for `hot_take`, and 65 for `reaction`. The exact balance may vary slightly because the real subreddit distribution is not perfectly even, but no label should have fewer than 50 examples in the initial dataset.

During annotation, each row should include the text, assigned label, source thread type if available, and a short note for borderline examples. If one label is underrepresented after 200 examples, I will deliberately sample from thread types where that label is more likely: stats and long discussion threads for `analysis`, transfer rumors and opinion threads for `hot_take`, and match clips or post-match threads for `reaction`. If a label still has fewer than 50 examples after targeted sampling, I will either collect more than 200 examples or merge/redefine labels before training rather than train on a badly imbalanced taxonomy.

### 5. Evaluation Metrics

I will report accuracy, macro F1, per-label precision, per-label recall, and a confusion matrix. Accuracy is useful as a quick overall score, but it is not enough because a model could look accurate by over-predicting the most common label. Macro F1 is the main metric because each label matters equally for this project, even if the dataset is not perfectly balanced.

Per-label precision matters because false positives have different costs: labeling weak posts as `analysis` would make the tool overstate discourse quality, while labeling emotional reactions as `hot_take` could make normal fan excitement look worse than it is. Per-label recall matters because the classifier should actually find the categories it claims to measure, especially `analysis`, which is the most valuable category for surfacing substantive discussion. The confusion matrix will show which boundaries are failing, especially `reaction` vs. `hot_take` and `hot_take` vs. `analysis`.

### 6. Definition of Success

For the class project, "good enough" means at least 0.70 accuracy, at least 0.65 macro F1, and no individual label with F1 below 0.55 on the evaluation set. Those thresholds would show that the classifier learned more than a majority-class shortcut and can distinguish the three discourse types at a usable baseline level.

For deployment in a real community tool, the bar should be higher: at least 0.80 accuracy, at least 0.75 macro F1, and `analysis` precision of at least 0.80. `analysis` precision is especially important because a tool that highlights or rewards "analysis" should avoid promoting posts that are actually unsupported hot takes. At the end of the project, success can be judged objectively by comparing the evaluation metrics against these thresholds and reviewing the confusion matrix for repeated boundary failures.

### AI Tool Plan

#### Label Stress-Testing

Before annotating the full dataset, I will give an AI tool the three label definitions, the edge case rule, and the soccer community context. I will ask it to generate 5-10 borderline `r/soccer`-style posts that sit between `reaction` and `hot_take`, and between `hot_take` and `analysis`. I will manually classify those generated examples using the written rules. If I cannot classify them consistently, I will tighten the definitions before labeling real examples.

#### Annotation Assistance

I may use an LLM to pre-label a small batch of examples, but only as annotation assistance, not as ground truth. If I use pre-labeling, I will add a column such as `ai_prelabeled` or `prelabel_source` to track which rows were initially labeled by an AI tool, then I will manually review and accept or correct every label. If this adds more confusion than speed, I will skip pre-labeling and annotate manually; either way, the final dataset labels will be my reviewed labels.

#### Failure Analysis

After evaluation, I will give an AI tool a list of wrong predictions with the true label, predicted label, and text. I will ask it to identify recurring error patterns, such as confusing tactical complaints with hot takes, treating all referee complaints as reactions, or overvaluing posts that mention a stat without making a real argument. I will verify any suggested pattern myself by checking the examples and confusion matrix before using it in the final writeup.

#### Stretch Features

Before starting any stretch features, I will update this planning document with the new feature goal, how it changes the data or labels, and how success will be evaluated.

## Milestone 3: Dataset Collection and Annotation Log

### Collection Source

The labeled dataset was collected from public `r/soccer` comments using the PullPush Reddit archive API after direct Reddit JSON access returned a network-security block. The collected rows are public subreddit comments and include source permalinks back to old Reddit so examples can be inspected later.

Dataset file: `data/labeled_dataset.csv`

Columns:

- `text`: the post or comment text used as the model input.
- `label`: one of `analysis`, `hot_take`, or `reaction`.
- `notes`: annotation notes for difficult or borderline cases.
- `source_type`: whether the row came from a comment or post.
- `source_permalink`: public Reddit permalink for traceability.
- `created_utc`: original Reddit timestamp from the archive.

### Annotation Counts

The final dataset contains 210 labeled examples:

- `analysis`: 70 examples.
- `hot_take`: 70 examples.
- `reaction`: 70 examples.

No label accounts for more than 70% of the dataset. Each label accounts for exactly one third of the dataset, so there is no majority-class imbalance problem before training.

### Annotation Process

I collected examples with targeted search terms that matched the label definitions, then filtered out deleted, removed, bot-like, extremely short, overly long, duplicate, and low-information text. Labels were assigned using the definitions from this planning document: `analysis` required specific soccer reasoning or evidence, `hot_take` required a broad unsupported claim, and `reaction` required immediate emotional response without a broader supported argument.

I kept annotation notes for genuinely difficult cases. The final CSV includes 57 rows with notes, mostly for posts that sit between `hot_take` and `analysis` or between `reaction` and `hot_take`.

### Difficult Cases Encountered

The most common hard case was a short emotional criticism that also made a broad claim, such as calling a manager or player overrated after a match. I labeled these `reaction` when the text was mainly immediate emotion, `hot_take` when it made a durable unsupported judgment, and `analysis` only when it supplied specific soccer reasoning such as tactics, xG, role fit, substitutions, squad construction, or source reliability.

Another hard case was comments that mentioned a statistic but used it rhetorically rather than analytically. I labeled those `hot_take` unless the comment connected the statistic to a clear argument about performance, tactics, squad fit, or match context.

### AI Usage Disclosure

Codex assisted with data collection, filtering, CSV generation, and initial label assignment using the label definitions in this file. Borderline examples were marked in the `notes` column for disclosure and later review. The dataset should still be treated as human-review-required training data before final submission because annotation quality directly affects model quality.

## Baseline: Zero-Shot LLM

Before fine-tuning, I ran a zero-shot Groq LLM baseline on the locked test split created by the notebook. The test set contained 32 examples, and all 32 model responses were parseable label names.

Baseline results:

- Accuracy: 0.844.
- Macro F1: 0.84.
- Weighted F1: 0.84.
- `analysis`: precision 1.00, recall 0.70, F1 0.82, support 10.
- `hot_take`: precision 0.79, recall 1.00, F1 0.88, support 11.
- `reaction`: precision 0.82, recall 0.82, F1 0.82, support 11.

Baseline reflection: The zero-shot model is conservative about predicting `analysis`: when it predicts `analysis`, it is correct, but it misses 30% of true `analysis` examples. My hypothesis is that the baseline tends to classify borderline evidence-based posts as `hot_take` unless the reasoning is very explicit. Fine-tuning should improve `analysis` recall while keeping `analysis` precision high.

## Fine-Tuned Model Evaluation

I fine-tuned `distilbert-base-uncased` with the notebook defaults: 3 epochs, learning rate 2e-5, and batch size 16. I did not adjust hyperparameters.

Validation accuracy during training:

- Epoch 1: 0.516.
- Epoch 2: 0.516.
- Epoch 3: 0.548.

Fine-tuned test results:

- Accuracy: 0.469.
- Macro F1: 0.42.
- Weighted F1: 0.42.
- `analysis`: precision 0.40, recall 1.00, F1 0.57, support 10.
- `hot_take`: precision 0.50, recall 0.18, F1 0.27, support 11.
- `reaction`: precision 1.00, recall 0.27, F1 0.43, support 11.
- Wrong predictions: 17 of 32.

Fine-tuned reflection: The fine-tuned model performed worse than the zero-shot baseline. The main failure pattern is over-predicting `analysis`: it caught every true `analysis` example, but it also mislabeled many `hot_take` and `reaction` examples as `analysis`. This suggests the small DistilBERT model learned shallow cues such as soccer terms, numbers, player names, or match context as signals for `analysis`, rather than learning the stricter boundary that `analysis` requires a structured argument with relevant evidence.

## Files Tracked Outside The Notebook

- `planning.md`: Project plan and artifact checklist.
- `data/labeled_dataset.csv`: Labeled dataset used for training and evaluation.
- `outputs/evaluation_results.json`: Metrics exported from Colab after evaluation.
- `outputs/confusion_matrix.png`: Confusion matrix exported from Colab after evaluation.

## Workflow

1. Prepare the labeled CSV dataset.
2. Load the dataset in the Colab notebook.
3. Split the data into training and evaluation sets.
4. Train or configure the model.
5. Run predictions on the evaluation set.
6. Calculate evaluation metrics.
7. Export `evaluation_results.json`.
8. Export `confusion_matrix.png`.
9. Download both output files and commit them to this repository.

## Evaluation Outputs

`evaluation_results.json` should include the final metrics needed to summarize model quality, such as accuracy, precision, recall, F1 score, or any project-specific score.

`confusion_matrix.png` should visualize predicted labels against true labels so the common failure cases are easy to inspect.

## Submission Checklist

- [x] Create GitHub repository for non-notebook files.
- [x] Add `planning.md`.
- [x] Add labeled dataset CSV.
- [ ] Download `evaluation_results.json` from Colab.
- [ ] Download `confusion_matrix.png` from Colab.
- [ ] Commit and push final Colab output files.
