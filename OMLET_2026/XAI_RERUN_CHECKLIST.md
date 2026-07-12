# XAI Re-run & Manuscript Re-integration Checklist
Goal: regenerate the XAI faithfulness/agreement results with the **fixed** (localized-peak) CAM code, produce the real Fig 2, and fold the corrected numbers back into the OMLET_2026 paper.

Why: the current Table IV / Fig 4 / Fig 5 were produced by a **broken** CAM target. The v5 run exposed two faults now fixed:
- The train-mode target indexed `mm[:, -nc+class_idx]`, which counts from the tail and lands in the **32 mask-coefficient channels, not the class logits** → smooth non-negative gradient → HiRes≡Layer, Grad≡XGrad collapse.
- The chosen peak could sit on a detection scale that bypasses the hooked layer 22 → backward hook never fired → `KeyError('grad')` skips (all images for 26m_640/26x_960).

Fix (applied in the repo): `compute_act_grad` / `YOLOExplainer._act_grad` now do **one eval-mode differentiable forward** and backprop the **top detection's class confidence** (via `pred_class_and_score` / `_top_class_score`) — correct class channel, downstream of the whole neck, so gradient always reaches layer 22.

---

## 0. Do I need to re-run everything from scratch? — No.
All the bugs were in **evaluation / XAI plumbing**, not in training. The trained weights are valid.

**Sound already — DO NOT retrain:**
- All checkpoints (26m/l/x @ 640/960; 9-class in `runs/segment/`, 4-class in `runs_4classes/`).
- 9-class results (Table II) — reproduced exactly on a fresh run (0.466 Box / 0.448 Mask).

**Must re-run, but inference-only (minutes, no training):**
- XAI faithfulness/agreement with the fixed CAM code → Table IV, Figs 4/5, real Fig 2 (§B–E below).
- 4-class `model.val()` against the correct `runs_4classes/` weights + fixed notebook → clean Table III.
- Per-class caries numbers — already in the existing val output; just parse, zero compute.

**Optional rigor — the ONLY things that require actual (re)training:**
- ≥3 seeds per config for significance on the mAP claims (expensive).
- A patient-disjoint train/val/**test** re-split (retrain). Splitting the existing val into val/test avoids retraining but is small.

Recommendation: the inference-only re-runs are **mandatory** (they make the XAI/4-class results correct). The seeds/test-split are a **venue-strength decision** — for an OMLET workshop, shipping the cheap fixes + honest single-run caveats is defensible; add seeds only if a stronger claim is required.

---

## A. Environment (GPU box: Kaggle/Colab/local CUDA)
- [ ] Pull the latest repo (the CAM fix must be present): `git pull` — confirm `alphadent-xai-eval.ipynb` contains `def _act_grad` / localized-peak `score = peak_t.max()` (NOT a `.sum()` target). If you run on Kaggle via `git clone`, make sure the fix is **pushed to GitHub first**.
- [ ] GPU with ≥14 GB (T4 is fine). Set once at top: `import os; os.environ['PYTORCH_CUDA_ALLOC_CONF']='expandable_segments:True'`.
- [ ] `pip install -q -r requirements.txt` + `matplotlib opencv-python scipy`.
- [ ] Dataset present at `./AlphaDent/images/valid` + `./AlphaDent/labels/valid` (Kaggle input or Zenodo fallback — the notebook handles this).
- [ ] 9-class weights present for all five configs under `runs/segment/…/train/weights/best.pt` (26m@640/960, 26l@640/960, 26x@960).

## B. Run the faithfulness/agreement eval
- [ ] Open `alphadent-xai-eval.ipynb`, set `N_IMAGES` (default 30). **Recommended: raise to cover all 9 classes** — current run only hit classes {0,1,2,7}. Bump to e.g. 60–80 and/or curate images so every pathology class appears.
- [ ] Run All. Confirm each config prints faithfulness + agreement without the old `tuple indices` / OOM errors.
- [ ] Outputs written to `xai_eval_results_9classes/`:
  - [ ] `faithfulness_all.csv`, `faithfulness_pooled.csv`
  - [ ] `agreement_all.csv`
  - [ ] `fig_faithfulness.png`, `fig_agreement.png`

## C. Verify the CAM fix actually took (critical gate)
- [ ] **No `KeyError('grad')` skips** in the run log — every image should produce rows (the eval-mode target guarantees the layer-22 hook fires).
- [ ] Run the added **cosine-similarity cell**. Requirement: `HiResCAM vs LayerCAM` cosine **< 1.000** (was exactly 1.000 before). If still 1.000, the old code is being used — stop and recheck the pull.
- [ ] In `faithfulness_pooled.csv`, `GradCAM`, `GradCAM++`, `XGradCAM` should **no longer be bit-identical**, and `HiRes-CAM`/`Layer-CAM` should differ.
- [ ] In `faithfulness_pooled.csv`, confirm HiRes-CAM and Layer-CAM rows are **no longer identical**.
- [ ] Sanity-check ranking still makes sense (EigenCAM strong on insertion; report whatever the corrected numbers show — do not assume the old ordering holds).

## D. Produce the real Fig 2 (7-CAM grid)
- [ ] Run the added Fig-2 cell → writes `fig_xai.png` (input + 7 overlays) from a confident caries detection.
- [ ] Copy it into the paper: `cp fig_xai.png OMLET_2026/fig_xai.png` (replaces the grey placeholder).

## E. Re-integrate into the manuscript (`OMLET_2026/conference_yolo26_dental.tex`)
- [ ] **Table IV** (`\label{tab:faith}`): replace all 7 rows with the new mean±std deletion/insertion AUC + Ins.−Del. from the refreshed `faithfulness_pooled.csv`. Rows should now be distinct.
- [ ] **Quantitative XAI Faithfulness** paragraph (~line 284): update the quoted numbers (best insertion, lowest deletion, the +margin, "Grad-CAM++ least stable", etc.) to the new values.
- [ ] **Delete the artifact caveat** — the clause "*under the summed-score attribution target … HiRes-CAM and Layer-CAM coincide, an artifact that a localized-score variant in our released code removes*" is no longer needed once the table is regenerated with the fixed code.
- [ ] **Inter-Method Agreement** paragraph: update the 0.54 overall / 0.73±0.04 best-config numbers and the correct-vs-incorrect figures if they change with the larger sample; keep the honest "agreement ≠ correctness" finding if it still holds.
- [ ] Update the **abstract** sentence naming the most-faithful method if the ranking changed.
- [ ] Copy refreshed figures: `cp xai_eval_results_9classes/fig_faithfulness.png xai_eval_results_9classes/fig_agreement.png OMLET_2026/`.
- [ ] Recompile (needs `algorithm`/`algorithmic` packages) and verify no errors, no undefined refs, figures render.

## F. Remaining manuscript gaps (separate from the XAI re-run)
- [ ] **Per-class table (caries honesty):** add Box/Mask AP@50 + P/R per class from the clean 9-class val (Abrasion/Filling/Crown high; caries subclasses 0.005–0.50). State plainly that caries — the titular task — is the weak point.
- [ ] **Statistical rigor:** introduce a patient-disjoint **test** split (split on `pXXX` IDs, not images); ≥3 seeds with mean±std; soften "surpasses" → "matches within noise" wherever Δ < std.
- [ ] **Table III provenance:** confirm the 4-class numbers (0.6729/0.6772) came from clean `runs_4classes/` weights, not a contaminated run.
- [ ] **Authors:** fill the 13 placeholder strings (names, affiliations, emails).
- [ ] **Page limit:** current PDF is 7 pages — confirm the OMLET 2026 limit; trim if needed.

---

### Definition of done (XAI portion)
Table IV has 7 distinct rows from the fixed code; Fig 2 is a real 7-CAM grid; Figs 4/5 regenerated; the artifact caveat removed; paper recompiles clean.
