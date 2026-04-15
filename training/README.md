# Training Pipeline

This folder contains everything needed to train the ~15M parameter SLM on the TinyStories dataset.

## Contents (coming in Phase 1)

- `train_slm.ipynb` — Google Colab notebook for the full training pipeline
- `tokenize_data.py` — BPE tokenization and .bin file creation
- `model.py` — Transformer architecture definition
- `train.py` — Training loop with gradient accumulation + mixed precision
- `generate.py` — Inference/story generation script

## Quick Start

1. Open `train_slm.ipynb` in Google Colab
2. Set runtime to GPU (A100 recommended, T4 works but takes 6-7 hours)
3. Run all cells
4. Download the trained checkpoint (`best_model_params.pt`)

See the [main README](../README.md) for full project context.
