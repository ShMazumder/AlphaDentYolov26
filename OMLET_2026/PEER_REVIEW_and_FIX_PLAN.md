# Brutal Peer Review + Fix Plan
**Paper:** Multi-Method Explainability for YOLO26-Based Caries Detection in Intra-oral Photos
**Date:** 2026-07-12

---

## Part 1 — Review

**Recommendation: Major Revision → leaning Reject.** Engineering is real and the writing is above average, but the empirical core is one-run margins on a validation set, the XAI module is partly degenerate, and the central qualitative figure is a placeholder.

### Major weaknesses

**M1. Several of the "7 methods" collapse into one (VERIFIED root cause).**
Row-level check of `xai_eval_results_9classes/faithfulness_all.csv`:
- **HiResCAM and LayerCAM are 100% bit-identical** (max|Δ| = 0 on every image, both deletion and insertion AUC).
- GradCAM and XGradCAM are identical on ~67% of images (partially collapsed, not fully).
- EigenCAM, GradCAM++, HiResCAM otherwise distinct.

Root cause is in `compute_act_grad`: the CAM target `score = Σ_locations sigmoid(class_logit)` is a **global spatial sum over all anchors/scales**. Backpropagating a global sum yields a gradient at the hook layer (idx 22) that is non-negative everywhere and near-uniform per channel. Then HiResCAM `relu(Σ act·grad)` and LayerCAM `relu(Σ act·relu(grad))` are identical iff grad ≥ 0 (it is → bit-identical), and GradCAM `w=mean(grad)` ≈ XGradCAM `w=Σ(grad·act)/Σact` when grad is spatially flat (→ the 67% overlap). The methods are correctly coded; the *objective* is wrong. Table IV currently reports a duplicate row (HiRes = Layer).

**M2. Headline accuracy win is inside the noise.**
Nine-class: 0.4455 vs 0.436 = +0.0095. Merged: 0.6913 vs 0.680 = +0.011. No CIs, seeds, repeats, or significance tests. Single run each. Baseline is the dataset authors' published number under a possibly different protocol (mask-IoU vs your column). Cannot support "surpasses."

**M3. No held-out test set; selection and reporting both on validation.**
"Best checkpoint by fitness" is chosen on val and metrics reported on val → optimistic bias / leakage. Every headline number is a cherry-picked val number.

**M4. XAI evaluation underpowered; qualitative figure is fake.**
n=30 images; only 4 of 9 classes present (0,1,2,7). Deletion-AUC std 0.29–0.57 dwarfs the +0.136 EigenCAM margin; EigenCAM insertion swings 0.25→0.90 across configs, so "most faithful" is a pooled artifact. `fig_xai.png` (Fig 2) is a grey placeholder — the core explainability visual does not exist.

**M5. Contributions are mostly bug-fixes.**
DDP enablement, dataset staging, `type=int`→float threshold fix, OOM retry-on-CPU are reproducibility patches, not research contributions. Remaining novelty reduces to a backbone swap (YOLOv8x→YOLO26) for +0.01.

**M6. "YOLO26x overfits" asserted, not shown.**
No per-class metrics, PR curves, or training curves to support the precision-up/recall-down narrative.

### Minor
- Section II titled **"Literate Review"** → "Literature Review" (typo).
- 1.85× on 2 GPUs is sub-linear; framed as a pure positive.
- Merged 26l run truncated at 83/100 epochs, "still improving" → comparison uses an unconverged model.
- Abstract now (correctly) states agreement does not track correctness — this undercuts the "multi-method cross-check as clinical audit tool" contribution.
- Table IV has no n, CI, or significance marker; deletion AUC 0.497 for the "faithful" method is barely below random (0.5).
- Author placeholders ("First Author Name", …) still present.

### Rebuttal questions
1. Why are HiResCAM/LayerCAM and GradCAM/XGradCAM numerically identical? Show the maps differ.
2. Where is the held-out test set? Re-report all mAP on test.
3. Provide ≥3 seeds + std, or soften "surpasses" to "matches."
4. Real Fig 2 + per-class table, or the XAI section is unsupported.

---

## Part 2 — Fix Plan (prioritized)

Legend: **[code]** notebook/script work · **[compute]** requires GPU re-runs · **[write]** text-only edit · effort in hours is rough.

### Tier 0 — Must fix or reject (blockers)

**F1. Fix the collapsed-CAM objective (M1) — DIAGNOSED.** [code] ~2h
Root cause confirmed: global-spatial-sum target in `compute_act_grad` gives near-uniform, non-negative gradients, collapsing HiRes≡Layer (bit-identical) and Grad≈XGrad. Fix = target the localized detection peak instead of the sum:

```python
maps = collect_cls_logit_maps(raw, nc)
peak_t = None
for mm in maps:
    cls = mm[:, -nc + class_idx]
    if peak_t is None or cls.max() > peak_t.max():
        peak_t = cls
score = peak_t.max().sigmoid()   # scalar at the single strongest location
score.backward()
```

- Re-run faithfulness; regenerate `faithfulness_pooled.csv`, `fig_faithfulness.png`, Table IV.
- Verify: per image, pairwise cosine similarity of the 7 raw CAMs; HiRes vs Layer must drop below 1.0.
- If any pair stays identical after the fix, state it and cut the method count honestly.

**F2. Introduce a real train/val/test split and re-report (M3).** [code][compute] ~1–2 days
- If AlphaDent ships only train/val, carve a patient-disjoint test set (never split by image — split by patient ID `pXXX` to avoid leakage; your filenames already encode patient).
- Select checkpoint on val, report all mAP on **test**. Update Tables II–III and every headline number.

**F3. Replace the placeholder Fig 2 (M4).** [code] ~2h
- Generate the real 7-method CAM grid for one representative caries instance (overlay heatmaps on the intra-oral photo). Save as `fig_xai.png`. (I can do this from your XAI outputs once F1 is fixed.)

**F4. Statistical rigor on the accuracy claims (M2).** [compute] ~1–2 days
- Run each reported configuration with ≥3 seeds; report mean ± std.
- If Δ to baseline < std, change wording throughout from "surpasses/exceeds" to "matches within noise" or "is competitive with." Abstract, Results, Discussion, Conclusion.
- Add a bootstrap or paired test over per-image AP if feasible.

### Tier 1 — Strongly expected

**F5. Per-class results table (M6, M4).** [code] ~2h
- Emit per-class Box/Mask AP@50 (you have this from Ultralytics `results.results_dict` / `metrics.box.maps`). Add as a table; it also substantiates the "x overfits: precision↑ recall↓" claim with numbers (report P/R per class for l vs x).

**F6. Finish the truncated merged-class run (minor→major).** [compute] ~4h
- Complete YOLO26l merged to 100 epochs, or drop the "surpasses 0.680" sentence.

**F7. Expand/There-characterize the XAI sample (M4).** [code][compute] ~4h
- Raise n well above 30 and ensure all 9 classes are represented, or explicitly scope the XAI claims to the 4 classes actually evaluated (state this in the paper). Add n and 95% CI to Table IV.

### Tier 2 — Cheap, do them all

**F8. Text fixes.** [write] ~1h
- "Literate Review" → "Literature Review".
- Fill real author names/affiliations.
- Reframe 1.85× as "1.85× (0.93 scaling efficiency)" so it reads honestly.
- Reconcile the audit-tool contribution with the agreement-≠-correctness finding: recast agreement as a *stability* diagnostic, not a reliability signal (partly done — make sure the contribution bullet and any "audit tool" phrasing match).
- Add n, CI, and a significance marker to Table IV; note deletion AUC≈0.5 caveat.

**F9. Honest limitations pass.** [write] ~0.5h
- Single dataset, patient-disjoint test now added, XAI on N images / K classes, no reader study, sub-linear scaling. Keep it explicit.

### Tier 3 — Optional, strengthens novelty (addresses M5)

**F10. Add a genuine research contribution beyond the backbone swap.** [code][compute]
- Options: (a) a faithfulness-guided CAM selection method; (b) ablation of the C3k2 hook layer (idx 22) vs alternatives on faithfulness; (c) calibration analysis linking faithfulness to clinician-relevant errors. Pick one; it converts "engineering paper" into "method paper."

---

### CRITICAL (found during Kaggle re-runs) — 4-class results are contaminated

**C1. 4-class notebook reuses the 9-class model (invalidates the merged-class run).**
Both notebooks build the identical run path `runs/segment/yolo_seg_{model}_proj_{sz}_b{b}_e{EP}/` with no `4class` suffix. The correct 4-class weights live in `runs_4classes/segment/…` (trained on `yolo_seg_train_4class.yaml`), but the 4-class notebook looks only in `runs/segment/`, finds the committed 9-class `best.pt`, prints "weights exist" and skips training. It then validates the **9-class** model against 4-class-merged labels. Evidence: val prints the 9-class name "Caries 1 class"; caries mAP collapses to ~0.10 (9-class model emits caries subclasses 3–8, merged GT is all class 3 → class-index mismatch). This is why the Kaggle 4-class numbers (0.610/0.630) differ from paper Table III (0.6729/0.6772) — different models, same folder name.

**C2. Destructive in-place label conversion.** `convert_labels_to_4class` overwrites the GT `.txt` files. Running 4-class then 9-class in the same working dir silently corrupts the 9-class eval too (order-dependent).

Fixes:
- Distinct project name for 4-class runs (e.g. `..._4class_proj_...`) or read/write under `runs_4classes/segment/`.
- Write merged labels to a separate `labels_4class/` dir + YAML; never overwrite the 9-class labels.
- After loading any checkpoint: `assert model.model.nc == EXPECTED_NC` so a wrong-nc weight fails loudly instead of mis-scoring.
- Note: the XAI `git clone` on Kaggle pulls the unpatched repo — the CAM fix (F1) must be pushed to GitHub or pasted into the cell to take effect.

### Suggested order of attack
1. **F1** (duplicate-CAM) — cheapest, highest credibility risk. Do first.
2. **F3** real Fig 2, **F8** text fixes — fast wins.
3. **F2** test split + **F4** seeds — the expensive but decisive block.
4. **F5–F7**, then **F9**.
5. **F10** if aiming above borderline.

**Realistic effort to reach "borderline accept":** F1+F2+F3+F4+F5+F8 ≈ 3–4 focused days (most of it GPU time for F2/F4).
