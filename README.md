# Brain-alignment-and-context-length

Update the `DRIVE` and `DATA` path variables at the top of each notebook to 
point to wherever you placed `lpp_data/`.

---

## Run Order

Run notebooks in the following order. Each saves results to a `results/` folder 
that subsequent notebooks load from.

### 1. `embedding_extraction.ipynb`
Extracts GPT-2 medium (layer 24) word embeddings for all 6 context sizes  
{1, 5, 20, 50, 100, full} across runs 15–18.  
**Saves:** `results/gpt2_word_embeddings_run{15,16,17,18}.npz`  
**Requires:** BOLD files + annotation CSV. GPU recommended.

### 2. `voxelwise_ridgeregression.ipynb`
Fits voxelwise ridge regression encoding models for all context sizes.  
Runs 5-fold CV on concatenated 4-run time series.  
**Saves:** `results/r2_scores_4runs.npz`  
**Requires:** output of step 1 + BOLD files.

### 3. `layer_sweep.ipynb`
Re-extracts embeddings at layers {0,3,6,9,12,15,18,21,24} and fits one 
encoding model per layer.  
**Saves:** `results/r2_layer_sweep.npz`  
**Requires:** BOLD files + annotation CSV. GPU recommended.

### 4. `context_saturation.ipynb`
Constructs the voxelwise saturation map from the context sweep results.  
**Saves:** `results/saturation_map_EN057.nii.gz`  
**Requires:** output of step 2.

### 5. `comparing_glove_baseline.ipynb`
Fits encoding models using GloVe embeddings and compares spatial extent  
and voxelwise ordering against GPT-2.  
**Saves:** `results/r2_glove_vs_gpt2.npz`  
**Requires:** BOLD files + GloVe annotation CSV.

### 6. `surprisal.ipynb`
Computes token-level GPT-2 surprisal for all 6,683 words across sections 1–4.  
Correlates surprisal voxelwise with BOLD signal.  
**Saves:** `results/surprisal_4sections.npy`, `results/surprisal_r_map.npy`  
**Requires:** BOLD files + annotation CSV. GPU recommended.

### 7. `linguistic_analysis.ipynb`
Decomposes surprisal by POS group (NOUN/PROPN, VERB/AUX, ADJ, FUNC).  
Produces the POS surprisal bar chart.  
**Requires:** output of step 6 + annotation CSV.

### 8. `coreference.ipynb`
Tests whether coreference words show higher alignment at longer context sizes.  
**Requires:** output of step 2 + `lppEN_coreference.csv`.

### 9. `multiple_subjects.ipynb`
Fits encoding models for EN057, EN058, EN062, EN065 (runs 15–16).  
Computes top-500 voxel overlap across subject pairs.  
**Requires:** BOLD files for all four subjects + output of embedding extraction.  
 Results are inflated due to temporal autocorrelation — see paper Section 3.5.

### 10. `End-to-end` / `data_interpretation.ipynb`
Aggregates results and produces all final figures used in the paper.  
**Requires:** all `results/` files from steps 1–9.

---

## Dependencies

```bash
pip install numpy pandas scipy scikit-learn nibabel matplotlib \
            torch transformers tqdm
```

All notebooks were developed in Google Colab with a T4 GPU.  
Steps 1, 3, and 6 are slow on CPU (expect 30–60 min each without GPU).

---

## Results Structure

results/
├── gpt2_word_embeddings_run{15,16,17,18}.npz
├── r2_scores_4runs.npz
├── r2_layer_sweep.npz
├── r2_glove_vs_gpt2.npz
├── saturation_map_EN057.nii.gz
├── surprisal_4sections.npy
└── surprisal_r_map.npy

## Reference

Goldstein, A. et al. (2022). Shared computational principles for language  
processing in humans and deep language models. *Nature Neuroscience*, 25, 369–380.

Dataset: Bhattasali, S. et al. (2020). The Little Prince fMRI corpus. *LREC*.  
OpenNeuro: [ds003643](https://openneuro.org/datasets/ds003643)
