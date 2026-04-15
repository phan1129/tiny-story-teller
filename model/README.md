# Model Artifacts

This folder contains model conversion scripts and configuration. Trained model files (.pt, .mlmodel, .mlpackage) are excluded from Git via .gitignore due to size.

## Contents (coming in Phase 2)

- `convert_to_coreml.py` — PyTorch → CoreML conversion script
- `quantize.py` — INT8/INT4 quantization scripts
- `benchmark.py` — Model size, speed, and quality benchmarks
- `config.json` — Model architecture configuration

## Model Variants

| Variant | Format | Size (est.) | Use Case |
|---------|--------|-------------|----------|
| Full precision | .pt (float32) | ~60MB | Training/development |
| Half precision | .mlpackage (float16) | ~30MB | CoreML baseline |
| INT8 quantized | .mlpackage | ~15MB | Default on-device |
| INT4 quantized | .mlpackage | ~8MB | Older/low-memory devices |

See [ARCHITECTURE.md](../ARCHITECTURE.md) for quantization strategy rationale.
