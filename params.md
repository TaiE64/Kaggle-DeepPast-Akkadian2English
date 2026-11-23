# Training notes and inference configuration

[Project overview](README.md) · [Notebook setup](submission/README.md)

This file separates the short original training notes from settings that can be read directly from the checked-in inference notebooks. The full training program and final checkpoint are not included in this repository.

## Original training notes

| Item | Recorded value | Interpretation |
|---|---|---|
| Continued pretraining (CPT) learning rate | `0.001` | Recorded in the original note |
| CPT schedule | Warmup + cosine decay | Warmup duration was not recorded |
| CPT duration | `10ep` | 10 epochs in the note |
| Supervised fine-tuning (SFT) learning rate | `0.00l` | Original spelling is ambiguous; no numeric value is inferred |
| SFT schedule | `warmuo+cosine decay` | Appears to mean warmup + cosine decay |
| Label smoothing | `0.1` | Recorded in the original note |
| NEFTune | `0.5?` | Tentative value; use and strength are not confirmed |
| Optimizer | AdamW | Recorded in the original note |

Original text, preserved for reference:

```text
CPT lr 0.001 warmup+cosine decay 10ep
SFT lr 0.00l warmuo+cosine decay
label smoothing 0.1
neftune 0.5?
optim=AdamW
```

These notes are not a complete training recipe. They do not establish the final data mixture, split, seed, batch size, checkpoint selection, or exact model used for the ranked submission.

## Current notebook defaults

| Setting | `fft5400` | `BackUp_Submission` |
|---|---|---|
| Model path | `/kaggle/input/byt5xl-fft-5400` | `/kaggle/input/akkadian-baseline` |
| Batch size | 2 | 1 |
| Source token limit | 512 | 512 |
| Maximum new tokens | 512 | 512 |
| Beams | 4 | 4 |
| Length penalty | 1.3 | 1.3 |
| Early stopping | Enabled | Enabled |
| No-repeat n-gram size | 0 | 0 |
| Bucket batching | Enabled | Enabled |
| Adaptive-beam hook | Enabled; resolves to 4 at the default width | Same |
| MBR | Disabled | Disabled |
| BetterTransformer | Disabled | Disabled |

Sources: [`fft5400` notebook](submission/fft5400/akkadian-fft5400.ipynb), [backup notebook](submission/BackUp_Submission/akkadian-baseline-ref.ipynb).

The source code takes precedence over saved notebook logs and output JSON when describing current defaults. See the [runtime notes](submission/README.md#runtime-dependencies-and-precision) for the model-loading dtype and BF16-autocast distinction.
