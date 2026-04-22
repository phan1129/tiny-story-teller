# Learnings Log

Documenting what I learned, what surprised me, and where reality differed from the plan.

---

## Phase 1: Training the SLM

**Date:** April 17, 2025

### Expected vs. Actual

| What            | Planned          | Actual  | Why                                                                                                                                                                              |
| --------------- | ---------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Parameters      | ~15M             | ~30M    | Underestimated the cost of the token embedding layer (50,257 × 384 = 19.3M alone) plus 6 transformer blocks (~10.6M). The embedding layer accounts for ~2/3 of total parameters. |
| Validation loss | ~2.39            | 1.69    | Got a more powerful GPU (RTX PRO 6000 Blackwell, 102GB) than expected, and 30M params gives the model more capacity to learn                                                     |
| Training time   | 25-30 min (A100) | 131 min | Different GPU architecture, larger effective model                                                                                                                               |
| Total tokens    | ~100M            | 474M    | Original estimate was rough. Actual stories average 169 words, not 50-100                                                                                                        |

### Key Concepts Learned

- **GPU vs CPU:** GPU is like 10,000 elementary students solving simple math in parallel vs. 1 Harvard student working sequentially. Matrix multiplication (the core ML operation) is perfectly suited for parallel work.
- **memmap:** Like a parking garage — every token ID is a fixed-size spot (2 bytes). You jump directly to the spot you need without scanning. Keeps data on disk instead of loading 474M tokens into RAM.
- **Tokenization (BPE):** Subword tokenization is the sweet spot between word-level (vocabulary too large) and character-level (sequences too long). Common words stay whole, rare words get split.
- **Training vs validation:** Training data = textbook you study. Validation data = test you've never seen. If you do well on both, you actually learned — not just memorized.
- **Loss:** A number measuring how wrong the model is. High = bad predictions, low = good. We track both training and validation loss to catch overfitting (memorizing vs learning).
- **Tensors:** Arrays of numbers that live on the GPU. 1D = a list, 2D = a table, 3D = a stack of tables. Our input batch is (32, 128) — 32 sequences of 128 tokens each.
- **Context size vs batch size:** Independent choices. Context = how many tokens the model sees at once (128). Batch = how many sequences processed in parallel (32). They don't need to match.
- **Why 384 dimensions:** More dimensions = more capacity to capture word meaning. 384 is a design choice balancing quality vs model size. GPT-2 uses 768.
