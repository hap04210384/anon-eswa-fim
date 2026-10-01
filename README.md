# anytime-failure-mining

Companion replication package for the paper **"Threshold-Free Anytime Discovery
of Recurrent Failure Patterns in the Trajectories of Large Language Model
Agents"** (under review). This repository contains the complete pipeline,
encoded datasets, and all result files reported in the paper.

## Contents

```
pipeline/    experiment scripts (encoding, mining, guardrails, ablations, figures)
data/        encoded transaction/sequence files (small, committable) + raw-data downloaders
results/     every numeric result reported in the paper, as machine-readable JSON/JSONL
engine/      AnyFIM engine source snapshot (CUDA/VS solution) that produced the itemset streams
annotation/  failure-trajectory annotation kit (guide, booklet, blank sheets, adjudicated codes; study complete, Cohen's kappa = 1.0)
docs/        RESULTS_MANIFEST.md (MD5 anchors for every packaged result file)
```

## Requirements

- Python 3.10+ (standard scientific stack: numpy, pandas)
- Windows + Visual Studio 2022 + CUDA 12.x **only for rebuilding the itemset
  engine** (`engine/AnyFIM-Anytime`); all Python-layer experiments run on any
  OS and do not require a GPU.
- The classical-miner baselines use the SPMF open-source library v2 (not
  bundled; download from spmf.ca and point `pipeline/run_baselines.py` at it).
- The learned-monitor baseline uses the public PrefixGuard code (MIT; see
  `pipeline/convert_swebench_prefixguard.py` for the adaptation).

## Quick start (verify the packaged numbers in minutes)

```bash
python - <<'PY'
import hashlib, json, pathlib
man = json.load(open("docs/manifest.json"))
bad = []
for rel, md5 in man.items():
    p = pathlib.Path(rel)
    if not p.exists() or hashlib.md5(p.read_bytes()).hexdigest() != md5:
        bad.append(rel)
print("OK: all packaged files intact" if not bad else f"MISMATCH: {bad}")
PY
python pipeline/guardrail_eval.py   # recompute Table 5 from the packaged encodings
```

## Reproduction map (claim in paper -> script -> packaged output)

| Paper location | Claim | Script | Packaged output |
|---|---|---|---|
| Table 2 | corpus statistics | `pipeline/encode_swebench.py` (encoding) | `data/encoded/labels_*.csv` |
| 6.1 | 72,972 chains, contrast distribution | `pipeline/sequence_layer.py`, `pipeline/annotate_signatures.py` | `results/chains.jsonl`, `results/signatures.jsonl` |
| Figure 2 | contrast profile | `pipeline/make_fig_contrast.py` | EMF figure |
| Table 3 | six-mode failure-mode catalog | `pipeline/m2_sample_annotation.py`, `pipeline/compute_kappa.py` | `annotation/`, `results/m2_kappa.json` |
| Figure 3 | human-catalog agreement (hit-rate, mode distribution) | `pipeline/make_fig3_hitrate.py` | `results/m2_hitrate.json` |
| Figure 4 | anytime quality-budget curve | `pipeline/make_fig3.py` (+ engine stream, `results/baselines_*.json`) | EMF figure |
| Figure 5 | 29-stage stream anatomy | `pipeline/make_fig5_stages.py` (engine per-stage Results file in `engine/`) | EMF figure |
| Table 4 | baseline grid, representative rows | `pipeline/run_baselines.py` | `results/baselines_*.json` |
| Figure 6 | guardrail operating curves | `pipeline/make_fig6_guardrail.py` | `results/guardrail_eval.json`, `results/prefixguard_eval.json` |
| Table 5 | guardrail sweep | `pipeline/guardrail_eval.py` | `results/guardrail_eval.json` |
| Table 6 | PrefixGuard side-by-side | `pipeline/eval_prefixguard_swebench.py` (external PrefixGuard repo) | `results/prefixguard_eval.json` |
| 6.3 | learned-monitor baseline | `pipeline/convert_swebench_prefixguard.py`, `pipeline/eval_prefixguard_swebench.py` (external PrefixGuard repo) | `results/prefixguard_eval.json` |
| Figure 7 | mining-pool sensitivity heatmap | `pipeline/make_fig7_heatmap.py` | `results/m4a_recall_sweep.json` |
| Table 7 | WebArena cross-corpus contrast | `pipeline/m3_webarena_mining.py` | `results/webarena_mining.json` |
| 6.4 | projection/support-floor/mining-pool ablations | `pipeline/ablation_projection.py`, `pipeline/sensitivity_floor.py`, `pipeline/m4a_recall_sweep.py` | `results/ablation_projection.json`, `results/sensitivity_floor.json`, `results/m4a_recall_sweep.json` |
| 6.5 | WebArena cross-corpus study | `pipeline/encode_webarena.py`, engine (WebArena config, see engine README), `pipeline/m3_webarena_mining.py` | `results/webarena_*.json(l)` |
| 5.3 | annotation study (complete, kappa = 1.0) | `pipeline/m2_sample_annotation.py`, `pipeline/compute_kappa.py` | `annotation/` |

## Data provenance

- **SWE-bench Verified trajectories**: public Hugging Face mirror
  `tarsur385/swebench-verified-trajectories` (5,000 trajectories, ten agents,
  accessed 2026-09; programmatic outcome labels, no human subjects).
  `pipeline/download_swebench.py` rebuilds the raw corpus.
- **WebArena GPT-4 CoT release 2**: 809 trajectories via the
  `Agent-Eval-Refine/Agent-Trajectories` Hugging Face dataset
  (`data/dl_webarena.cmd` downloads the archive; labels from the official
  WebArena evaluation release).
- Encoded files in `data/` are the exact inputs to every experiment; the raw
  corpora are not bundled (multi-GB) but are reconstructible bit-for-bit from
  the download scripts.

## Engine build notes (Windows only)

Open `engine/AnyFIM-Anytime/SustainableMining.sln` in Visual Studio 2022 with
CUDA 12.x installed and build `x64|Release`. The packaged `kernel.cu` defaults
to the SWE-bench failing-pool configuration (`swebench_mine.txt`, 29 stages);
a commented line switches it to the WebArena configuration (`webarena_mine.txt`,
19 stages) used for Section 6.5. Input transaction files and the packaged per-stage stream outputs live in
`engine/AnyFIM-Anytime/AnytimeMining/x64/TransactionSets/`.

## License

MIT (see LICENSE). The PrefixGuard and SPMF baselines retain their own
licenses and are not bundled.
