# aac-models

Public hosting for the on-device speech-to-text model assets used by **AAC (Ambient Audio Companion)**.

The AAC app's code repository is private, so these model files live here as a public release so the app can **download them on first launch** (no manual `adb push` needed). The files are repackagings / mirrors of open-source models — all credit to the original authors (see [Licensing](#licensing)).

## Assets

### Release [`models-v1`](https://github.com/jetzhu/aac-models/releases/tag/models-v1)

| Asset | Size | Role | Default |
|---|---|---|---|
| `sherpa-stream-zh-en-int8.zip` | ~168 MB | **Real-time** on-device STT (the app's default engine) | Auto-downloaded on first launch (Wi-Fi) |
| `qwen3-asr-0.6b-q8_0.gguf` | ~1.35 GB | **Offline** high-quality re-transcription pass | Opt-in (download only when the user enables it) |

#### `sherpa-stream-zh-en-int8.zip`
Trimmed **int8** repackaging of `sherpa-onnx-streaming-zipformer-bilingual-zh-en-2023-02-20` — a streaming Zipformer transducer, **bilingual Chinese + English**, runs fully offline on-device (no GMS required). Used as AAC's real-time transcription engine.

Unzips (flat) to:
- `encoder-epoch-99-avg-1.int8.onnx`
- `decoder-epoch-99-avg-1.onnx`
- `joiner-epoch-99-avg-1.int8.onnx`
- `tokens.txt`

The full upstream model (including fp32 weights, ~511 MB `.tar.bz2`) is published by k2-fsa under the [`asr-models`](https://github.com/k2-fsa/sherpa-onnx/releases/tag/asr-models) release; AAC falls back to that URL if this asset is unavailable.

#### `qwen3-asr-0.6b-q8_0.gguf`
**Qwen3-ASR 0.6B**, `Q8_0` GGUF quantization (for qwen3-asr.cpp / llama.cpp-style runtimes). AAC uses it for an **offline re-processing pass** that re-transcribes recordings at higher quality when the device is idle. Not downloaded by default — the user opts in from Settings.

## How AAC consumes these

The app's `ModelDownloader`:
1. On first launch (Wi-Fi only), downloads `sherpa-stream-zh-en-int8.zip` from this release, unzips it into the app's private models directory, and starts transcribing.
2. Has a built-in **fallback chain** — if this primary host is unreachable it pulls the upstream k2-fsa tarball instead.
3. Downloads `qwen3-asr-0.6b-q8_0.gguf` only on explicit user opt-in.

On GMS devices, the app can also bootstrap with Google's on-device recognizer (SODA) while the sherpa model downloads — that engine is part of the OS and is **not** hosted here.

## Updating / adding a model

1. Upload the asset to a release here:
   ```
   gh release upload models-v1 <file> --repo jetzhu/aac-models
   ```
2. Update the corresponding URL in the app's `ModelDownloader.kt`.

> ⚠️ Don't upload a file that's still being written/copied — `gh` reads the size at the start, and if the file grows you'll get `http2: request body larger than specified content length`. Wait until the file is complete, then upload (re-run with `--clobber` to replace).

## Licensing

These are redistributions of third-party models under their original licenses:

- **sherpa-onnx / Next-gen Kaldi (k2-fsa)** — Apache-2.0 — <https://github.com/k2-fsa/sherpa-onnx>
- **Qwen3-ASR (Alibaba / QwenLM)** — see the upstream model card for its license terms — <https://github.com/QwenLM>

No model weights here are original work; this repo only mirrors/repackages them for convenient first-run download. If you are a rights holder and want an asset removed, open an issue.
