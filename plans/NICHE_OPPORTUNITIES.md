# Niche opportunities

Observations about gaps in the field that could matter for a future company. Each entry has a date and a confidence level. Re-check before acting on any of them: the field moves monthly.

## 1. Small, general pick-and-place policy
*Noted 2026-11-26 · confidence: medium (from general knowledge, not a systematic search)*

Nobody seems to have published a distilled, general pick-and-place model small enough to work reliably on any setup. Today you choose between:
- **small and specific:** ACT, about 50M parameters, which works only for the camera and calibration it was trained on;
- **large and general:** VLAs such as SmolVLA (about 450M) or pi0 (billions), which still need fine-tuning.

**Why it's hard:** generalising across cameras, lighting and arms seems to need scale and diverse data. Distillation usually loses exactly that robustness.

**Possible angles:**
- Distil a VLA into a small student model for one arm type (SO-101) but any camera placement.
- Standardised camera mounts plus calibration so that "any setup" shrinks to a few known setups.

**How to check:** search the Hub and arXiv for distilled VLAs and small generalist policies. Then test a candidate on our own SO-101.

## 2. Robot skills learned from annotated human video
*Noted 2026-11-09 · confidence: low–medium (the founding hunch of this project)*

Video transformers can now annotate human hand motion at scale. If that motion can be mapped onto arm actions, demonstrations become cheap: film people, not teleoperators.

**Our edge:** the chess robot gives a concrete benchmark and a dataset of human and robot moves on the same board.

**How to check:** a literature review on human-video-to-robot transfer. Then a small experiment in roadmap Phase 5.
