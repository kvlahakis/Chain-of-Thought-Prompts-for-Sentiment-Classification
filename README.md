# Chain-of-Thought Prompts for Sentiment Classification

**A Controlled Evaluation on SST-2**

Kieran Vlahakis, Noor Ibrahim, Ritvik Teegavarapu, Annika Viswesh
California Institute of Technology — CS 159

[](https://github.com/kvlahakis/Chain-of-Thought-Prompts-for-Sentiment-Classification)

---

## Overview

Chain-of-thought (CoT) prompting is well known to boost performance on arithmetic and commonsense reasoning benchmarks, but its effect on short-text, subjective classification tasks is comparatively unexplored. This project presents a controlled empirical study comparing three prompting strategies on a balanced 400-example subset of the Stanford Sentiment Treebank (SST-2), all evaluated with `gpt-4o-mini`:

- **Direct** — zero-shot, immediate binary decision (T = 0)
- **CoT** — zero-shot chain-of-thought ("Let's think step by step...") (T = 0)
- **SC-CoT** — self-consistency CoT, majority vote over N = 7 sampled reasoning chains (T = 0.7)

The goal is to isolate the effect of explicit reasoning — holding model, system prompt, and post-processing constant — and to characterize *when* and *why* reasoning helps or hurts on affective, lexically-driven classification tasks.

## Key Findings

| Strategy | Accuracy | Wilson 95% CI    | Cost Ratio |
|----------|---------:|------------------|:----------:|
| Direct   | 0.908    | (0.875, 0.932)   | ×1         |
| CoT      | 0.918    | (0.886, 0.941)   | ×1         |
| SC-CoT   | 0.898    | (0.863, 0.924)   | ×7         |

- CoT yields a modest **+1.0 pp** gain over Direct prompting, but the difference is **not statistically significant** (paired McNemar, *p* = 0.454).
- SC-CoT **underperforms** single-sample CoT despite a 7× increase in API cost (*p* = 0.057 vs. CoT).
- Bootstrap analysis (10,000 replications) corroborates the McNemar results: both the CoT-vs-Direct and SC-CoT-vs-Direct 95% confidence intervals for the paired risk difference span zero.
- Error-overlap analysis shows **52.5%** of all misclassifications are shared across all three strategies, suggesting an intrinsic difficulty in certain snippets rather than a strategy-specific weakness.
- Qualitative analysis suggests CoT is most helpful on reviews with subtle negation, while SC-CoT is vulnerable to "spurious unanimity" — majority votes swayed by a single salient lexical cue (e.g., "unbearably").

**Takeaway:** In lexically-driven, short-form sentiment classification, explicit reasoning provides only marginal benefit, and self-consistency aggregation is unlikely to be worth its added cost.

## Repository Structure

```
.
├── data/                # 400-example balanced SST-2 subset (200 pos / 200 neg)
├── prompts/             # Prompt templates for Direct, CoT, and SC-CoT (Appendix A.1)
├── src/                 # Evaluation pipeline: querying, parsing, scoring
├── notebooks/           # Jupyter notebooks for visual diagnostics and figures
├── results/             # Raw completions, extracted labels, reasoning chains
└── README.md
```

*(Adjust the tree above to match the actual repo layout.)*

## Methodology

- **Dataset:** 200 positive + 200 negative snippets sampled from the SST-2 train split (fixed seed), capped at 20 words per snippet.
- **Model:** `gpt-4o-mini` for all conditions.
- **Prompting:** All prompts share an identical system prefix and tag-rule suffix (`##LABEL## POSITIVE/NEGATIVE`), differing only in whether reasoning is elicited and whether multiple chains are aggregated. See `prompts/` or Appendix A.1 of the report for verbatim templates.
- **Temperature:** Direct and CoT use T = 0 (deterministic, single call); SC-CoT uses T = 0.7 across N = 7 samples with majority-vote aggregation (ties broken randomly).
- **Metrics:**
  - Exact-match accuracy with Wilson 95% confidence intervals
  - Paired McNemar's test (continuity-corrected χ² for b + c ≥ 25, exact binomial otherwise) on the 391/400 snippets that received valid labels from all three strategies
  - Bootstrap-estimated paired risk differences (10,000 replications)
  - Confusion matrices and misclassification-overlap (Venn diagram) analysis

## Getting Started

```bash
git clone https://github.com/kvlahakis/Chain-of-Thought-Prompts-for-Sentiment-Classification.git
cd Chain-of-Thought-Prompts-for-Sentiment-Classification
pip install -r requirements.txt
```

Set your OpenAI API key as an environment variable:

```bash
export OPENAI_API_KEY="your-key-here"
```

Run the evaluation pipeline:

```bash
python src/run_evaluation.py --strategy direct
python src/run_evaluation.py --strategy cot
python src/run_evaluation.py --strategy sc_cot --n_samples 7 --temperature 0.7
```

Reproduce the figures and statistical tests:

```bash
jupyter notebook notebooks/analysis.ipynb
```

*(Update commands/flags to match your actual scripts.)*

## Limitations

- Single model architecture (`gpt-4o-mini`); results may not generalize to other LLMs (e.g., Mistral, Falcon).
- Binary-only SST-2 subset — no multi-sentence context or neutral-class examples.
- SC-CoT evaluated only at a fixed sample size (N = 7); sensitivity to N is unexplored.

## Future Work

- Replicate across open-weight models (Mistral, Falcon) to test model-dependence of CoT gains.
- Extend to multilingual and domain-specific sentiment datasets with less direct lexical cues.
- Sweep the number of self-consistency samples to disentangle under-sampling from inherent redundancy.

## Citation

If you use this work, please cite:

```bibtex
@techreport{vlahakis2024cot,
  title        = {Chain-of-Thought Prompts for Sentiment Classification: A Controlled Evaluation on SST-2},
  author       = {Vlahakis, Kieran and Ibrahim, Noor and Teegavarapu, Ritvik and Viswesh, Annika},
  institution  = {California Institute of Technology, CS 159},
  year         = {2024}
}
```

## License 

This project is licensed under the MIT License.
 
```
MIT License
 
Copyright (c) 2024 Kieran Vlahakis, Noor Ibrahim, Ritvik Teegavarapu, Annika Viswesh

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
 
The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
 
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
