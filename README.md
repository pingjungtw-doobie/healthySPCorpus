# Path Corpus Analysis — Language & MD Networks

Companion code for the manuscript:

> **Severe Aphasia After Small Bilateral Lesions: A Case Study of Disrupted High-Probability Structural Pathways** *(under review)*

`SPL_Corpus_Language_MD_final.ipynb` builds a **probabilistic corpus of weighted shortest paths** across all 116 AAL brain regions, aggregated over a large healthy fMRI sample. The corpus captures, for every ordered pair of AAL regions, which shortest paths are most likely to be traversed in a healthy brain and how discoverable each path is under a random walk. This corpus is the reference against which patient-level path disruption is measured in the accompanying analyses.

---

## What the notebook produces

Four artefacts, all keyed by the shortest-path tuple `(source, …, target)`:

| Artefact | Meaning |
|---|---|
| `path_counts[path]` | Number of healthy participants whose weighted shortest path from `source` to `target` is exactly this tuple. |
| `all_path_inclusion_rate[path]` → `df_all_inclusionRate` | Fraction of healthy participants for whom `path` is the shortest path between its endpoints. Returned as a DataFrame with the dataset name (`Language` / `MD`) as the column header. |

`df_all_inclusionRate` is the primary output; it can be index-joined with a second run to place both networks side-by-side.

---

## Requirements

- Python ≥ 3.9
- `numpy`, `pandas`, `scipy`
- [`bctpy`](https://pypi.org/project/bctpy/) — Brain Connectivity Toolbox (Python port), for `distance_wei_floyd` and `retrieve_shortest_path`
- Jupyter (to run the notebook)

```bash
pip install numpy pandas scipy bctpy jupyter openpyxl
```

`openpyxl` is needed because the AAL table is read from `.xlsx`.

---

## Expected directory layout

The notebook is written to be run from a project root that contains:

```
project_root/
├── SPL_Corpus_Language_MD_final.ipynb
├── sne_measure/
│   └── aal_table.xlsx                       # 116-region AAL parcellation table
├── language_large/
│   └── LQT_result/
│       └── fMRI_Language/
│           ├── <participant_1>/
│           │   └── Parcel_Disconnection/
│           │       └── <participant_1>_AAL_percent_parcel_SDC.mat
│           ├── <participant_2>/…
│           └── …
└── MD_large/
    └── LQT_result/
        └── fMRI_MD/
            ├── <participant_1>/
            │   └── Parcel_Disconnection/
            │       └── <participant_1>_AAL_percent_parcel_SDC.mat
            └── …
```

Each `.mat` file must contain a 116×116 matrix under the key `pct_sdc_matrix`. These are produced by the [Lesion Quantification Toolkit (LQT)] pipeline applied to the fMRI-defined participant sets described below.

---

## Configuration

A single toggle at the top of Section 2 selects which dataset to build the corpus from:

```python
NETWORK = "Language"   # or "MD"
```

Everything downstream reads from the resolved `config` dict, so no other cell needs editing when switching datasets. To produce both corpora, run the notebook end-to-end twice — once with each toggle.

| Config field | Language | MD |
|---|---|---|
| `column_name` | `LanguageNetwork` | `MD` |
| `healthy_data_dir` | `language_large/LQT_result/fMRI_Language` | `MD_large/LQT_result/fMRI_MD` |
| `file_tag` | `Lang` | `MD` |

---

## Pipeline

The notebook is organised into five sections that run top-to-bottom:

1. **Imports** — `numpy`, `pandas`, `scipy.io`, `bct`.
2. **Configuration** — pick `NETWORK`.
3. **Search-information function** — `search_information_ori`, adapted from Rosvall et al. (2005). Returns an `N × N` matrix where `SI[i, j] = −log₂ P(shortest path i → j under a random walk)`.
4. **Data loading**
   - Load per-participant `.mat` connectivity matrices from `config["healthy_data_dir"]` → list `all_data`.
   - Load the AAL parcellation table → `aal_df`, and construct `aal_nodes = [0, …, 115]`.
5. **Build the corpus**
   - For every participant and every ordered pair `(source, target)` of AAL nodes, compute the weighted shortest path via Floyd–Warshall (`transform='inv'`, so weight-to-distance is `1/w`), retrieve the path as a tuple of AAL indices, and update `path_counts`, `path_prob`, and `passnode_counts`.
   -  Convert `path_counts` into `all_path_inclusion_rate` and wrap as `df_all_inclusionRate`.

---

## Data sources

The two healthy fMRI datasets used to define participant samples come from:

- **Language network** — Lipkin, B., Tuckute, G., Affourtit, J., et al. (2022). *Probabilistic atlas for the language network based on precision fMRI data from >800 individuals.* *Scientific Data*, 9, 529.
- **MD (Multiple-Demand) network** — Diachek, E., Blank, I., Siegelman, M., Affourtit, J., & Fedorenko, E. (2020). *The domain-general multiple demand (MD) network does not support core aspects of language comprehension: A large-scale fMRI investigation.* *Journal of Neuroscience*, 40 (23), 4536–4550.

The AAL parcellation follows:

- Tzourio-Mazoyer, N., Landeau, B., Papathanassiou, D., et al. (2002). *Automated anatomical labeling of activations in SPM using a macroscopic anatomical parcellation of the MNI MRI single-subject brain.* *NeuroImage*, 15, 273–289.

The search-information measure follows:

- Rosvall, M., Grönlund, A., Minnhagen, P., & Sneppen, K. (2005). *Searchability of networks.* *Physical Review E*, 72, 046117.

---

## Notes on scale and runtime

- With all 116 AAL nodes, the corpus iterates 116 × 115 = **13,340 ordered source→target pairs** per participant.
- `search_information_ori` is the dominant cost inside the per-participant loop; it computes shortest-path probabilities across the full `116 × 116` grid.
- Rank-deficient connectivity matrices receive a tiny diagonal ridge (`1e-7 · I`) so the transition matrix solve inside `search_information_ori` stays well-defined.
- Progress is printed once per participant (`participant k/N: M unique paths so far`).

---

## Citation

If you use this code, please cite the manuscript above once it is published. Until then, cite as:

> [Duh et al.] (under review). *Severe Aphasia After Small Bilateral Lesions: A Case Study of Disrupted of High-Probability Structural Pathways.*

---

