# Dissertation

# A Multi-Agent LLM Benchmark for Clinical Guideline Adherence

This repository contains the pipeline developed for an MSc Health Data Science
dissertation. It automatically generates a clinical benchmark from NICE
guidelines and evaluates large language models (LLMs) on their adherence to
those guidelines.

## Overview

The pipeline has two purposes:

1. **Benchmark creation** — from a NICE guideline, it generates clinical
   vignettes and questions, plus adversarial *corner-case* and *counterfactual*
   robustness items. Every generated item is quality-scored by a four-agent
   majority-voting jury (MAJ) and filtered against quality thresholds.
2. **LLM evaluation** — a set of LLMs answer the benchmark questions, and their
   answers are scored (adherence, completeness, safety) by the same MAJ jury.

The MAJ jury comprises four Claude Haiku agents with distinct clinical personas.
Each item is scored independently, then targeted consensus rounds resolve any
criteria where fewer than three of four agents agree. Scores are aggregated by
mean.

## Pipeline stages

| Stage | Description |
|-------|-------------|
| 1 | Load and preprocess the guideline HTML into clean text |
| 2 | Generate vignettes and questions (DeepSeek) |
| 3 | MAJ-score vignette quality (V1–V4) and question quality (Q1–Q4, P1) |
| 3b | Select the best question per recommendation |
| 4 | Filter items against quality thresholds |
| 5 | Generate corner-case and counterfactual robustness items |
| 6 | MAJ-score robustness items (CC1–CC2, CF1–CF2) |
| 6b | MAJ-score clinical gravity of robustness items |
| 7 | Evaluate LLMs on the benchmark |
| 8 | MAJ-score the LLM answers (adherence, completeness, safety) |
| 9 | Assemble results and export to a formatted Excel workbook |

## Scoring rubrics

All scoring criteria and their 0–2 level definitions are documented in
`docs/scoring_rubrics.docx` (or see the methods chapter of the dissertation).

## Requirements

Install dependencies:

```bash
pip install -r requirements.txt
```

The pipeline is written for Google Colab (it mounts Google Drive and reads API
keys from Colab secrets), but the core class runs anywhere with the listed
packages.

## Setup

1. **API keys.** The pipeline calls several model providers. Set the following
   as Colab secrets (or environment variables if running locally) — do **not**
   hardcode them:
   - `ANTHROPIC_KEY` — Anthropic (MAJ judge, Claude Haiku)
   - `DEEPSEEK_KEY` — OpenRouter/DeepSeek (vignette generation)
   - `OPENAI_KEY` — OpenAI (evaluated model)
   - `GOOGLE_KEY` — Google Gemini (evaluated model)
   - `TOGETHER_KEY` — Together AI (evaluated model, Qwen)
   - `HF_KEY` and `MEDGEMMA_URL` — Hugging Face inference endpoint (MedGemma)

2. **Guideline files.** The NICE guideline HTML files are **not** included in
   this repository, as NICE guidance is copyrighted. Obtain the guidelines
   directly from https://www.nice.org.uk and place the HTML files in your data
   folder. The guidelines used are listed in the `all_guidelines` cell.

3. **Data path.** Set `BASE` to your own data folder.

## Running

The notebook is organised so each stage runs in its own cell, per guideline,
with intermediate results checkpointed to disk (JSON) after each stage. This
allows a run to be resumed from the last completed stage if a session
disconnects. To run a guideline end to end, set its name in the config cell,
then run the stage cells in order. Each stage cell reloads its inputs from the
previous stage's checkpoint.

## Outputs

- Per-stage JSON checkpoints (in `checkpoints/<guideline>/`)
- A formatted Excel workbook per guideline, and a combined workbook, containing
  the benchmark items, quality scores, robustness scores, and LLM evaluation
  results.

## Limitations

- **Single-model jury.** The four MAJ agents are personas of one underlying
  model (Claude Haiku), so they share that model's biases; consensus mitigates
  persona-level variance but not model-level bias. Inter-rater agreement
  therefore reflects internal consistency of the scoring protocol rather than
  fully independent judgement.
- Generated content is machine-produced and quality-filtered but not clinically
  validated by a human expert across the full set.
- Guideline text is truncated to a fixed character budget where required; very
  long guidelines may not be fully represented.

## Licence

Code is released under the MIT Licence (see `LICENSE`). NICE guideline content
is not included and remains the property of NICE.
