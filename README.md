# Hypergraph Connectomics of Epileptic Human Cortex (Google H01)

Computational neuropathology pipeline applied to Google's H01 dataset (1.4 PB EM
reconstruction of human temporal cortex from a patient with drug-resistant epilepsy).
Standard pairwise graphs miss the "multicast" broadcast structure of cortical
microcircuits — this project builds a Hyper-Adjacency Tensor representation instead,
and shows it outperforms pairwise graphs at predicting seizure propagation.

**Core claim:** Hypergraph-defined routing hubs (Layer 4) are disproportionately
important for network-wide communication and seizure propagation, in a way standard
graph topology alone cannot explain (validated across 17/17 conditions: 5 patches,
6 hyperedge radii, 6 graph baselines).

## Repo structure

Folders are ordered to match the actual analysis pipeline, from tutorial/setup through
final validation:

| Folder | Contents |
|---|---|
| `01_Tutorial_and_Data_Loading` | Start here. Loads H01 soma table, filters cell types, builds your first hyperedges. |
| `02_Single_Patch_Fitting` | Spectral analysis + diffusion simulation on a single 2,618-neuron patch. |
| `03_Hypergraph_Model_and_Running_Example` | **Core model.** Hyper-Adjacency Tensor, tensor PDE, SVD latent modes, reaction-diffusion PDE. Run this to reproduce the main model end-to-end. |
| `04_Astrocyte_Expansion` | Adds astrocytes as a third cell population (modulator + hub hyperedges). |
| `05_Oligo_Immune_Expansion` | Adds oligodendrocytes + microglia/OPCs -> full 6-population tensor. |
| `06_Incremental_Spatial_Expansion` | Scales from one patch to the full ~13,491-neuron intersection. |
| `07_Structural_Anomaly_Detection` | Axon whorls, torpedoes, synaptic clustering, and other structural anomaly signatures. |
| `08_Seizure_Propagation_PDE` | FitzHugh-Nagumo seizure model parameterized from the anomaly atlas. |
| `09_Hub_Ablation_Validation` | Causal test: does removing hypergraph hubs collapse the network? |
| `10_Graph_vs_Hypergraph_Comparison` | Head-to-head benchmark vs. standard pairwise graphs. |
| `11_Robustness_Testing` | Generalization across patches, radii, and graph baselines. |
| `12_Biological_Characterization_and_Atlas` | Biological hub profiling + the Hyperedge Glass Atlas visualization. |
| `13_Axonal_Morphology_Skeletons` | Real C3 skeleton reconstructions cross-validated against hypergraph centrality. |
| `docs/` | Full research log (from project notes) and reference list. |

## Key results

- Hypergraph features predict seizure recruitment timing **~105% more accurately**
  than the best pairwise-graph predictor (Hyperdegree r=0.760 vs. PageRank r=0.514).
- Hypergraph territory clarity is **~193% clearer** than graph territory clarity.
- Ablating the top 5% of hypergraph hubs drops seizure recruitment from 100% -> 85.2%;
  equivalent graph-hub ablation barely moves it (99.6%).
- Layer 4 fires first in every seizure scenario tested, regardless of injection site —
  a network-structural invariant, not an artifact of where the seizure starts.

## Running this repo

**Important:** this analysis was originally written as one continuous Colab session, so
later notebooks depend on kernel state (variables, tensors, hyperedges) built by earlier
ones. Splitting it into 13 folders is much easier to read and cite from a paper, but it
means you have two options for actually running it:

1. **Single session (simplest):** open notebooks `01` through `13` in that numeric order
   inside one Colab runtime and just run all cells top to bottom, in order, without
   restarting the runtime in between.
2. **Independent sessions (checkpoints):** every notebook after `01` starts with a cell
   that restores a `dill` checkpoint saved by the previous notebook (`checkpoints/0N_state.pkl`),
   and ends with a cell that saves its own checkpoint for the next one. This lets you run
   notebooks in separate sessions, over separate days, etc. `pip install dill` first.
   Checkpoints aren't included in this zip (they can get large) — they're generated the
   first time you run the notebooks end-to-end.

**One real bug fix, not just reorganization:** in the original notebook, a block of 6
cells (hub-vs-low-degree diffusion simulation) was positioned *before* the tensor cells
it actually depends on — you can tell because they were literally labeled `# Recovery cell`
and `# Fix: this cell predates our tensor work` in the original source. That happened
because Colab lets you re-run/re-paste cells out of file order, so file position drifted
from execution order after a kernel restart. Those 6 cells have been moved to their correct
position in `03_Hypergraph_Model_and_Running_Example` (right after the tensor Laplacian is
built) — no code was changed, only cell position. The astrocyte folder (`04`) has a similar
set of `# RECOVERY` cells, but those genuinely do need `03`'s tensor outputs *and* `04`'s
own earlier cells, so they're left in place with the checkpoint mechanism handling the
cross-folder part.

## Requirements

```
pip install cloud-volume hypernetx dask networkx meshio numpy scipy matplotlib pandas dill
```

## Citation

See `docs/references.md` for the full reference list used in the associated paper.
