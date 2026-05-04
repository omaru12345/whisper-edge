# 02 — Scope

A small team can only win by being narrower than the competition. This file defines
the hard scope. Anything outside it must be rejected, even if technically interesting.

## DO focus on

- Model size (≤ 50 MB after INT8 quantization)
- End-to-end latency (RTF ≤ 0.1) and stable streaming
- Browser stability across WebGPU and WASM SIMD fallback
- Japanese WER (a strong differentiator vs. English-only competitors)
- A clean, minimal SDK API that drops into any web app

## DO NOT build

- **Server-side inference.** Touching a backend defeats the entire moat (zero-cloud, zero-cost).
- **Adjacent features**: meeting summarization, speaker diarization, action-item extraction.
  These are someone else's product. Stay focused on transcription quality.
- **Your own foundation model.** Distill and quantize Whisper / Distil-Whisper. Google has Gemini;
  beating them on raw model training is impossible and irrelevant.
- **Native iOS/Android apps.** Browser-first. Native bindings can come post-acquisition.
- **A standalone product business.** This is a technology and team, not a SaaS.

## Why this matters

Every "DO NOT" item above has been considered and rejected because it would either
(a) duplicate Google's existing capability, (b) dilute the technical moat, or
(c) push the team toward operational work that a 1–5 person crew cannot sustain.

If you believe a "DO NOT" item should be reopened, escalate to the human owner —
do not start implementing it.
