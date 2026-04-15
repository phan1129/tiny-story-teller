# Tiny Story Teller — iOS App

SwiftUI app that runs the trained SLM on-device via CoreML to generate children's stories.

## Contents (coming in Phase 3)

- `TinyStoryTeller/` — Xcode project
- `TinyStoryTeller/Models/` — CoreML model integration
- `TinyStoryTeller/Views/` — SwiftUI views
- `TinyStoryTeller/Services/` — Inference engine, tokenizer, storage

## Features (planned)

- Story generation from user prompts
- On-device inference (no internet required)
- Lazy model loading
- Memory pressure monitoring
- Battery-aware inference throttling
- Encrypted local story storage

## Requirements

- iOS 17+
- Xcode 15+
- Apple Silicon Mac for development

See [ARCHITECTURE.md](../ARCHITECTURE.md) for design decisions.
