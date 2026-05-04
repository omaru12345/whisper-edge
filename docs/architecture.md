# whisper-edge — System Architecture

## 1. System diagram

The project runs entirely client-side in the browser, so the architecture is shaped by Chrome Extension Manifest V3 (MV3) constraints while extracting the maximum compute from WebGPU. Inference runs in a Web Worker hosted inside an Offscreen Document so the main thread is never blocked.

```mermaid
graph TD
    subgraph Chrome Extension [Chrome Manifest V3]
        CS[Content Script <br> Google Meet DOM]
        BG[Background <br> Service Worker]
        OD[Offscreen Document <br> Audio & Inference]
    end

    subgraph "whisper-edge SDK"
        AP[Audio Processor <br> AudioWorklet]
        VAD[Silero VAD <br> Voice Activity Detection]
        WW[Inference Worker <br> Web Worker]
        ORT[ONNXRuntime Web <br> WebGPU / WASM]
        IDB[(IndexedDB <br> Model Cache)]
    end

    %% Data flow
    CS -- 1. Meeting start event --> BG
    BG -- 2. chrome.tabCapture --> OD
    OD -- 3. MediaStream --> AP
    AP -- 4. Float32Array --> VAD
    VAD -- 5. Voiced chunk (1-30s) --> WW
    WW -- 6. Tensor conversion --> ORT
    ORT -- 7. Model load/save --> IDB
    ORT -- 8. Inference --> WW
    WW -- 9. Transcribed text --> OD
    OD -- 10. Message API --> CS
    CS -- 11. Render captions / minutes --> User((User))

    classDef sdk fill:#f9f,stroke:#333,stroke-width:2px;
    class AP,VAD,WW,ORT,IDB sdk;
```

## 2. Module responsibilities

| Module | Responsibility |
|---|---|
| **Content Script** | Observe the Google Meet DOM, inject UI (caption overlay, minutes export). |
| **Background (Service Worker)** | Manage extension lifecycle, capture tab audio with `chrome.tabCapture`, create / tear down the Offscreen Document. |
| **Offscreen Document** | Hidden page used to access DOM / Audio APIs from MV3. Hosts the AudioContext and is the SDK entry point. |
| **Audio Processor** | Downsample to 16 kHz mono in real time using an `AudioWorklet`. |
| **VAD (Voice Activity Detection)** | Trim silence so we never spend compute on empty audio. Slice voiced regions into chunks for the Inference Worker. |
| **Inference Worker** | Web Worker isolated from the main thread. Handles preprocessing (log-mel spectrogram) and tensor construction for the ONNX model. |
| **ONNXRuntime Web** | The inference engine. Runs on WebGPU (fp16/int8) and transparently falls back to WASM SIMD when WebGPU is unavailable. |

## 3. Browser environment constraints and mitigations

* **Manifest V3 (MV3) constraints:** Service Workers cannot reach DOM, Web Audio, or WebGPU APIs. Mitigation: receive the audio stream and run inference entirely inside the Offscreen Document.
* **WebGPU availability:** Stable on Windows / Mac Chrome but unreliable on Linux and older GPUs. Probe `navigator.gpu` at init; fall back to WASM (SIMD + multi-thread) when absent.
* **Memory limits:** Browser tabs typically cap at 2–4 GB. To prevent KV cache bloat we reset the maximum context length in 30-second chunks.

## 4. Dependencies

* `onnxruntime-web` (inference engine, WebGPU / WASM)
* `@ricky0123/vad-web` (or an equivalent lightweight Silero VAD)
* `huggingface/transformers.js` (tokenizer and feature-extraction routines)
* `localforage` (IndexedDB-backed cache for the large model file)

## 5. Performance targets

* **Model:** Whisper tiny / base or Distil-Whisper, INT8-quantized.
* **Model size:** **≤ 50 MB** (minimize network load; warm start ≤ 1 s after IndexedDB cache).
* **Latency (RTF, real-time factor):** **≤ 0.1** (process 10 s of audio in under 1 s).
* **Accuracy (WER, word error rate):**
    * English (LibriSpeech test-clean): **≤ 12%**
    * Japanese (Common Voice): **≤ 18%**
* **CPU / GPU footprint:** keep Meet's render FPS drop **below 10%** during a meeting.
