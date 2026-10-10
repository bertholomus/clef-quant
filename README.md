# clef-quant

Original low-bit quantizations of Cloudflare's **Clef** decision model (27B, Apache-2.0), built so it
runs on a single 24 GB RTX 4090, plus the serving and evaluation harness that proves the quant still
gives honest, calibrated probabilities.

Status: **development**. Nothing is published yet.

## Why

Clef is a decision model, not a chat model. A Qwen3.8-27B backbone reads the input once (prefill only),
then a small joint schema head turns the last hidden states into probabilities for typed questions
(`noul`, `choice`, `score`). At bf16 the 27B needs about 54 GB. The target here is about 16 GB:
a 4-bit backbone with the vision encoder and the joint head kept at bf16.

The risk is calibration. The head was trained on bf16 hidden states. Quantizing the backbone shifts
what the head sees, so accuracy can hold while the probabilities drift. Calibration is a first-class
gate in this repo, not an afterthought.

## Layout

| Path | Contents |
| --- | --- |
| `docs/DESIGN.md` | Quant design, open choices, and the reasoning behind each one |
| `docs/GATES.md` | Acceptance gates: what a quant must pass before it is served or published |
| `quant/` | Quantization recipes and scripts |
| `serve/` | Jev/System-One-compatible HTTP server for Clef and Clef-Flash |
| `eval/` | Fair evaluation harness (tune / held-out split, same wording for every candidate) |
| `results/` | Measured result tables (JSON + Markdown), one folder per run |
| `pipeline/` | Private working notes (not in git) |

## Candidates

1. **Clef 27B, 4-bit backbone** (this repo's quant) on the RTX 4090.
2. **Clef-Flash 9B, bf16** (Cloudflare's own weights, no quant) on the RTX 4090.
3. **Reference:** Clef 27B at bf16 (Cloudflare Workers AI or a larger host) as the probability oracle.
4. **Incumbent:** hosted Jev (Typesafe) as currently used by `jev-decide`.

## License

Apache-2.0. Clef weights are Apache-2.0 from Cloudflare; see `THIRD_PARTY_NOTICES.md`.
