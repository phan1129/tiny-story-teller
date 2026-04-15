# Architecture Decisions

This document records the key technical decisions made in this project and the reasoning behind each one. Written before implementation to force clear thinking.

---

## Decision 1: Why TinyStories Dataset?

**Context:** We need training data for a small language model. Options include web scraping, Wikipedia, books, or specialized datasets.

**Decision:** Use the TinyStories dataset (2.12M stories for 3-4 year olds) from HuggingFace.

**Why:**

- The [TinyStories paper](https://arxiv.org/abs/2305.07759) proved that a carefully curated, domain-specific dataset enables small models to produce coherent output — something that fails when training small models on general internet data
- Stories use limited vocabulary but cover the full structure of English (grammar, narrative flow, cause and effect)
- The dataset is small enough to train on a single GPU in ~30 minutes (A100) to ~7 hours (T4)
- It's publicly available on HuggingFace — no scraping, no licensing issues

**Tradeoff:** The model will only generate children's stories, not general text. That's the point — we're proving that a constrained dataset + small model can still produce coherent language.

---

## Decision 2: Why ~15M Parameters?

**Context:** We need to choose a model size. Options range from 1M to 1B+ parameters.

**Decision:** Target approximately 15 million parameters.

**Why:**

- Small enough to run on a mobile device without excessive memory or battery drain
- Large enough to capture grammar patterns and basic story logic (proven by TinyStories research)
- Roughly 10,000x smaller than GPT-3 (175B) — demonstrates true edge AI, not just "slightly smaller cloud model"
- Trains in 25-30 minutes on an A100 GPU (Colab Pro) or 6-7 hours on T4 (free tier) — practical for iteration

**Configuration:**

- 6 transformer blocks × 6 attention heads = 36 total attention heads
- 384 embedding dimensions (not 768 like GPT-2 — reduced to fit parameter budget)
- 128 token context window (sufficient for short stories, keeps memory footprint low)
- 50,257 vocabulary size (GPT-2 BPE default)

---

## Decision 3: Why GPT-2 BPE Tokenizer (tiktoken)?

**Context:** We need to convert text to numbers before the model can process it. Three main approaches: word-based, character-based, or subword tokenization.

**Decision:** Use GPT-2's Byte Pair Encoding (BPE) tokenizer via the `tiktoken` library.

**Why:**

- **Word-based tokenization** creates a huge vocabulary (500K+ words in English) and can't handle misspellings or novel words
- **Character-based tokenization** creates extremely long sequences that overwhelm the attention mechanism
- **Subword tokenization (BPE)** is the sweet spot — common words stay as single tokens, rare words get broken into meaningful subwords
- GPT-2's tokenizer is battle-tested, well-documented, and `tiktoken` is fast
- Vocabulary size of 50,257 is manageable for our model's output layer

**Tradeoff:** Using a pre-built tokenizer means we're not training our own from scratch. This is the right call — the tokenizer isn't the learning objective here, the model architecture is.

---

## Decision 4: Why CoreML for On-Device Inference?

**Context:** We need to run the model on an iPhone. Options include CoreML, ONNX Runtime, TensorFlow Lite, or custom inference code.

**Decision:** Convert to CoreML format using `coremltools`.

**Why:**

- CoreML is Apple's native ML framework — it automatically uses the Neural Engine (specialized ML hardware) on Apple Silicon
- No third-party dependencies in the iOS app
- Built-in support for model quantization (INT8, INT4)
- Integrates naturally with SwiftUI
- Apple actively maintains and optimizes CoreML for new hardware

**Tradeoff:** This locks us into Apple's ecosystem. Android deployment would require a separate conversion (ONNX or TFLite). For a portfolio project targeting iOS, this is the right choice.

---

## Decision 5: Why Train From Scratch Instead of Fine-Tuning?

**Context:** We could either train a model from zero (random initialization) or fine-tune an existing pre-trained model.

**Decision:** Train from scratch.

**Why:**

- **Learning objective** — This project is meant to demonstrate understanding of the full ML pipeline, not just API usage
- **Size control** — Pre-trained models come with fixed architectures. Training from scratch lets us design for exactly 15M parameters
- **Portfolio differentiation** — Most people fine-tune. Training from scratch is rarer and demonstrates deeper understanding
- **Transparency** — Every parameter in the model was learned from our dataset. No hidden knowledge from pre-training on unknown data

**Tradeoff:** The model will be less capable than a fine-tuned version of the same size. We accept this because the learning and portfolio value outweigh raw performance.

---

## Decision 6: Quantization Strategy

**Context:** The trained model in float32 will be too large and slow for mobile. We need to compress it.

**Decision:** Dynamic quantization based on device capabilities — INT8 as baseline, INT4 for older devices.

**Why:**

- INT8 quantization typically reduces model size by ~4x with minimal quality loss
- INT4 reduces by ~8x but with more noticeable degradation — acceptable on memory-constrained devices
- CoreML supports both through `coremltools` quantization utilities
- We'll benchmark quality degradation at each level before shipping

**Implementation plan:**

1. Convert float32 PyTorch model → CoreML float16 (baseline)
2. Apply INT8 quantization → measure size + quality
3. Apply INT4 quantization → measure size + quality
4. At runtime: detect device capabilities → load appropriate model variant

---

## Decision 7: Offline-First Architecture

**Context:** The app could work fully offline, require internet, or use a hybrid approach.

**Decision:** Fully offline. No network calls for inference. Optional cloud sync for saved stories only.

**Why:**

- **Privacy** — Children's content demands maximum privacy. No user data leaves the device for inference
- **Reliability** — Works on airplanes, in areas with poor connectivity, without Wi-Fi
- **Cost** — Zero ongoing API costs. The model runs on-device for free
- **Portfolio signal** — Demonstrates edge AI capability, not just "wrap an API in a mobile app"

**Implementation:**

- Model bundled in the app binary (or downloaded once on first launch)
- Stories saved locally using SwiftData with CryptoKit encryption
- Optional: iCloud sync for cross-device access (user opt-in only)

---

## Decision 8: Memory and Battery Management

**Context:** Running ML models on mobile devices competes with other apps for memory and drains battery.

**Decision:** Implement lazy loading, memory pressure monitoring, and battery-aware throttling.

**Why:**

- iOS will kill your app if it uses too much memory. We need to be a good citizen
- Users will stop using an app that kills their battery
- These are production-grade concerns that separate toy projects from real engineering

**Implementation:**

- **Lazy loading** — Don't load the model at app launch. Load when user requests a story, unload after idle timeout
- **Memory pressure** — Subscribe to `os_proc_available_memory()` or `didReceiveMemoryWarning`. Unload model if system signals pressure
- **Battery** — Check `UIDevice.current.batteryLevel` and `batteryState`. Throttle inference (reduce max tokens, increase delay between tokens) below 20%. Defer non-critical processing until charging
- **Sliding context window** — If conversation/story exceeds 128 tokens, keep most recent context and drop oldest. Use simple recency (not semantic similarity) for v1
