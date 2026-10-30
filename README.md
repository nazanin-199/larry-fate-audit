# Evaluating Cell Fate Prediction Without Clonal Supervision

**Can trajectory-learning models forecast fate decisions without clonal supervision?**
A leakage-controlled evaluation on real lineage-tracing data, built as a teaching notebook that runs on a free Kaggle GPU.

> **Scope, read first.** This project does **not** run the released CellTempo model. It uses simplified stand-ins (graph-based trajectory arms and small CellTempo-*style* models trained on PCA features) and scores them against independent clonal outcomes. Nothing here is a claim about the published model, velocity-based supervision, or long-horizon forecasting.

## The idea

Methods like [CellTempo](https://doi.org/10.64898/2026.02.08.704720) learn to forecast cell fate from trajectories that were *inferred* from static snapshots, so there is no direct ground truth for the forecast. The LARRY dataset has one: heritable barcodes mark sister cells, so the later fates of an early cell's clone-mates show where its lineage went. This notebook uses that as an independent check, and treats **building the evaluation correctly** as the main goal: clone-disjoint splits, honest floors, negative controls, and diagnosis of every failure.

## What is in the repo

| File | Purpose |
|---|---|
| `larry_fate_audit.ipynb` | The whole project: concepts, all code in run order, results, interpretations, exercises, a claims ledger |
| `README.md` | This file |

Each step in the notebook follows the same pattern: *concept note → code → interpretation of the output*. Tags mark status: `[from run]` (output stored from a real run), `[NEW — not yet run]`, and `[post-hoc]` (chosen after seeing results, so exploratory).

## Main findings (details in the notebook)

Primary task: predict the late fate of 534 day-2 clones (49,116 cells, mouse haematopoiesis, three timepoints). Scores are mean log-likelihoods in nats; gains are over a constant "floor".

- **Supervised reference:** a logistic regression trained with clonal labels gains **+0.462 nats** [95% CI +0.392, +0.531] on held-out clones.
- **Label-free arms fail, for diagnosable reasons.** One-step kNN propagation is starved of labelled neighbours; an absorbing-chain arm ranks cells well (clone-level AUROC 0.71–0.97) but is strongly biased toward one fate. Calibrating it with training-clone labels recovers +0.207, but that is no longer label-free.
- **Training on the chain's labels** transfers ordering for some fates (Baso, Mast, Meg) but not Monocyte vs Neutrophil.
- **Minimal CellTempo-style models** trained on label-free walks score below a constant; the shortfall is mostly a biased average prediction. The ablations (discrete vs continuous, Transformer vs GRU vs MLP) are **inconclusive**, and one control arm turned out to be an invalid design.
- **Methodological lessons:** a floor discrepancy (−1.245 vs −1.328) was an *estimand* mismatch (cell- vs clone-weighted averaging), not a bug; and leakage *exposure* (up to 95% of test cells with a sister in training) did not translate into visible inflation for a low-capacity model.

## What it does not show

The released CellTempo's behaviour on LARRY; velocity-derived supervision; negative-binomial tokenization on raw counts; any biological mechanism. Clone-mates are an imperfect proxy for fate, 534 clones limits statistical power, and many hypotheses in the notebook are explicitly marked **untested**. The final section is a claims ledger separating what was evaluated from what was not.

## Run it

1. Open the notebook on Kaggle with **Internet on**; a GPU is needed only for Part 8.
2. Run top to bottom (about 30–60 minutes on a T4). Paths assume `/kaggle/working`; other platforms need edits.
3. Data are **downloaded at runtime**, not shipped here (see below).

Long computations are checkpointed per fold with a settings fingerprint, so an interrupted session can resume. Appendix B lists run-order hazards (for example, `N_ITER` is changed globally in two cells).

## Reproducibility

Fixed seeds, recorded library versions, a pre-specified protocol frozen before modelling, shuffled-label and random-supervision controls for every arm, and results reproduced across two independent sessions.

## Data and licensing

- **LARRY**: Weinreb et al., *Science* 2020 (GEO GSE140802). The CoSpar-preprocessed subset is downloaded at runtime and not redistributed.
- **CellTempo preprint** (Luo, Gao et al., bioRxiv 2026) is CC BY-NC-ND. The notebook paraphrases and cites it and reproduces none of its text or figures.
- **CoSpar** package: <https://github.com/ShouWenWang-Lab/cospar>.
- Code and commentary: see `LICENSE`.

## Citing and credit

An independent educational audit, not affiliated with the CellTempo authors. Please cite the original papers above for the data and the method being audited.

**Author:** *your name* · feedback and corrections welcome via issues.
