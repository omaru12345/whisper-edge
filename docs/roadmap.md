# Roadmap (12 weeks)

This roadmap maps out the 12-week journey from SDK development through Chrome extension delivery to acquisition-oriented marketing, with concrete tasks and Definition of Done (DoD) for each.

## Phase 1: SDK core development and model optimization (Weeks 1–4)

**Goal:** Whisper runs standalone in the browser and meets the benchmark targets.

* **Week 1: Model pipeline**
    * Tasks: Export Whisper (tiny/base) and Distil-Whisper to ONNX. Apply INT8 static / dynamic quantization with the ONNX Runtime toolchain.
    * DoD: An ONNX model file ≤ 50 MB exists and shows ≤ 2% WER degradation in Python.
* **Week 2: Inference engine (WebGPU / WASM)**
    * Tasks: TypeScript inference wrapper around `onnxruntime-web`. Run inference on a Web Worker and message-pass with the main thread.
    * DoD: A local `.wav` file decodes to text in the browser. Both WebGPU and WASM fallbacks work.
* **Week 3: Audio preprocessing and VAD**
    * Tasks: 16 kHz downsampling in an AudioWorklet. Integrate Silero VAD and chunk-splitting logic.
    * DoD: Only voiced regions of the live mic stream are forwarded to the Inference Worker.
* **Week 4: SDK packaging and benchmarks**
    * Tasks: Configure the npm package build. Add automated benchmark tests.
    * DoD: Tests on LibriSpeech (test-clean) and Common Voice (ja) finish and meet RTF ≤ 0.1, WER ≤ 12% (en) / 18% (ja).

## Phase 2: Chrome extension and browser integration (Weeks 5–8)

**Goal:** Capture audio from a Google Meet tab and transcribe it inside the MV3 sandbox.

* **Week 5: MV3 boilerplate and Offscreen lifecycle**
    * Tasks: MV3 base setup. Background Service Worker that creates and disposes the Offscreen Document.
    * DoD: Clicking the extension icon spins up the Offscreen Document and tears it down cleanly.
* **Week 6: Audio capture wired to the SDK**
    * Tasks: Use `chrome.tabCapture` to intercept Meet tab audio and pass the MediaStream into the SDK inside the Offscreen Document.
    * DoD: When someone speaks in Meet, transcribed text appears in the Offscreen console.
* **Week 7: Content script overlay on the Meet UI**
    * Tasks: Inject a caption overlay and side panel (minutes list) into the Meet DOM via React or similar.
    * DoD: Transcribed text renders live on screen without breaking Meet's native UI.
* **Week 8: Model cache and UX polish**
    * Tasks: Cache the ONNX model in IndexedDB. Show a progress bar on first load.
    * DoD: On second launch, the inference engine becomes ready in ≤ 1 s with no network requests.

## Phase 3: Performance tuning and launch (Weeks 9–12)

**Goal:** Production-grade stability, technical assets for outreach, begin contacting Google.

* **Week 9: Profiling and memory leak fixes**
    * Tasks: Heap snapshot analysis with Chrome DevTools. Stress test simulating a 1+ hour meeting.
    * DoD: 60 minutes of continuous inference holds steady memory (e.g. ≤ 500 MB), no crashes, FPS drop stays below 10%.
* **Week 10: Hugging Face Spaces demo**
    * Tasks: Build a standalone browser demo of the SDK and deploy it to Hugging Face Spaces.
    * DoD: Anyone can open the URL, allow the mic, and try realtime transcription.
* **Week 11: Chrome Web Store review and docs**
    * Tasks: Submit the extension to CWS (privacy policy explicitly stating no data leaves the device). Publish README and architecture docs.
    * DoD: Extension is "Published" on the Chrome Web Store and installable.
* **Week 12: Marketing and outreach to Google**
    * Tasks: Product Hunt launch. Publish benchmarks (the moat) on X / LinkedIn. Direct contact with Googlers.
    * DoD: 1,000 initial MAU and at least one positive response (or meeting) from a target Google engineer / PM.

---

## Risks and mitigations

1. **Risk:** WebGPU incompatibility / crashes across environments.
    * **Mitigation:** Strict feature-detection at init. Force the safer WASM (SIMD + threads) backend when the GPU vendor / driver has a known high error rate.
2. **Risk:** Offscreen Document killed by MV3 memory limits.
    * **Mitigation:** Stream to IndexedDB, periodically GC and rebuild the AudioContext / inference instance every 30 seconds.
3. **Risk:** Initial ONNX download size hurts UX.
    * **Mitigation:** Split into a base model and a high-accuracy model. Start with the small base (tens of MB) immediately and prefetch the high-accuracy model in the background.

## Areas of high technical uncertainty

* **KV cache growth control:** Long-running streaming inference bloats the KV cache and can starve memory. Tuning when to truncate context (against WER) carries unresolved uncertainty.
* **Meet DOM volatility:** Google Meet's DOM changes often, which can break the content-script UI injection. We need a robust MutationObserver-based anchor search algorithm.

## Benchmark / dataset selection criteria

* **English:** `LibriSpeech test-clean` (clean baseline) and `test-other` (noise robustness).
* **Japanese:** Latest `Mozilla Common Voice (ja)` test set.
* **Eval scripts:** Standard WER via Python `jiwer`, with punctuation and case normalized. RTF computed as `processing time / audio duration`.
