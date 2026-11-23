# Inference notebooks

[Project overview](../README.md) · [中文介绍](../README.zh-CN.md) · [Configuration notes](../params.md)

This directory contains two Kaggle inference notebooks and archived runtime assets. The notebooks load **local model weights**, preprocess Akkadian transliterations, generate English text, and write `submission.csv`.

## Public Hugging Face checkpoint

The owner has published **[TaiE64/OA2Eng-ByT5XL-V4-14000](https://huggingface.co/TaiE64/OA2Eng-ByT5XL-V4-14000)**. It is accessible without login and includes `model.safetensors`, `config.json`, `generation_config.json`, and `tokenizer_config.json`.

For an exact model snapshot:

```bash
hf download TaiE64/OA2Eng-ByT5XL-V4-14000 \
  --revision f328700ddf5095623bb5903105e41b4b8d36da3f \
  --local-dir ./models/OA2Eng-ByT5XL-V4-14000
```

Set `UltraConfig.model_path` to that folder. For an offline Kaggle run, download beforehand and attach the model files as a dataset. Keep the notebook's generation limits and preprocessing; the checkpoint's saved generation configuration alone is not the competition recipe.

The final leaderboard records **JoJoImpact, rank 18, score 39.1357**. Separately, the highest Private score among the 107 retrieved submissions is **39.4779**, from [infer-v4-14000, script version 302448234](https://www.kaggle.com/code/yuanchuan64/infer-v4-14000?scriptVersionId=302448234), with Public score **36.9959**. The Kaggle API exposes this notebook as version 1. Its dataset name is `yuanchuan64/akkadian-v4-14000`; the HF artifact is owner-provided with matching version naming, without a binary-identity check. The notebook is currently private, while the HF model and [archived output preview](../media/evidence/v4-14000-preview.csv) are publicly available.

## Choose a notebook

| Notebook | Model dataset recorded in metadata | Default batch size |
|---|---|---:|
| [fft5400](fft5400/akkadian-fft5400.ipynb) | `yuanchuan64/byt5xl-fft-5400` | 2 |
| [Backup](BackUp_Submission/akkadian-baseline-ref.ipynb) | `yuanchuan64/akkadian-baseline` | 1 |

Both metadata files also list `yuanchuan64/akkadian-namematch`. These are recorded dataset identifiers, not a guarantee that the datasets are publicly accessible. The current notebook source does not implement a separate name-matching stage merely because that dataset is listed.

The repository does not currently identify either notebook as the exact final ranked submission. Use `fft5400` as an entry point for reading the ByT5-XL checkpoint inference path; the backup has a different preprocessing path and model target.

## Run on Kaggle

1. Create a Kaggle notebook and upload one of the `.ipynb` files above.
2. Attach the [Deep Past Challenge](https://www.kaggle.com/competitions/deep-past-initiative-machine-translation) data and the matching model dataset you can access. The model folder must contain the weights, `config.json`, and tokenizer assets.
3. Select a GPU runtime. The checked-in metadata enables GPU and disables internet; the model and tokenizer are loaded with `local_files_only=True`.
4. Review the `UltraConfig` class near the top of the notebook and set the actual input paths. In particular, check `model_path`, `test_data_path`, and `output_dir`.
5. Run in a fresh session and inspect the resolved model directory in the log. The fallback search can select the first discovered `config.json`; an explicit path avoids selecting an unintended attached model.
6. Inspect the generated translations and output files. For a competition submission, check that every test ID occurs exactly once and each translation is non-empty.

The two `kernel-metadata.json` files preserve the author's original notebook IDs and `is_private: true`. If using Kaggle CLI to publish your own copy, change the IDs to your account and choose the visibility intentionally.

## Configuration to review

| Field | Current checked-in default | Meaning |
|---|---|---|
| `max_length` | `512` | Tokenized source truncation limit |
| `max_new_tokens` | `512` | Maximum generated token count |
| `num_beams` | `4` | Beam-search width |
| `length_penalty` | `1.3` | Generation length penalty |
| `use_bucket_batching` | `True` | Group inputs by length |
| `use_mbr` | `False` | Optional candidate-consensus selection |
| `use_better_transformer` | `False` | Optional Optimum integration |
| `output_dir` | `/kaggle/working/` | Output CSV, settings, and logs |

Edit `UltraConfig` to change the run. The saved `ultra_config.json` is an **output summary**, not a configuration file automatically read by the notebook.

The adaptive-beam hook is enabled, but with `num_beams=4` its short-input branch also resolves to four beams. Treat this as a configurable implementation feature, not a demonstrated speedup.

## Runtime dependencies and precision

The main imports are PyTorch, Transformers, pandas, NumPy, and tqdm. Optimum is optional and only needed for the BetterTransformer branch. Archived wheel directories record earlier environment work; installing every archived wheel is not a reproducibility recipe.

The current model-loading call requests `dtype=torch.float16`, while another helper controls BF16 autocast. Some retained comments and saved summaries say “BF16-only” or “never FP16”; those labels do not fully describe the current loading code. Check the actual runtime, GPU support, Transformers compatibility, and memory use when adapting the notebook.

Both notebooks retain older execution outputs. In particular, the `fft5400` notebook's saved output mentions the baseline model despite its current source targeting the `byt5xl-fft-5400` dataset. A fresh execution is needed to establish the behavior of the current source.

## Outputs

| File | Purpose |
|---|---|
| `submission.csv` | `id,translation` prediction pairs |
| `ultra_config.json` | A summary of generation and optimization settings |
| `inference_ultra.log` | Runtime logging |
| `checkpoint_*.csv` | Periodic partial predictions when the checkpoint condition is met |

The committed CSV in `BackUp_Submission/` is an archived prediction artifact. It is not a substitute for generating predictions against your current test set or evidence of the complete hidden-test submission.

## Optional MBR experiment

With `use_mbr=True`, the notebook can generate a candidate pool using beams and optional sampling, then choose a translation using an inline chrF++-style sentence-similarity function. Candidate counts and sampling settings are exposed as `mbr_*` fields.

This helper is distinct from the official corpus-level BLEU/chrF++ competition evaluation. Both notebooks and the saved backup configuration have MBR disabled by default. Any effect on quality or runtime needs a separate evaluation with fixed data and model weights.

## Data and attribution notes

The archived `merged_all.csv` has 69,211 rows; this is not a statement that every row was used in the final training run. Extraction outputs may overlap. Some data scripts retain original Windows paths and write CSVs in place, and many large PDFs/CSVs are LFS pointers in a fresh checkout.

Third-party datasets, publications, models, and dependency assets retain their own terms and attribution. No repository-wide code license is currently specified.
