# Tiny Story Teller

An offline-first iOS app powered by a ~15M parameter small language model (SLM) trained from scratch on the TinyStories dataset. Generates children's stories for 3-4 year olds — fully on-device, zero API costs, complete privacy.

## Why This Project

Large language models are powerful but expensive to run and require internet connectivity. This project explores the opposite end of the spectrum: **how small can a language model be and still produce coherent English?**

Inspired by the [TinyStories paper](https://arxiv.org/abs/2305.07759) (Eldan & Li, 2023), which showed that small language models trained on a carefully curated dataset of children's stories can learn grammar, vocabulary, and basic reasoning — despite having 10,000x fewer parameters than GPT-3.

## What This Project Demonstrates

- **ML from scratch** — Training a transformer-based language model from the ground up (not fine-tuning a pre-trained model)
- **Edge AI deployment** — Converting and quantizing the model for on-device inference via CoreML
- **Resource optimization** — Lazy model loading, memory pressure handling, battery-aware inference
- **Offline-first architecture** — No API calls, no cloud dependency, complete user privacy

## Architecture Overview

┌─────────────────────────────────────────────────┐
│ TRAINING PIPELINE │
│ (Google Colab + GPU) │
│ │
│ TinyStories Dataset (2.1M stories) │
│ ↓ │
│ BPE Tokenization (GPT-2 tokenizer) │
│ ↓ │
│ Transformer Model (~15M params) │
│ - 6 transformer blocks │
│ - 6 attention heads per block │
│ - 384 embedding dimensions │
│ - 128 context window │
│ ↓ │
│ Trained checkpoint (.pt). │
└──────────────────────┬──────────────────────────┘
│
↓
┌─────────────────────────────────────────────────┐
│ MODEL CONVERSION │
│ │
│ PyTorch (.pt) → CoreML (.mlpackage) │
│ + INT8/INT4 quantization │
│ + Optimization for Apple Neural Engine │
└──────────────────────┬──────────────────────────┘
│
↓
┌─────────────────────────────────────────────────┐
│ iOS APP (SwiftUI) │
│ │
│ ┌─────────────┐ ┌──────────────────────┐ │
│ │ Story UI │ │ CoreML Inference │ │
│ │ (SwiftUI) │←→│ Engine │ │
│ └─────────────┘ └──────────────────────┘ │
│ │
│ Edge AI Features: │
│ - Lazy model loading │
│ - Memory pressure monitoring │
│ - Battery-aware inference throttling │
│ - Encrypted local storage │
│ - Sliding context window │
└─────────────────────────────────────────────────┘

## Tech Stack

| Layer             | Technology                        | Why                                                                |
| ----------------- | --------------------------------- | ------------------------------------------------------------------ |
| Training          | PyTorch + Google Colab (A100 GPU) | Industry-standard ML framework, free GPU access                    |
| Dataset           | TinyStories (HuggingFace)         | 2.1M curated children's stories, proven for SLM research           |
| Tokenizer         | tiktoken (GPT-2 BPE)              | Subword tokenization, balances vocabulary size vs. sequence length |
| Model Conversion  | coremltools                       | Official Apple tool for PyTorch → CoreML                           |
| iOS App           | SwiftUI + CoreML                  | Native Apple frameworks, Neural Engine access                      |
| On-device Storage | SwiftData + CryptoKit             | Encrypted local persistence                                        |

## Project Phases

- [x] **Phase -1:** Dev environment setup
- [x] **Phase 0:** Project architecture & planning
- [ ] **Phase 1:** Train the SLM in Google Colab
- [ ] **Phase 2:** Model export & quantization
- [ ] **Phase 3:** SwiftUI app + on-device inference
- [ ] **Phase 4:** Edge AI optimizations
- [ ] **Phase 5:** Polish & portfolio packaging

## Model Details

| Parameter           | Value                                | Why                                                       |
| ------------------- | ------------------------------------ | --------------------------------------------------------- |
| Parameters          | ~15M                                 | Small enough for mobile, large enough for coherent output |
| Architecture        | GPT-2 style decoder-only transformer | Proven for text generation, well-documented               |
| Vocabulary size     | 50,257                               | GPT-2 BPE tokenizer default                               |
| Embedding dimension | 384                                  | Balanced for 15M param budget                             |
| Transformer blocks  | 6                                    | Enough depth for language patterns                        |
| Attention heads     | 6                                    | One per block, captures multiple perspectives             |
| Context window      | 128 tokens                           | Sufficient for short children's stories                   |
| Training data       | 2.12M stories (TinyStories)          | Curated for 3-4 year old vocabulary                       |
| Training loss       | ~2.39 (target)                       | Coherent grammar, developing story logic                  |

## Running Locally

### Prerequisites

- macOS (Apple Silicon recommended)
- Python 3.11+ with PyTorch
- Xcode 15+ (for iOS app)
- Google account (for Colab training)

### Training the Model

See [`training/README.md`](training/README.md) for the full training guide.

### Building the iOS App

See [`app/README.md`](app/README.md) for build instructions.

## Acknowledgments

- [TinyStories paper](https://arxiv.org/abs/2305.07759) by Eldan & Li (Microsoft Research)
- [nanoGPT](https://github.com/karpathy/nanoGPT) by Andrej Karpathy — architecture inspiration
- [Build a Small Language Model From Scratch](https://www.youtube.com/watch?v=VIDEO_ID) by Dr. Raj Gandekar (Vijuara AI Labs) — training pipeline reference

## License

MIT
