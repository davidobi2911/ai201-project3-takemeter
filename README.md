# TakeMeter

TakeMeter is the non-notebook repository for AI201 Project 3. It contains the planning notes, labeled dataset, and exported evaluation artifacts used for the Colab model workflow.

## Repository Contents

- `planning.md`: Project plan, model workflow, evaluation approach, and artifact checklist.
- `data/labeled_dataset.csv`: Labeled dataset used by the notebook.
- `outputs/evaluation_results.json`: Evaluation metrics exported from Colab.
- `outputs/confusion_matrix.png`: Confusion matrix image exported from Colab.

## Colab Output Artifacts

After running the notebook in Colab, download these files into `outputs/`:

```text
outputs/evaluation_results.json
outputs/confusion_matrix.png
```

The repository intentionally excludes the notebook itself so this repo only tracks the supporting project files and final outputs.

## Reproducing The Artifact Set

1. Confirm the labeled dataset is present at `data/labeled_dataset.csv`.
2. Run the TakeMeter notebook in Colab.
3. Download the evaluation JSON and confusion matrix PNG from Colab.
4. Place both files in `outputs/`.
5. Commit the new outputs to GitHub.
