# Acceptance gates

A quant is **served** only after every gate below passes on the held-out half. It is **published** only
after Albert says so. Write the bar down before any run; do not move it after seeing results.

## G0. Integrity
- Source weights banked at a pinned Hugging Face revision and verified (size + LFS sha256 against the HF
  tree API). Receipts in `/tank/models/receipts/`.
- Quant artifacts carry their own sha256 manifest and the exact recipe commit that produced them.

## G1. Fits the card
- Loads on one RTX 4090 (24 GB) with room for a 16k-token state plus 4 images per request.
- Report peak VRAM at load and at the largest test request.

## G2. Fidelity to the bf16 reference (the core gate)
Same requests, same wording, quant vs bf16 Clef 27B reference:
- Top-1 agreement on `choice` and `noul` (at 0.5) answers: target >= 98 %.
- Mean KL divergence of per-question distributions: report it; target set in DESIGN.md before the run.
- Calibration: expected calibration error (ECE) on labeled items must not exceed the reference's ECE by
  more than the agreed margin.
- `score` answers: mean absolute difference of the probability-weighted score.

## G3. Beats the cheaper option
- On the held-out half, the 4-bit 27B must beat Clef-Flash bf16 on the decision bar (accuracy at the
  agreed false-flag budget). If it does not, Clef-Flash is the served choice and this quant is parked.

## G4. Latency
- Measure p50 / p95 per request on the 4090, text-only and with 1–4 images.
- State which hook the latency supports: inline per-tool-call screen, turn-end, or batch.

## G5. Fair-test rules (from MiniMe's decision-model evaluation method)
- Tune / held-out split. Thresholds and wording are chosen on tune only, then frozen.
- Every candidate gets the identical state shape and question wording.
- One decision item per state.
- Configs authored after seeing results are labeled as such and discounted.
- Real session content stays local. Only crafted or synthetic corpora may go to hosted candidates.

## Still Albert's call
Serving it in `jev-decide`, Hugging Face uploads, public posts, merges to `main`, force-pushes.
