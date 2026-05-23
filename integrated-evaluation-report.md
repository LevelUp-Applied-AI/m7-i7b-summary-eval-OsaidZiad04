# Module 7 Integrated Evaluation Report — Fine-Tuning vs. Pre-Trained Inference

> The Module 7 deliverable. Synthesizes Lab 7A (fine-tuning), Integration 7A (domain shift), Lab 7B (QA), and Integration 7B (summarization).

---

## 1. Comparison Table

| Task | Approach | Model | Training cost | Inference cost | Quality metric | Value |
|---|---|---|---|---|---|---|
| Sentiment classification (Lab 7A) | Fine-tuning | DistilBERT | ~30 min CPU + 3K labels | ~50 ms / example | Macro-F1 | **[Insert your Lab 7A Macro-F1, e.g., 0.88]** |
| Domain transfer (Integration 7A) | Fine-tuned model out-of-domain | (same) | already trained | ~50 ms / example | Domain-shift judgment | **[Insert brief judgment, e.g., Performance degraded noticeably on long news text compared to short app reviews]** |
| Extractive QA (Lab 7B) | Pre-trained inference | distilbert-base-cased-distilled-squad | 0 | ~50 ms / example | EM / token-F1 | EM: 0.34 / F1: 0.46 |
| Summarization (Integration 7B) | Pre-trained inference | distilbart-cnn-6-6 | 0 | ~3 sec / example | ROUGE-1 / 2 / L F1 | R1: 0.3683 / R2: 0.1572 / RL: 0.2670 |

## 2. Findings

- **Fine-tuning** produced strong domain-specific results on short app reviews (Macro-F1: [Insert Lab 7A Macro-F1]), but struggles when transferred to out-of-domain tech news, highlighting the brittleness of task-specific classifiers.
- **Pre-trained QA** achieved moderate overlap (F1: 0.46) but low exactness (EM: 0.34). While it successfully extracts short entities, it frequently fails on long descriptive spans or embedded quotes due to boundary compression and quote-answer drift.
- **Pre-trained Summarization** achieved reasonable lexical overlap (ROUGE-1: ~0.37). However, ROUGE-2 (~0.16) indicates that exact phrasing matches are low, confirming the model's highly abstractive nature which reorganizes sentences entirely rather than copying them.

## 3. Faithfulness Check

### Example A — high ROUGE (Article ID: NEWS_0001)

> **Article excerpt / Reference:** "In 1977 director Roman Polanski pleaded guilty to having sex with a 13-year-old girl . He has lived in France for decades to avoid arrest if he enters the U.S. Oscar winner was arrested en route to Zurich Film Festival, which was honoring him."
> **Predicted summary:** "NEW: Roman Polanski arrested in Switzerland on a decades-old arrest warrant stemming from a sex charge in California . Polanski, 76, was arrested trying to enter Switzerland on Saturday, Zurich police say . He has lived in France for decades to avoid being arrested if he enters the United States . He pleaded guilty in 1977 to"
> **ROUGE-1:** 0.5172 ; **ROUGE-2:** 0.2807 ; **ROUGE-L:** 0.3275
> **Faithful?** Yes. The summary perfectly captures the core facts (arrest in Switzerland, decades in France, guilty plea in 1977) without inventing details. ROUGE correctly rewarded this summary because of the high n-gram overlap, even though the text abruptly truncates at the end due to the `max_length` limit.

### Example B — mid ROUGE (Article ID: NEWS_0004)

> **Article excerpt / Reference:** "Forbes crowns American investor Warren Buffett as world's richest person . After 13 years on top, Microsoft's Bill Gates drops to number three position . Russia replaces Germany as No. 2 country with 87 billionaires."
> **Predicted summary:** "American investor Warren Buffett is worth an estimated $62 billion, up $10 billion from a year ago . Bill Gates, co-founder of Microsoft, is now ranked as the world's third richest person . For the first time, Forbes' rich list named more than 1,000 billionaires from around the world . Russia is now"
> **ROUGE-1:** 0.3400 ; **ROUGE-2:** 0.1224 ; **ROUGE-L:** 0.2000
> **Faithful?** Yes. The model successfully extracted factually correct information (Buffett's wealth, Gates' rank, Russia's status) likely present in the original article body. However, ROUGE scores are moderate because the model used specific numbers ($62 billion) instead of the generalized wording found in the reference summary, penalizing it despite being factually accurate.

### Example C — low ROUGE (Article ID: NEWS_0003)

> **Article excerpt / Reference:** "Some industry heavy hitters support Roman Polanski after his arrest . Many in the international film community continue to hail him as a hero . Polanski has remained a Hollywood insider while living as a fugitive ."
> **Predicted summary:** "Swiss filmmaker Roman Polanski was arrested Saturday in Switzerland . He was on his way to accept an award for Lifetime Achievement at the Zurich Film Festival . "He's a brilliant guy, and he made a little mistake 32 years ago," says filmmaker Otto Weisser ."
> **ROUGE-1:** 0.2417 ; **ROUGE-2:** 0.0449 ; **ROUGE-L:** 0.1758
> **Faithful?** No. The model hallucinates/misattributes a key fact right at the beginning by calling him a "Swiss filmmaker" (he is Polish/French), likely confusing his nationality with the location of his arrest. Furthermore, it focuses entirely on a specific quote rather than summarizing the general support mentioned in the reference. The low ROUGE score here successfully reflects both the lack of alignment and the hallucination.

## 4. Production Decision Matrix

| Scenario | Recommendation | Justification |
|---|---|---|
| Real-time app store review triage dashboard for a product team | Fine-tuning | We have labeled data and need high precision (Macro-F1: [Insert Lab 7A Macro-F1]) to inform product decisions reliably. |
| Daily tech / entertainment news summary digest for an internal newsroom | Pre-trained inference | The baseline ROUGE-1 of 0.37 and ROUGE-L of 0.27 are sufficient for an internal gist-reading digest without the high cost of manual summarization labeling. |
| Domain-expert QA on legal contracts | Fine-tuning | The pre-trained QA model's low Exact-Match (0.34) and boundary compression errors pose an unacceptable risk for strict legal compliance. |

## 5. What You Would Do Differently

If I had a labeled summarization dataset for tech and entertainment news, I would not necessarily fine-tune the entire encoder-decoder model right away due to the massive compute cost. Instead, I would invest in training a smaller, specialized "Faithfulness Calibration" classifier on top of the pre-trained outputs. Since ROUGE does not catch hallucinations, having a dedicated guardrail model to flag unfaithful summaries before publication would yield the highest operational ROI and directly address the safety gap.

## 6. Limits of the Evaluation

These numbers do not tell us the full story about production safety. ROUGE only measures n-gram overlap, meaning a model could theoretically hallucinate a completely wrong entity but still score high if it reuses surrounding context words. Additionally, EM and F1 do not capture calibration; the QA model might be highly confident even when predicting an incorrect, truncated span, making it dangerous to deploy autonomously without a measured confidence-threshold filter.