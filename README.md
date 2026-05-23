# Module 7 Week B — Integration Task: Summarization & Integrated Evaluation Report

This is the starter repo for the Module 7 Week B Integration Task. **The integrated evaluation report you produce here is the M7 deliverable.**

The full integration guide is at <a href="https://levelup-applied-ai.github.io/aispire-14005-pages/modules/module-7/496c1c2b" target="_blank">the integration guide page</a> — read it first.

## Evaluation Details

This integration evaluates abstractive summarization using the **`sshleifer/distilbart-cnn-6-6`** model. This is a fast, distilled version of the BART architecture that has been fine-tuned specifically on the CNN/Daily Mail dataset, making it highly effective for generating concise news summaries.

The evaluation runs against a corpus of **120 technology and entertainment news articles** (a curated subset from the Module 6 `glnmario/news-qa-summarization` dataset). The model's generated summaries are evaluated against the human-authored golden references located in the `data/tech_news_summaries_reference.csv` file to calculate ROUGE-1, ROUGE-2, and ROUGE-L metrics.

To reproduce the evaluation and generate the predictions and metrics locally, simply use the provided Makefile command:
`make summarize`
*(Note: You can also run `python summarize.py` directly if you are on a Windows environment).*

## Quick start

```bash
pip install -r requirements.txt
make summarize    # runs full pipeline; first run downloads ~250 MB

```

The first call to `pipeline("summarization", ...)` downloads the model. Plan ~3 minutes for the first run; subsequent runs use cached weights. The full evaluation on 120 articles completes in ~6–8 minutes on CPU after the model is cached.

## What you will produce

Committed:

* `summarize.py` — your implementation
* Updated `README.md` — 1-2 paragraphs documenting the sshleifer/distilbart-cnn-6-6 model, the 120-article news corpus, and the reproduction command.
* `summary_predictions.csv` — 120 rows with reference, predicted, and per-summary ROUGE
* `summary_metrics.json` — aggregate ROUGE-1/2/L F1
* `integrated-evaluation-report.md` — six-section integrated report (the M7 deliverable). Includes an optional Section 7 (Challenge Extensions) for learners completing challenge tiers — see the integration's learner guide.

**No model file** — pre-trained model loads from Hugging Face Hub at runtime.

## Data

* `data/tech_news_articles.csv` — 1,033 tech / entertainment / digital-culture news articles, curated from glnmario/news-qa-summarization. The full pool is here for inspection and stretch use; the integration evaluates on the 120-article subset that has reference summaries.
* `data/tech_news_summaries_reference.csv` — 120 reference summaries (one per evaluated article), shipped with the curated dataset (CNN editor-authored summaries from the source dataset).
* `data/tiny_articles_smoke.csv` + `data/tiny_refs_smoke.csv` — 3-row CI smoke fixtures (articles and references in separate files, matching the real-data schema).

## Make targets

```bash
make summarize    # full pipeline against the 120-article evaluation set
make smoke        # CI-only target — 3-row fixture
make clean        # remove generated outputs

```

## Submission

Open a Pull Request from your working branch into `main`. The autograder runs `make smoke` against the 3-row fixture and validates artifact schemas. PR description requirements are in the integration guide.

---

## License

This repository is provided for educational use only. See [LICENSE](https://www.google.com/search?q=LICENSE) for terms.

You may clone and modify this repository for personal learning and practice, and reference code you wrote here in your professional portfolio. Redistribution outside this course is not permitted.