# YeastCaduceus

Fine-tuning [PlantCAD2-Small](https://huggingface.co/kuleshov-group/PlantCAD2-Small-l24-d0768) for *S. cerevisiae* regulatory genomics via LoRA.

**Goals:**
- Predict RNA-seq and ChIP-seq coverage from DNA sequence (512 bins × 16bp per 8,192bp window)
- Score eQTL variants zero-shot as `log p(ref) − log p(alt)`

**Status:** Phase 2 (MLM domain adaptation) running · Phase 3–4 pending

---

## Pipeline

| Phase | Notebook | Description | Status |
|---|---|---|---|
| 0 | `00_environment_smoke_test.ipynb` | Verify PlantCAD2-Small loads + forward pass | ✅ Complete |
| 0a | `00a_download_dataset.ipynb` | Download Shorkie dataset from GCS | ✅ Complete |
| 1 | `01_data_preprocessing.ipynb` | FASTAs + BigWigs → HuggingFace Datasets | ✅ Complete |
| 2 | `02_domain_adaptation_mlm_9_.ipynb` | MLM domain adaptation on 165 Saccharomycetales genomes | 🟡 Running |
| 3 | *(pending)* | Supervised fine-tuning on R64 + 4,201 BigWig tracks | ⬜ Pending |
| 4 | *(pending)* | Benchmarking vs Shorkie + DNABERT-2 | ⬜ Pending |
| 5 | *(pending)* | HuggingFace release + bioRxiv preprint | ⬜ Pending |

## Model

**Backbone:** `kuleshov-group/PlantCAD2-Small-l24-d0768` — 176M param BiMamba2, 8,192bp context, RC-equivariant.

**LoRA config** (from Kuleshov published adapters):
```
r=8, lora_alpha=32, dropout=0.1
target_modules: [out_proj, x_proj, in_proj]
trainable: 2.42M / 178.4M (1.36%)
```

## Data

Source: [Shorkie](https://github.com/calico/shorkie-paper) GCS bucket (`gs://shorkie-paper/data/`) via GCP project `caduceus-486605`.

| Dataset | Contents | Size |
|---|---|---|
| 165 Saccharomycetales genomes | MLM domain adaptation corpus | 2.11 GB |
| R64 reference | S. cer. supervised training | 0.01 GB |
| 80 S. cerevisiae strains | Strain diversity | 1.02 GB |
| BigWig tracks | RNA-seq + ChIP coverage targets | 4,201 tracks / 110 GB |

Chromosome split matches Shorkie exactly: val = {chrXI, chrXIII, chrXV}, test = {chrXII, chrXIV, chrXVI}.

## Phase 4 Benchmarks

| Benchmark | Dataset | Target |
|---|---|---|
| eQTL AUROC | Caudal et al. (~1,901 variants) | >0.85 (Shorkie: 0.85–0.87) |
| eQTL AUROC by TSS distance | Kita et al. (683 variants) | — |
| Expression Pearson | Val chroms | approach 0.78 (Shorkie baseline) |
| DREAM MPRA Spearman | Rafi et al. (71,103 seqs) | Novel benchmark |

## Requirements

All notebooks run on **Google Colab Pro A100**. See notebook Cell 1 for the mandatory 4-step install order (mamba-ssm CUDA kernel constraints require a specific pip sequence).

Key version pins: `torch==2.3.1+cu121`, `mamba-ssm==2.2.2`, `causal-conv1d==1.4.0`, `transformers==4.46.3`, `peft==0.14.0`.

## Reference

Built on the [Shorkie](https://github.com/calico/shorkie-paper) and [PlantCAD2](https://github.com/kuleshov-group/PlantCaduceus) frameworks.
SLIBTEC · 2026
