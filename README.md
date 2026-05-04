# whisper-edge

Browser-only realtime ASR library that runs entirely on **WebGPU / WebAssembly**.
A JavaScript SDK that transcribes tab audio **without sending a single byte to the cloud**, plus a reference Chrome extension that uses it to take Google Meet minutes.

---

## 🎯 Goal: Acquisition by Google

This project targets a **Jetpac-style acqui-hire by Google** (2–5 person team / technical core + UX demo).

**The acquisition target is the SDK (the technical core); the Chrome extension is a distribution channel and showcase.**

| Likely buyer | Integration scenario |
|---|---|
| **Google Meet team** | Eliminate server-side transcription cost and unblock enterprise customers' security concerns with a "local transcription mode". |
| **Chrome team** | Browser-standard caption API for any tab audio (the web counterpart of Live Caption). |
| **Workspace** | Auto-extract action items into Docs / Tasks; auto-archive minutes into Drive. |

### Technical moat we are paid for
1. **Model size** (Whisper distilled and quantized below 50 MB)
2. **Latency** (streaming response under 300 ms)
3. **Stable execution in the browser** (WebGPU primary → WASM SIMD fallback)
4. **Multilingual quality** (winning on Japanese accuracy)

---

## Tech stack

| Layer | Tech |
|---|---|
| Model | Whisper / Distil-Whisper exported to ONNX, INT8-quantized |
| Inference runtime | onnxruntime-web (WebGPU backend preferred, WASM SIMD fallback) |
| SDK | TypeScript (npm: `whisper-edge`) |
| Demo extension | Chrome Manifest V3 + WebAudio API (tab capture) |
| Benchmarks | Playwright + public speech datasets (e.g. Common Voice Japanese) |

---

## Repository layout

```
whisper-edge/
├── src/                    # Core SDK (TypeScript)
├── examples/
│   └── meet-extension/     # Chrome extension demo for Google Meet
├── bench/                  # Latency / accuracy benchmarks
└── docs/                   # Architecture and acquisition pitch material
```

---

## Three-phase plan

### Phase 1 (≤ 1 month): Technical PoC
- [ ] Run Whisper-tiny on WebGPU and produce a transcription
- [ ] WER < 20% on five minutes of Japanese audio
- [ ] Latency measurement script

### Phase 2 (≤ 3 months): SDK
- [ ] Publish the `whisper-edge` npm package
- [ ] Streaming API (30-second chunks → partial results)
- [ ] Switch to Distil-Whisper to stay under 50 MB

### Phase 3 (≤ 6 months): Showcase & exposure
- [ ] Publish the Meet minutes Chrome extension on the Chrome Web Store
- [ ] HackerNews / Product Hunt launch
- [ ] Send demos to contacts at Google DeepMind / Workspace

---

## Development commands

```bash
npm install
npm run dev            # Hot reload for SDK + demo extension
npm test
npm run bench          # Latency / WER measurements
npm run build:ext      # Produce the Chrome extension zip
```

(Implementations land when Phase 1 starts.)
