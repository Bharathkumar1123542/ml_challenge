# Business Entity Resolution — Pipeline README

## Overview

End-to-end entity resolution pipeline that matches Source 2 / Source 3 business
records against Source 1 reference entities, implemented as a single Jupyter
notebook (`src/business_entity_resolution/pipeline.ipynb`).

## Prerequisites

- Python 3.9+
- Jupyter / `nbconvert` (see `requirements.txt`)

## Installation

```bash
# From this directory (student_resource/code/business_entity_resolution/)
pip install -r requirements.txt
```

## Configuration

Edit `configs/pipeline.yaml` before running to set:
- `run_mode`: `"train"` (uses `dataset/train/`, prints F₀.₅) or `"test"` (writes final output)
- Blocking parameters, matching threshold, split ratio/seed, cache flags

## How to Run

### Interactive (Jupyter)

1. Open `src/business_entity_resolution/pipeline.ipynb` in JupyterLab or Jupyter Notebook.
2. Set `run_mode` in `configs/pipeline.yaml` to `"train"` or `"test"`.
3. **Kernel → Restart & Run All** — this is the only valid execution method.
   > ⚠️ Never execute individual cells out of order. The pipeline is designed as a
   > top-to-bottom linear run in a fresh kernel.

### Headless / CI (exact reproduce command)

```bash
# From student_resource/code/business_entity_resolution/
jupyter nbconvert \
  --to notebook \
  --execute \
  --inplace \
  --ExecutePreprocessor.timeout=8000 \
  src/business_entity_resolution/pipeline.ipynb
```

Both commands produce identical output given the same input data and the random
seed pinned in `configs/pipeline.yaml`.

## Output

Both output files are written to `student_resource/output/` (relative to the
workspace root):

| File | Description |
|---|---|
| `output/matching_results.tsv` | Final predicted matches — scored on the leaderboard |
| `output/candidate_pairs.tsv` | Blocking candidate set — audit trail |

## Validation

Run the organizer's validator from `student_resource/`:

```bash
python utils/validate_submission.py \
    --matching output/matching_results.tsv \
    --candidate output/candidate_pairs.tsv \
    --test-dir dataset/test
```

Exit code `0` / `PASS` = safe to submit.

## No-Network Check

Before packaging, verify the notebook contains no outbound network calls:

```bash
jupyter nbconvert --to script --stdout \
    src/business_entity_resolution/pipeline.ipynb \
    | grep -E "requests|httpx|urllib\.request|socket\."
```

Expect **no output**.

## Matching Model

- **Model:** GradientBoostingClassifier (scikit-learn)
- **License:** BSD-3-Clause (permissive; scikit-learn itself is BSD-3)
- **Parameter count:** N/A — classical ML, no pretrained weights
- **Satisfies constraint:** ✅ MIT/Apache-2.0-compatible; ≤ 8B parameters

## Notebook Sections (§0–§9)

| # | Section | Responsibility |
|---|---|---|
| §0 | Setup & Configuration | Imports, YAML load, dataclasses, random seed |
| §1 | Data Ingestion | Read TSVs, validate schemas, tag source |
| §2 | Normalization | Canonicalize names/addresses, country-dispatch |
| §3 | Blocking | Inverted-index blocking, emit candidate pairs |
| §4 | Feature Engineering | 7 similarity features per candidate pair |
| §5 | Matching Model | Train GBC classifier, score all pairs |
| §6 | Aggregation | Threshold, group by S1, enforce invariants |
| §7 | Output Writing | Write matching_results.tsv + candidate_pairs.tsv |
| §8 | Validation | Shell out to validate_submission.py |
| §9 | Evaluation | Macro-averaged F₀.₅ (train mode only) |
