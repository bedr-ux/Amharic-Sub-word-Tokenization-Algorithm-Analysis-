# Amharic Sub-word Tokenization Algorithm Analysis

**Empirical Evaluation of Subword Tokenization on MasakhaNEWS (Amharic & English)**
WordPiece vs. Byte-Level BPE vs. Unigram, compared against tokenizers from widely used LLMs.

**Author:** Bedru Yimam Ahmed, Department of Information Technology, Wollo University (KIoT), Ethiopia

## Motivation
Tokenizers sit between raw text and every LLM. Most were built mainly on English-centric data, so Amharic (Ethiopic script, a morphologically rich abugida) is split into far more tokens than English. That "token tax" means higher cost, shorter effective context and weaker modeling. This project measures the tax and tests what reduces it.

## Setup
- **Data:** [MasakhaNEWS](https://huggingface.co/datasets/masakhane/masakhanews), official splits.
  - Amharic: 1,311 train / 376 test articles
  - English: 3,309 train / 948 test articles
- **Training:** tokenizers are trained on `train` only; all metrics are on held-out `test`.
- **Normalization:** Amharic Fidel homophone merging (ሐ/ኀ/ሀ, ሠ/ሰ, ዐ/አ, ፀ/ጸ) and Ethiopic punctuation isolation.
- **Algorithms:** WordPiece, Byte-Level BPE, Unigram (Hugging Face `tokenizers`), default vocabulary 16k.
- **Metrics:** fertility (tokens/word, lower is better), bytes/token, chars/token, OOV rate, vocabulary utilization, and the Amharic/English fertility ratio ("token tax").

## Results (test sets, V = 16k)

| Tokenizer | Amharic fertility | Amharic bytes/token | English fertility |
|---|---|---|---|
| WordPiece | 1.400 | 8.927 | 1.161 |
| Byte-Level BPE | **1.399** | **8.931** | **1.136** |
| Unigram | 1.593 | 7.846 | 1.384 |

### Token tax versus pretrained LLM tokenizers

| Tokenizer | Amharic fertility | English fertility | Tax ratio |
|---|---|---|---|
| Our BPE (16k, trained on MasakhaNEWS) | 1.399 | 1.136 | 1.23 |
| Qwen-2.5 (152k) | 6.274 | 1.124 | 5.58 |
| GPT-4o (`o200k_base`) | 8.646 | 1.091 | 7.92 |
| Mistral-7B (32k) | 10.602 | 1.217 | 8.71 |
| GPT-4 (`cl100k_base`) | 11.369 | 1.105 | 10.29 |

Pretrained LLM tokenizers need roughly 4.5x to 8x more tokens per Amharic word than a small in-domain tokenizer.

### Ablations (Amharic)
- **Vocabulary size:** BPE fertility falls from 1.856 (4k) to 1.585 (8k), 1.399 (16k) and 1.277 (32k), with diminishing returns and falling vocabulary utilization (95.8% to 71.7%).
- **Algorithm:** WordPiece and BPE are nearly tied at every size; Unigram is consistently worse at the same vocabulary size.
- **Normalization (BPE):** fertility 1.529 raw, 1.514 with Fidel merging only, 1.399 with full normalization.
- **Monolingual vs. joint bilingual (16k):** Amharic fertility rises from 1.399 to 1.639 when the vocabulary is shared with English; English rises from 1.136 to 1.197.

## Caveats
- The comparison is not like-for-like: the 16k tokenizer is trained on the same news domain it is tested on, while the LLM tokenizers are general-purpose. Read the tax ratios as a measure of coverage, not of tokenizer quality.
- The normalization ablation changes whitespace word counts (punctuation isolation), so fertility is not strictly comparable across conditions. Bytes/token and total token counts are the safer cross-check.
- The Amharic corpus is small (about 0.5M training words); results at larger vocabularies may reflect data size.
- Gated models (LLaMA-3, Gemma-2) need a Hugging Face login to load.

## Run it
```bash
pip install -r requirements.txt
jupyter notebook tokenizer_evaluation_masakhanews.ipynb
```
Also runs in Google Colab. The notebook saves its summary figure as `tokenizer_evaluation_summary.png`.

## Citation
Please cite this repository and the MasakhaNEWS dataset (Adelani et al., 2023).
